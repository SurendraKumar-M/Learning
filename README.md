# Axios Interview Questions & Answers for 5+ Years React, React Native & Expo Developers

**Audience:** Developers preparing for mid-level to senior React / React Native / Expo interviews.

**How to use this guide:** For every question, be ready to explain (1) the definition, (2) how it works, (3) when you would use it, and (4) the trade-offs or failure modes. Code samples use TypeScript unless noted. Adapt endpoint URLs, API response schemas, and token-storage APIs to your own application.

> **Version note:** Axios APIs and adapter behavior can vary by version and runtime. The examples use current Axios 1.x-style APIs. Validate upload progress, cookie behavior, adapters, and native integrations against the version in your project.

## Contents

1. Axios Fundamentals
2. Request Configuration and Responses
3. Interceptors
4. Error Handling, Timeouts, Cancellation, and Retries
5. TypeScript and API Architecture
6. React / React Native / Expo Integration
7. Authentication, Security, and Reliability
8. Testing and Debugging
9. Senior-Level Scenario Questions
10. Axios Coding Questions and Answers
11. Rapid-Fire Revision
12. Official References

---

# Part 1 — Axios Fundamentals

## Q1. What is Axios?

**Interview answer:** Axios is a Promise-based HTTP client for JavaScript and TypeScript. It provides methods for making HTTP requests and common features such as request/response interceptors, timeouts, cancellation, request configuration, and consistent error objects.

```ts
import axios from 'axios';

const response = await axios.get('https://api.example.com/users');
console.log(response.data);
```

**Senior follow-up:** Axios handles the HTTP-client concerns; it does not replace API design, runtime validation, application state management, caching strategy, or secure credential handling.

## Q2. Why use Axios instead of `fetch`?

Axios provides a convenient API with instances, interceptors, configurable timeouts, request transforms, cancellation, and Axios-specific error metadata. Native `fetch` is built into many current runtimes and can be sufficient for smaller applications or teams that prefer fewer dependencies.

A critical difference: `fetch` generally resolves for HTTP statuses such as 404 or 500 and requires checking `response.ok`; Axios normally rejects responses outside its configured `validateStatus` range.

```ts
// fetch
const response = await fetch(url);
if (!response.ok) throw new Error(`HTTP ${response.status}`);
const data = await response.json();

// Axios
const { data } = await axios.get(url);
```

**Senior answer:** Choose based on project needs and team conventions. Axios is valuable when centralized interceptors, consistent request configuration, typed error handling, and shared API-client behavior are helpful. It is not automatically faster than `fetch`.

## Q3. How do you install Axios?

```bash
npm install axios
# or
yarn add axios
```

For TypeScript, Axios includes its own type definitions; you generally do not need a separate `@types/axios` package.

## Q4. What HTTP methods does Axios support?

Common aliases include:

```ts
axios.get(url, config);
axios.post(url, data, config);
axios.put(url, data, config);
axios.patch(url, data, config);
axios.delete(url, config);
axios.head(url, config);
axios.options(url, config);
```

`post`, `put`, and `patch` accept the request body as the second argument. `get` and `delete` usually take configuration as the second argument. A body on `DELETE` is possible through `config.data`, but support and semantics should be agreed with the backend.

## Q5. What is an Axios instance, and why use `axios.create()`?

An instance is a configured Axios client. It centralizes base URL, timeout, common headers, and interceptors without changing every request manually.

```ts
import axios from 'axios';

export const api = axios.create({
  baseURL: 'https://api.example.com/v1',
  timeout: 10_000,
  headers: { Accept: 'application/json' },
});
```

**Why it matters:** You can create separate clients for public APIs, authenticated APIs, uploads, or different backend services. Avoid using the global Axios object for every concern if instances provide clearer boundaries.

## Q6. Explain Axios configuration precedence.

In general, configuration supplied to an individual request overrides instance defaults, which override library defaults.

```ts
const api = axios.create({ timeout: 5_000 });

api.get('/users', { timeout: 12_000 }); // request timeout wins
```

Be careful when merging nested settings such as headers. Prefer Axios's supported config behavior over manually deep-merging configuration objects without understanding the consequences.

## Q7. What is `baseURL`?

`baseURL` is prepended to relative request URLs.

```ts
const api = axios.create({ baseURL: 'https://api.example.com/v1' });
await api.get('/users'); // https://api.example.com/v1/users
```

Don't assume `baseURL` is a security boundary. Absolute URLs can override it, and URL normalization can make untrusted path input behave unexpectedly. Never pass arbitrary user-controlled URLs or request config directly to an authenticated Axios instance without validating them.

## Q8. How do you pass query parameters?

Use `params` rather than manually concatenating query strings.

```ts
const response = await api.get('/products', {
  params: { page: 2, limit: 20, search: 'phone' },
});
```

Axios serializes parameters. For complex arrays or nested objects, configure `paramsSerializer` to match the backend's expected format.

## Q9. How do you send a request body?

```ts
type CreateUserInput = { name: string; email: string };

const payload: CreateUserInput = {
  name: 'Sam',
  email: 'sam@example.com',
};

const response = await api.post('/users', payload);
```

Axios normally serializes plain JavaScript objects as JSON. For `FormData`, URL-encoded data, streams, or binary payloads, use the format the endpoint expects and verify adapter/runtime requirements.

## Q10. What does an Axios response contain?

Common fields are:

```ts
const response = await api.get('/users');

response.data;       // payload returned by server
response.status;     // HTTP status, e.g. 200
response.statusText; // status text; may be empty depending on runtime
response.headers;    // response headers
response.config;     // request config
response.request;    // underlying request object, runtime-specific
```

Most application service functions should return `response.data` rather than leaking the complete Axios response to UI components, unless the caller needs status or headers.

## Q11. What is the difference between `data`, `params`, and `headers`?

- `data`: request body, common for POST/PUT/PATCH.
- `params`: query-string parameters.
- `headers`: metadata such as `Authorization`, `Accept`, and `Content-Type`.

A frequent bug is putting query parameters in `data` for a GET request, or putting a JSON object in `params` when the backend expects a request body.

## Q12. How do you set headers?

Per request:

```ts
await api.get('/profile', {
  headers: { 'X-Client-Version': '2.1.0' },
});
```

For common headers, use instance defaults or a request interceptor. For dynamic authorization, a request interceptor or a dedicated authenticated client is usually more appropriate than hardcoding a token at startup.

## Q13. What is `validateStatus`?

`validateStatus` determines which HTTP status codes Axios treats as fulfilled instead of rejected.

```ts
const response = await api.get('/resource', {
  validateStatus: status => status < 500,
});
```

Use it deliberately. If a 401 or 404 becomes a fulfilled response, response-error interceptors won't handle it as a rejection, so callers must check those statuses themselves.

## Q14. Does Axios automatically retry failed requests?

No. Axios does not provide a universal automatic retry policy by default. You must implement retry logic yourself or choose a compatible library. A sound policy needs to decide which errors are transient, how many attempts are allowed, and whether retrying is safe for the HTTP operation.

## Q15. Does Axios cache responses by default?

Axios itself is primarily an HTTP client, not a full server-state cache. A project can use an explicit cache strategy or a library such as RTK Query, TanStack Query, or a carefully designed adapter/cache layer. Think separately about HTTP caching headers, application cache lifetime, invalidation, and stale data.

---

# Part 2 — Request Configuration and Responses

## Q16. What is `timeout`?

It defines the maximum time Axios waits before aborting a request, in milliseconds.

```ts
const api = axios.create({ timeout: 10_000 });
```

A timeout is not the same as an application-level deadline across several requests. If an operation involves retries or several dependent API calls, define the overall time budget as well as per-request timeouts.

## Q17. How do you upload a file with Axios in React Native?

For a React Native file picker that returns a URI, construct `FormData` using the file information expected by the platform and backend.

```ts
const form = new FormData();
form.append('title', 'Profile photo');
form.append('file', {
  uri: image.uri,
  name: image.fileName ?? 'upload.jpg',
  type: image.mimeType ?? 'image/jpeg',
} as unknown as Blob);

await api.post('/uploads', form);
```

The exact file object type depends on the picker and React Native TypeScript definitions. Don't treat a local `file://` or content URI as a normal browser `File`. Test on actual target platforms. Avoid manually inventing a multipart boundary; let the runtime/client generate it as appropriate.

## Q18. How do you send URL-encoded data?

```ts
const body = new URLSearchParams({
  username: 'sam',
  grant_type: 'refresh_token',
});

await api.post('/token', body, {
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
});
```

Confirm that the backend expects URL-encoded content. Authentication endpoints often have specific body-format requirements.

## Q19. What are `transformRequest` and `transformResponse`?

They allow transforming request data before it is sent and response data before it reaches the caller. Axios already handles common JSON cases, so custom transforms should be used only when the API contract requires them. Avoid placing large, complicated business transformations in global transforms; keep domain-specific normalization in service or mapper layers.

## Q20. What is `paramsSerializer`?

It lets you customize query parameter serialization. This matters when APIs expect arrays as repeated keys, bracket notation, comma-separated values, or another defined representation. Use the backend's contract rather than assuming every server interprets array parameters identically.

## Q21. What is the difference between HTTP errors and network errors in Axios?

- **HTTP error:** The server responded with a status rejected by `validateStatus`; `error.response` is typically available.
- **Network/transport error:** No usable HTTP response was received; `error.request` may exist, but `error.response` is usually absent.
- **Request setup error:** The request could not be configured or started; inspect `error.message` and related metadata.

In React Native, a generic network error may reflect connectivity, TLS, server reachability, emulator networking, a cleartext HTTP restriction, or a native/runtime issue. Don't assume it always means the user's internet is disconnected.

## Q22. What should a service function return?

Prefer a small, well-defined contract:

```ts
type User = { id: number; name: string; email: string };

export async function getUsers(): Promise<User[]> {
  const response = await api.get<User[]>('/users');
  return response.data;
}
```

Returning only the domain data makes screens less dependent on Axios. Return a richer result only when the caller intentionally needs status, pagination headers, or other transport metadata.

## Q23. Is `axios.get<User[]>()` runtime validation?

No. The generic gives TypeScript a compile-time expectation, but it does not check that the actual server response is an array of `User` objects. Runtime payloads may be malformed or change unexpectedly.

For critical boundaries, validate data using schema validation (for example, Zod or Valibot) or a handwritten type guard before trusting it. Treat remote JSON as untrusted input.

---

# Part 3 — Interceptors

## Q24. What are Axios interceptors?

Interceptors let you run logic before a request is sent or before a response/rejection reaches the caller. Common uses include authorization headers, tracing IDs, standard error normalization, request timing, and refresh-token handling.

```ts
const requestId = api.interceptors.request.use(config => {
  config.headers.set('X-Request-Source', 'mobile');
  return config;
});

// Remove later if needed:
api.interceptors.request.eject(requestId);
```

## Q25. Request interceptors vs response interceptors?

- **Request interceptor:** modifies or inspects config before sending, such as adding a bearer token.
- **Response interceptor:** inspects successful responses or handles rejected responses, such as normalizing errors or coordinating a token refresh.

Keep interceptors small and predictable. Business-specific decisions often belong in a service or feature layer rather than a global interceptor.

## Q26. In what order do multiple interceptors execute?

In current Axios documentation, request interceptors execute in reverse registration order (LIFO), while response interceptors execute in registration order (FIFO). This is easy to forget and can matter when one interceptor depends on another. Confirm exact behavior with the Axios version in use and write a focused test if ordering is important.

## Q27. How do you add a bearer token to requests?

```ts
api.interceptors.request.use(async config => {
  const token = await tokenStore.getAccessToken();

  if (token) {
    config.headers.set('Authorization', `Bearer ${token}`);
  }

  return config;
});
```

For React Native, `tokenStore` may use platform-backed secure storage. Keep the interceptor's behavior well-defined when no token exists, and avoid attaching credentials to untrusted absolute URLs. A dedicated authenticated instance can help separate public and protected APIs.

## Q28. Why should an interceptor return `config` or `Promise.reject(error)`?

Interceptors are part of a Promise chain. Returning the config allows the request to continue. Returning `Promise.reject(error)` preserves rejection so the caller or next rejection handler can handle it. Swallowing an error or returning an unexpected value can turn failures into confusing successes.

## Q29. How do you remove an interceptor?

`use()` returns an ID; pass it to `eject()`.

```ts
const id = api.interceptors.response.use(onSuccess, onError);
api.interceptors.response.eject(id);
```

This matters when registering interceptors in a React effect or a module that can be initialized repeatedly. Prefer registering global interceptors once in a stable API-client module. Avoid adding new interceptors on every render, or requests may run duplicate logic.

## Q30. Should all error handling live in a response interceptor?

No. A response interceptor is useful for cross-cutting rules, but a global handler should not decide every screen's UX. It can classify or normalize an error; the feature/UI layer should decide whether to show inline validation, a retry button, a toast, a login screen, or a silent background failure.

---

# Part 4 — Error Handling, Timeouts, Cancellation, and Retries

## Q31. How do you handle Axios errors in TypeScript?

Caught values should be treated as `unknown` and narrowed.

```ts
import axios from 'axios';

try {
  await api.get('/profile');
} catch (error: unknown) {
  if (axios.isAxiosError(error)) {
    console.log(error.response?.status);
    console.log(error.message);
  } else {
    console.error('Unexpected error', error);
  }
}
```

This avoids assuming every thrown value is an Axios error.

## Q32. What are useful Axios error fields?

Useful fields include `message`, `code`, `config`, `request`, `response`, and `isAxiosError`. `response` is only available when a response exists. The error's config can contain headers or other sensitive request details, so don't log whole error objects indiscriminately.

## Q33. How do you classify common API error responses?

A reasonable starting point:

- `400`: invalid request; show field or request validation feedback when the backend supplies it.
- `401`: unauthenticated/invalid token; consider one coordinated refresh attempt if appropriate.
- `403`: authenticated but not permitted; don't automatically treat as token expiry.
- `404`: resource not found.
- `409`: conflict, such as duplicate or version conflict.
- `422`: semantic/field validation, depending on the API contract.
- `429`: rate-limited; honor `Retry-After` if provided and retry only under a bounded policy.
- `5xx`: server-side failure; a retry may be appropriate for safe operations.

These are conventions, not universal laws. Follow the backend's documented contract.

## Q34. How do you cancel an Axios request?

Use `AbortController` with current Axios versions.

```ts
const controller = new AbortController();

const promise = api.get('/search', {
  params: { q: 'phone' },
  signal: controller.signal,
});

controller.abort();
```

Cancellation is useful for type-ahead search, screen cleanup, or superseded requests. Handle cancellation separately from a real failure so you don't show an error alert when the user simply navigated away.

## Q35. Is `CancelToken` still recommended?

No. Axios supports `AbortController` signals and documents `CancelToken` as deprecated. Prefer the standard `AbortSignal` approach for new code unless maintaining a legacy integration.

## Q36. How do you prevent stale responses from overwriting newer search results?

Cancel the previous request or use a request sequence/version guard. Cancellation reduces wasted work, while a sequence guard protects state even if an older operation completes late or cancellation races with completion.

```ts
let latestRequest = 0;

async function searchUsers(query: string) {
  const requestId = ++latestRequest;
  const response = await api.get('/users', { params: { q: query } });

  if (requestId !== latestRequest) return; // stale result
  return response.data;
}
```

In React components, keep the controller and request identity within the right lifecycle or use a data-fetching library that already manages query identity.

## Q37. How should retries work?

Retries must be bounded and selective. Consider retrying transient network failures, `429` responses when the server indicates when to retry, or selected `5xx` responses. Avoid retrying most `4xx` errors. Use exponential backoff with jitter, a maximum retry count, an overall deadline, and observability.

**Critical:** Retrying a POST can duplicate a payment, order, or other side effect. Retry only if the operation is naturally idempotent, the backend supports idempotency keys, or the request can otherwise be safely deduplicated.

## Q38. What is exponential backoff with jitter?

Backoff increases the wait between retries; jitter adds randomness so many clients don't retry simultaneously.

```ts
const base = 300;
const cap = 8_000;
const exponential = Math.min(cap, base * 2 ** attempt);
const delayMs = Math.random() * exponential;
```

A production policy should also honor `Retry-After` where applicable, stop on cancellation, and cap attempts/time. Avoid infinite interceptor retry loops.

## Q39. Why can a timeout be misleading?

A request timeout applies to the HTTP operation, but a user-facing workflow can include token retrieval, several API calls, retries, and processing. The total duration can exceed the timeout of any individual request. Define a workflow-level deadline where the user experience requires one, and make cancellation propagate through the complete operation.

## Q40. What is `Promise.all`'s relationship to multiple Axios requests?

```ts
const [profile, settings, alerts] = await Promise.all([
  api.get('/profile'),
  api.get('/settings'),
  api.get('/alerts'),
]);
```

It starts the independent promises and rejects as soon as one rejects. It does not automatically cancel the other requests. Use `Promise.allSettled` when each result should be handled independently, and use shared cancellation when the operation's requirements call for it.

---

# Part 5 — TypeScript and API Architecture

## Q41. How do you type an Axios request and response?

```ts
type Product = {
  id: number;
  title: string;
  price: number;
};

const response = await api.get<Product[]>('/products');
const products: Product[] = response.data;
```

The generic describes the expected payload for the compiler. It does not validate the remote response at runtime.

## Q42. How do you type a POST request body and response?

```ts
type CreateProductInput = { title: string; price: number };
type CreatedProduct = CreateProductInput & { id: number };

const response = await api.post<CreatedProduct, AxiosResponse<CreatedProduct>, CreateProductInput>(
  '/products',
  { title: 'Headphones', price: 49.99 }
);
```

The full generic signature may not be necessary in everyday code. A service wrapper often gives the simplest and clearest call site.

## Q43. How do you build a reusable API service layer?

Separate transport from domain operations.

```ts
// services/products.ts
import { api } from './apiClient';

type Product = { id: string; title: string };

export async function fetchProducts(): Promise<Product[]> {
  const { data } = await api.get<Product[]>('/products');
  return data;
}
```

Screens call `fetchProducts()` instead of building URLs and request config repeatedly. A larger system may add DTO-to-domain mapping, validation, error normalization, and query caching.

## Q44. How do you share API response types between screens and services?

Place API contracts/domain types in modules with clear ownership. Avoid one huge global `types.ts` file that everyone imports without boundaries. If server DTOs differ from UI/domain models, keep both types and map explicitly to prevent transport details from leaking throughout the UI.

## Q45. How do you type Axios errors with a backend error schema?

```ts
type ApiErrorBody = {
  message?: string;
  code?: string;
  fieldErrors?: Record<string, string>;
};

try {
  await api.post('/login', values);
} catch (error: unknown) {
  if (axios.isAxiosError<ApiErrorBody>(error)) {
    const message = error.response?.data?.message ?? 'Request failed';
    console.log(message);
  }
}
```

Use optional chaining because `response` or fields in the error payload may be absent.

## Q46. Should Axios be called directly in every component?

Usually not in a medium/large application. Keep API calls in services or a query layer. Components should focus on rendering and interaction. A common structure is:

```text
Screen / Component
       ↓
Custom hook or query hook
       ↓
Feature service
       ↓
Axios instance
       ↓
Backend
```

This improves testing, reuse, consistency, and maintainability.

## Q47. Axios services vs RTK Query/TanStack Query?

Axios is the HTTP transport client. RTK Query and TanStack Query are server-state management tools that can use Axios underneath. They add features such as cache identity, request deduplication, loading/error lifecycle, invalidation, background refetch, and mutation workflows.

Use plain Axios for straightforward requests when appropriate. Use a query/cache library when the application needs structured server-state caching and synchronization. They are complementary, not direct substitutes.

---

# Part 6 — React, React Native, and Expo Integration

## Q48. Where should Axios code live in a React Native project?

One reasonable feature-oriented structure:

```text
src/
  app/
    config/
  services/
    apiClient.ts
    authService.ts
  features/
    products/
      api/
        productsService.ts
      hooks/
      screens/
      components/
      types.ts
  shared/
    errors/
    storage/
```

The structure is a starting point, not a universal rule. Maintain a clear dependency direction and avoid circular imports, especially between the API client, authentication store, and UI.

## Q49. Why might `localhost` fail in React Native?

On a physical phone, `localhost` means the phone itself, not the developer's computer. On Android emulators, the host machine may be exposed through a special emulator address; iOS simulators commonly have different networking behavior from physical devices. Confirm the correct host, port, firewall, same Wi-Fi network, and backend binding address for your environment.

Do not hardcode a developer's LAN IP in production. Use environment-specific configuration.

## Q50. Why might an HTTP URL work on one platform but fail on another?

Android and iOS can enforce transport security policies differently, and build configuration can affect cleartext HTTP. Prefer HTTPS. If local development genuinely requires HTTP, configure the relevant platform's development-only policy carefully and never weaken production transport security just to silence a networking error.

## Q51. How do Expo environment variables work with Axios?

Expo supports environment variables exposed to client code through the `EXPO_PUBLIC_` prefix, for example:

```ts
const baseURL = process.env.EXPO_PUBLIC_API_URL;
```

These values are bundled into the client and are **not secrets**. An API URL or public feature identifier may be appropriate; private API keys, signing secrets, and privileged credentials must remain server-side. Validate that the required URL exists and is correctly configured for each EAS profile/environment.

## Q52. How do you manage loading, error, empty, and success states?

Model the request lifecycle explicitly rather than inferring everything from whether an array has items.

```ts
type LoadState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; message: string };
```

A screen should distinguish an empty successful response from an error and from a request that is still loading. For refetching, it may need to keep previous data visible while indicating background activity.

## Q53. Should `useEffect` perform an Axios request?

It can, but the effect needs correct dependencies, cancellation/cleanup where appropriate, and handling for errors and stale results. For complex server-state cases, a query library often provides safer lifecycle behavior.

```tsx
useEffect(() => {
  const controller = new AbortController();

  api.get<User[]>('/users', { signal: controller.signal })
    .then(({ data }) => setUsers(data))
    .catch(error => {
      if (axios.isCancel(error) || controller.signal.aborted) return;
      setError('Could not load users');
    });

  return () => controller.abort();
}, []);
```

Don't omit dependencies just to make the request run once. If the request depends on an ID, include it and manage stale results.

## Q54. How do you paginate APIs?

Use a service that accepts page/cursor and page size, then append or replace results based on the UI interaction.

```ts
type Page<T> = { items: T[]; nextCursor?: string | null };

async function getProducts(cursor?: string): Promise<Page<Product>> {
  const { data } = await api.get<Page<Product>>('/products', {
    params: { cursor, limit: 20 },
  });
  return data;
}
```

For mobile lists, guard against duplicate `onEndReached` calls, track loading state per page, handle refresh separately, and prevent older requests from replacing newer search results.

## Q55. How do you handle offline mode?

Axios does not supply a complete offline architecture. Detect connectivity as a signal—not proof that a server is reachable—then define which reads can use cached data and which mutations need to be queued. For queued writes, consider conflict resolution, ordering, idempotency, expiry, and clear user feedback.

Retrying every request as soon as connectivity returns can overwhelm the server or duplicate side effects.

---

# Part 7 — Authentication, Security, and Reliability

## Q56. How do you implement token refresh safely?

A robust flow should:

1. Detect an eligible authentication failure (often 401, based on the backend contract).
2. Exclude the refresh endpoint itself from refresh handling.
3. Ensure only one refresh operation runs at a time.
4. Queue or coordinate requests that fail while refresh is in progress.
5. Update the stored access token securely.
6. Retry each eligible original request no more than once.
7. If refresh fails, clear session state and require sign-in.
8. Avoid retrying cancelled requests or non-replayable request bodies unless replay safety is guaranteed.

A single `isRefreshing` boolean alone can be insufficient if concurrent requests need to wait for the new token. A shared refresh Promise is often a cleaner primitive. Ensure refresh uses a client that will not recursively invoke the same refresh interceptor.

## Q57. How do you avoid an infinite 401 retry loop?

Attach a per-request retry marker and only retry once. Do not refresh for the refresh endpoint itself. If refresh fails or the retried request returns 401, reject the error and transition to signed-out state. The precise status and endpoint exclusions must match backend behavior.

## Q58. Where should tokens be stored in React Native?

Use platform-backed secure storage for sensitive credentials where appropriate, instead of ordinary unencrypted key-value storage. Choose token lifetime and refresh strategy based on the threat model and backend. Secure storage reduces exposure but does not make a compromised device or a malicious runtime harmless.

Never embed long-lived server secrets in the app bundle.

## Q59. Is adding a bearer token in an interceptor always safe?

No. If an authenticated instance accepts arbitrary absolute URLs, an interceptor might attach a token to a host that should not receive it. Validate trusted origin/path assumptions and keep authenticated clients restricted to known API hosts. Avoid passing untrusted URL strings or full request configs into an authenticated client.

## Q60. Is it safe to log `error.config`?

Not without redaction. Request configuration can include authorization headers, credentials, query parameters, and request bodies with sensitive user data. Log a safe subset such as request method, sanitized path, status, correlation ID, and error code. Redact secrets from analytics, crash reports, and console logs.

## Q61. How do you prevent duplicate form submissions?

Use UI state to disable the submit button while the request is pending, but don't rely only on the UI. For financial or other critical operations, backend idempotency keys and server-side deduplication are essential. Network retries and client double taps can otherwise create duplicate effects.

## Q62. How do you handle 429 rate limiting?

Use the backend's `Retry-After` contract where present, apply bounded backoff, and avoid retrying faster than permitted. Also reduce excess calls through debouncing, caching, pagination, request deduplication, and appropriate UI behavior. A client should not blindly loop on 429 responses.

## Q63. What is idempotency and why does it matter with Axios retries?

An operation is idempotent if repeating it has the same intended effect as performing it once. GET and PUT are designed to be idempotent by HTTP semantics, though real APIs can be misimplemented; POST is not generally idempotent. For payment/order creation, use backend-supported idempotency keys and retry rules designed around them.

---

# Part 8 — Testing and Debugging

## Q64. How do you test Axios service functions?

Test both success and failure paths. Use a request-mocking tool or mock the API client, depending on the team's test architecture. Test that the service calls the correct endpoint/parameters and maps the response as expected; don't require every test to hit a real backend.

## Q65. How do you test interceptors?

Test observable behavior: whether authorization headers are attached, how 401 responses are handled, whether refresh happens once under concurrency, whether retry markers prevent loops, and whether the original error is preserved. Because interceptor ordering can matter, test that explicitly rather than relying on memory.

## Q66. How do you debug a request that never reaches the backend?

Check, in order:

1. Is the request function executed?
2. Are URL and query parameters correct?
3. Is the runtime using the intended base URL?
4. Can the device/emulator resolve and reach the host/port?
5. Is HTTPS/TLS or cleartext policy blocking it?
6. Is an interceptor throwing or returning an invalid config?
7. Is a proxy, VPN, certificate, or firewall involved?
8. What do native device logs and backend logs show?

A generic “Network Error” alone often does not identify the cause.

## Q67. How do you debug a request that appears twice?

Check duplicate handler registration, effects that rerun, Strict Mode development behavior, navigation remounts, multiple event handlers, retries, interceptor logic, and query-library refetching. Instrument request IDs and method/path to distinguish two independent requests from one request retried by the client.

## Q68. What should be included in request observability?

Useful metadata includes a correlation/request ID, method, sanitized route, duration, status, retry count, and a normalized error category. Avoid logging access tokens, passwords, personal data, or entire payloads by default. Keep logs actionable and privacy-aware.

---

# Part 9 — Senior-Level Scenario Questions

## Q69. Ten API calls return 401 simultaneously. What do you do?

Use one shared refresh Promise. The first eligible 401 starts refresh; subsequent requests await the same Promise. If refresh succeeds, update the token and retry each eligible request once. If refresh fails, reject all waiting requests, clear the session, and navigate through the app's central auth state. Ensure cancellation and replay safety are respected.

## Q70. Search results flicker and old results replace new results. Why?

Requests are completing out of order. Use debouncing to reduce unnecessary calls, cancellation to stop superseded work, and a request identity/query key guard so only the current search result updates the UI. Debouncing alone does not guarantee correctness.

## Q71. Some requests time out only on mobile. How do you investigate?

Check device connectivity, host addressing, VPN/firewall, TLS, backend reachability, payload size, upload constraints, server latency, platform-specific networking behavior, and timeout values. Compare the same request from simulator/emulator and a real device, and collect safe request timing diagnostics.

## Q72. A request returns HTTP 200, but the UI crashes. Is Axios the problem?

Not necessarily. The response may not match the assumed schema, or the UI may not handle a null/empty field. Validate external data, map DTOs into domain models, handle optional fields, and add tests for malformed responses. TypeScript generics alone do not validate data at runtime.

## Q73. Your app is making too many API requests. How do you reduce them?

Find the source first. Consider request deduplication, server-state caching, appropriate cache invalidation, pagination, debounced search, memoized selectors, consolidating endpoints when justified, and removing duplicate effect/event logic. Don't hide needed refetches simply to reduce counts; freshness requirements matter.

## Q74. Should a response interceptor convert every error into a string?

Usually not. Converting everything to a string discards useful status codes, cancellation signals, field errors, and retry metadata. Prefer a typed normalized error model that preserves safe machine-readable information; let the UI decide how to display it.

## Q75. How would you design Axios for multiple backend services?

Create service-specific instances with explicit base URLs, timeouts, and auth policies. Share only cross-cutting helpers that genuinely match across services. A public service should not inherit private bearer tokens accidentally, and the same retry policy may not be correct for every backend.

## Q76. An upload fails halfway through. How should the UI behave?

Expose progress if the chosen adapter/runtime supports it, allow cancellation where appropriate, present a retry action, and handle temporary local files/permissions. Check if the backend supports resumable uploads before promising resume behavior. Retrying a full upload may duplicate server records unless the endpoint is idempotent.

## Q77. A release build fails while debug works. Could Axios be responsible?

It could be part of the issue, but don't assume so. Verify environment-specific API URLs, HTTPS certificates, release manifest/plist policies, native dependency/build differences, network permissions, and minification/environment configuration. Inspect device logs and backend access logs; reproduce with the exact release binary where possible.

## Q78. How do you explain Axios's role in an architecture interview?

Describe Axios as the transport layer. It sends HTTP requests and handles request configuration, interceptors, timeout, and cancellation. Feature services define endpoint/domain operations; query/state tools manage server state and caching; validation enforces runtime data contracts; components present loading, success, empty, and failure states. This clear separation prevents Axios concerns from spreading throughout the UI.

---

# Part 10 — Axios Coding Questions and Answers

The code below demonstrates patterns, not a drop-in complete application. Replace the storage implementation, backend schemas, and environment values with your app's real contracts.

## Coding Q1. Create a reusable Axios instance with a base URL and timeout

```ts
// src/services/apiClient.ts
import axios from 'axios';

const baseURL = process.env.EXPO_PUBLIC_API_URL;
if (!baseURL) {
  throw new Error('EXPO_PUBLIC_API_URL is not configured');
}

export const api = axios.create({
  baseURL,
  timeout: 10_000,
  headers: { Accept: 'application/json' },
});
```

**Explanation:** One configured instance provides consistent defaults. The environment variable is configuration, not a secret. A missing URL should fail clearly rather than silently sending requests to the wrong host.

## Coding Q2. Fetch users with a typed service function

```ts
import { api } from './apiClient';

export type User = {
  id: number;
  name: string;
  email: string;
};

export async function fetchUsers(): Promise<User[]> {
  const { data } = await api.get<User[]>('/users');
  return data;
}
```

**Follow-up:** The generic is only a compile-time promise. Validate the payload at runtime when correctness or trust boundaries require it.

## Coding Q3. Send query parameters for search and pagination

```ts
export type UserPage = {
  items: User[];
  total: number;
};

export async function fetchUsersPage(page: number, limit: number, search: string) {
  const { data } = await api.get<UserPage>('/users', {
    params: { page, limit, search: search.trim() || undefined },
  });
  return data;
}
```

**Important:** Confirm whether the API uses page numbers, offsets, or cursors. Keep pagination semantics in the service contract.

## Coding Q4. Create a user using a typed request body

```ts
type CreateUserInput = { name: string; email: string };

export async function createUser(input: CreateUserInput): Promise<User> {
  const { data } = await api.post<User>('/users', input);
  return data;
}
```

The UI should validate inputs for usability, while the server remains authoritative for validation and authorization.

## Coding Q5. Add a request ID and safe logging

```ts
api.interceptors.request.use(config => {
  const requestId = `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`;
  config.headers.set('X-Request-ID', requestId);
  // Do not log Authorization headers or request bodies here.
  return config;
});
```

For production systems, prefer a well-tested ID generator where required and coordinate the header name with the backend. Don't use `Math.random()` for security tokens.

## Coding Q6. Add a bearer token using a request interceptor

```ts
// Replace this with the secure-storage/auth abstraction used by your app.
const tokenStore = {
  async getAccessToken(): Promise<string | null> {
    return null;
  },
};

api.interceptors.request.use(async config => {
  const token = await tokenStore.getAccessToken();
  if (token) config.headers.set('Authorization', `Bearer ${token}`);
  return config;
});
```

**Production follow-up:** Ensure the token is sent only to trusted API origins and don't register the interceptor on every render.

## Coding Q7. Handle an Axios error safely in TypeScript

```ts
import axios from 'axios';

export function getErrorMessage(error: unknown): string {
  if (axios.isAxiosError<{ message?: string }>(error)) {
    return error.response?.data?.message ?? error.message ?? 'Request failed';
  }
  if (error instanceof Error) return error.message;
  return 'An unexpected error occurred';
}
```

**Follow-up:** A production app often normalizes errors into a typed structure with status, code, field errors, and retryability instead of returning only a message.

## Coding Q8. Cancel a request when a React Native screen unmounts

```tsx
import { useEffect, useState } from 'react';
import axios from 'axios';
import { api } from '../services/apiClient';

type User = { id: number; name: string };

export function UsersScreen() {
  const [users, setUsers] = useState<User[]>([]);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    api.get<User[]>('/users', { signal: controller.signal })
      .then(({ data }) => setUsers(data))
      .catch((err: unknown) => {
        if (controller.signal.aborted || axios.isCancel(err)) return;
        setError('Could not load users. Please try again.');
      });

    return () => controller.abort();
  }, []);

  // Render loading/error/data UI as needed.
  return null;
}
```

**Explanation:** Cleanup aborts the request associated with the effect. In a complete screen, model loading and empty states too. If this request depends on an ID, include that reactive dependency and prevent stale results.

## Coding Q9. Prevent an older search response from overwriting a newer one

```ts
let latestRequestId = 0;

export async function searchUsers(query: string): Promise<User[] | undefined> {
  const requestId = ++latestRequestId;
  const { data } = await api.get<User[]>('/users/search', {
    params: { q: query },
  });

  if (requestId !== latestRequestId) return undefined;
  return data;
}
```

**Caution:** A module-level counter is shared among all callers. In a real hook/component, keep the counter within the correct search instance, or use query keys and cancellation through a query library.

## Coding Q10. Implement a simple debounce utility for search

```ts
export function debounce<T extends (...args: any[]) => void>(
  callback: T,
  delayMs: number,
) {
  let timer: ReturnType<typeof setTimeout> | undefined;

  const debounced = (...args: Parameters<T>) => {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => callback(...args), delayMs);
  };

  debounced.cancel = () => {
    if (timer) clearTimeout(timer);
    timer = undefined;
  };

  return debounced;
}
```

In React, be careful to keep the debounced function stable and cancel it during cleanup. Debouncing reduces request volume; it does not replace stale-response protection.

## Coding Q11. Add bounded retry with exponential backoff

This example retries only a narrow set of safe failures. A real implementation should also handle jitter, `Retry-After`, cancellation, an overall deadline, and endpoint-specific idempotency requirements.

```ts
const sleep = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

export async function getWithRetry<T>(
  url: string,
  maxRetries = 3,
): Promise<T> {
  for (let attempt = 0; ; attempt += 1) {
    try {
      const { data } = await api.get<T>(url);
      return data;
    } catch (error: unknown) {
      const retryable = axios.isAxiosError(error) &&
        (!error.response || error.response.status >= 500);

      if (!retryable || attempt >= maxRetries) throw error;
      const delay = Math.min(8_000, 300 * 2 ** attempt);
      await sleep(delay);
    }
  }
}
```

**Senior follow-up:** Do not use this generic retry wrapper for arbitrary POST mutations without a safe replay/deduplication contract.

## Coding Q12. Normalize API errors to a stable shape

```ts
type NormalizedApiError = {
  message: string;
  status?: number;
  code?: string;
  isCanceled: boolean;
};

export function normalizeApiError(error: unknown): NormalizedApiError {
  if (axios.isAxiosError<{ message?: string; code?: string }>(error)) {
    return {
      message: error.response?.data?.message ?? error.message ?? 'Request failed',
      status: error.response?.status,
      code: error.response?.data?.code ?? error.code,
      isCanceled: axios.isCancel(error) || error.code === 'ERR_CANCELED',
    };
  }

  return {
    message: error instanceof Error ? error.message : 'Unexpected error',
    isCanceled: false,
  };
}
```

Avoid returning internal details to end users; log only safe metadata.

## Coding Q13. Upload a profile image using FormData in React Native

```ts
type UploadAsset = {
  uri: string;
  fileName?: string | null;
  mimeType?: string | null;
};

export async function uploadAvatar(asset: UploadAsset) {
  const form = new FormData();
  form.append('file', {
    uri: asset.uri,
    name: asset.fileName ?? 'avatar.jpg',
    type: asset.mimeType ?? 'image/jpeg',
  } as unknown as Blob);

  const { data } = await api.post<{ url: string }>('/profile/avatar', form);
  return data;
}
```

This uses a common React Native FormData shape; actual typings and asset fields vary by picker library and RN version. Test iOS and Android, permissions, server multipart handling, file size, and MIME validation. Don't assume a URI alone is enough for every file provider.

## Coding Q14. Make independent API requests concurrently

```ts
const [profile, settings, notifications] = await Promise.all([
  api.get<Profile>('/profile'),
  api.get<Settings>('/settings'),
  api.get<Notification[]>('/notifications'),
]);

return {
  profile: profile.data,
  settings: settings.data,
  notifications: notifications.data,
};
```

Parallel requests can reduce total waiting time when independent. If one optional request can fail without blocking the whole screen, consider `Promise.allSettled` and design partial success states.

## Coding Q15. Add a response interceptor for common error handling

```ts
api.interceptors.response.use(
  response => response,
  (error: unknown) => {
    // Add only cross-cutting handling here.
    // Do not log full config: it may contain secrets.
    return Promise.reject(error);
  },
);
```

A useful interceptor might attach telemetry metadata or normalize a well-defined API error format. Avoid triggering navigation from many competing locations; use a central auth/session coordinator for global session changes.

## Coding Q16. Coordinate concurrent token refresh with a shared Promise

This simplified pattern shows the idea; production refresh flows need a separate refresh client, token rotation rules, request replay checks, cancellation handling, and secure storage.

```ts
let refreshPromise: Promise<string> | null = null;

async function getFreshAccessToken(): Promise<string> {
  if (!refreshPromise) {
    refreshPromise = refreshAccessTokenFromServer()
      .finally(() => { refreshPromise = null; });
  }
  return refreshPromise;
}

async function refreshAccessTokenFromServer(): Promise<string> {
  // Call a dedicated refresh client/endpoint here.
  // Persist the new token using the app's secure token store.
  throw new Error('Implement using your authentication contract');
}
```

The shared Promise prevents simultaneous 401 failures from starting several refresh operations. Integrate it with a per-request retry marker and exclude the refresh endpoint from the interceptor to avoid recursion.

## Coding Q17. Prevent duplicate form submission

```tsx
const [isSubmitting, setIsSubmitting] = useState(false);

async function handleSubmit() {
  if (isSubmitting) return;
  setIsSubmitting(true);
  try {
    await createOrder(formValues);
  } finally {
    setIsSubmitting(false);
  }
}
```

In production, consider a ref/guard if multiple events can fire before React commits the state update, and use backend idempotency protection for sensitive mutations. A disabled button improves UX but cannot guarantee exactly-once server behavior.

## Coding Q18. Test a service function with a mocked Axios instance

```ts
import { describe, expect, it, vi } from 'vitest';
import { api } from './apiClient';
import { fetchUsers } from './usersService';

vi.spyOn(api, 'get').mockResolvedValue({
  data: [{ id: 1, name: 'Sam', email: 'sam@example.com' }],
} as never);

describe('fetchUsers', () => {
  it('returns the response data', async () => {
    await expect(fetchUsers()).resolves.toEqual([
      { id: 1, name: 'Sam', email: 'sam@example.com' },
    ]);
    expect(api.get).toHaveBeenCalledWith('/users');
  });
});
```

This is an illustrative Vitest example. With Jest, use `jest.spyOn`; with a separate test instance, inject the client to avoid shared mocks/interceptors leaking between tests. Add rejected-request tests as well.


## Coding Q19. Handle a 404 separately from network failure

```ts
import axios from 'axios';

async function getProduct(id: string): Promise<Product | null> {
  try {
    const { data } = await api.get<Product>(`/products/${encodeURIComponent(id)}`);
    return data;
  } catch (error: unknown) {
    if (axios.isAxiosError(error) && error.response?.status === 404) {
      return null; // Only if a missing product maps to null in this API contract.
    }
    throw error;
  }
}
```

Treat a missing product as `null` only if that is the service contract; otherwise, preserve the error so the caller can decide how to respond.

## Coding Q20. Use `Promise.allSettled` for independent optional widgets

```ts
const results = await Promise.allSettled([
  api.get<Weather>('/weather'),
  api.get<Promotion[]>('/promotions'),
  api.get<Recommendation[]>('/recommendations'),
]);

const weather = results[0].status === 'fulfilled' ? results[0].value.data : null;
const promotions = results[1].status === 'fulfilled' ? results[1].value.data : [];
const recommendations = results[2].status === 'fulfilled' ? results[2].value.data : [];
```

This is useful when the screen can provide partial content. In strongly typed production code, use named helper functions or typed result mapping to avoid fragile numeric indexing.

---

# Part 11 — Rapid-Fire Revision

## Q79. Does Axios reject on 404 by default?

Yes, under its default `validateStatus` behavior, 404 is normally treated as a rejected response. If `validateStatus` is customized, behavior may change.

## Q80. Where is the response body?

`response.data`.

## Q81. Where do query parameters go?

In the `params` config field.

## Q82. How do you set a request timeout?

Use `timeout` in milliseconds.

## Q83. How do you cancel an in-flight request?

Pass an `AbortController` signal through `signal` and call `abort()`.

## Q84. Is `CancelToken` the preferred new API?

No. Use `AbortController` for new code.

## Q85. What does a request interceptor return?

The updated request config, or a promise resolving to it.

## Q86. How should a rejected interceptor preserve an error?

Return `Promise.reject(error)` unless it intentionally handles or transforms the error.

## Q87. Do TypeScript generics validate server JSON?

No. They provide compile-time expectations, not runtime validation.

## Q88. Does Axios itself provide server-state caching and invalidation?

Not as a complete query-cache system. Use an explicit strategy or a tool such as RTK Query/TanStack Query when appropriate.

## Q89. Is it safe to log an Axios error object?

Not automatically. Its config may contain sensitive headers, request data, or credentials. Redact and log a safe subset.

## Q90. Should all POST requests automatically retry after a network error?

No. The server may have processed the request even if the client did not receive the response. Use idempotency keys or another backend-supported deduplication strategy for mutations.

## Q91. Does a request timeout guarantee the whole user workflow finishes within that timeout?

No. Several requests, storage reads, retries, and other processing can exceed an individual request's timeout.

## Q92. Where should token refresh logic live?

In a centralized authentication/API layer with single-flight refresh, bounded retry, secure token storage, and a clear sign-out path—not duplicated in every screen.

## Q93. What does `axios.isAxiosError()` do?

It helps narrow an `unknown` caught value to an Axios error in TypeScript so response, status, code, and message can be inspected safely.

## Q94. What causes duplicate requests in React Native?

Possible causes include repeated effects, Strict Mode development behavior, navigation remounts, multiple event handlers, retries, interceptor duplication, or query-library refetch rules. Instrument requests before deciding which it is.

## Q95. What's the most important Axios production principle?

Treat the network as unreliable and server data as untrusted. Define timeouts, cancellation, error contracts, bounded retry, data validation, safe logging, secure authentication, and meaningful loading/empty/error UI.

---

# Practical Axios Interview Checklist

Before the interview, be able to explain and implement each item without copying:

- [ ] Create separate public and authenticated Axios instances.
- [ ] Add an access token without leaking it to an untrusted URL.
- [ ] Implement a single-flight refresh Promise and prevent infinite retries.
- [ ] Use `AbortController` for a search request and handle cancellation silently.
- [ ] Prevent stale results from overwriting newer data.
- [ ] Type request parameters, bodies, and responses with TypeScript.
- [ ] Explain why those TypeScript types do not validate network JSON at runtime.
- [ ] Normalize HTTP, network, timeout, and cancellation failures.
- [ ] Implement bounded retry with backoff only for operations safe to retry.
- [ ] Upload a picked file from React Native using `FormData`.
- [ ] Diagnose `Network Error` across simulator, emulator, and physical devices.
- [ ] Explain the difference between Axios, RTK Query, and TanStack Query.
- [ ] Test services, interceptors, token refresh concurrency, and failure states.
- [ ] Redact authorization headers and personal data from logs.

# Official References

Use the official documentation for the behavior of the Axios version in your project:

- [Axios GitHub repository and README](https://github.com/axios/axios)
- [Axios request configuration](https://github.com/axios/axios/blob/v1.x/docs/pages/advanced/request-config.md)
- [Axios interceptors](https://github.com/axios/axios/blob/v1.x/docs/pages/advanced/interceptors.md)
- [Axios error handling](https://github.com/axios/axios/blob/v1.x/docs/pages/handling_errors.md)
- [Axios cancellation](https://github.com/axios/axios#cancellation)
- [Axios TypeScript examples](https://axios.rest/pages/getting-started/examples/typescript)
- [Axios security policy](https://axios.rest/pages/misc/security)

## Final preparation advice

For a 5+ years interview, do not stop at describing an Axios method. Explain the whole request lifecycle, failure modes, retry safety, authentication races, platform-specific networking, runtime validation, and how your design can be tested. Use concrete examples from projects you have actually built; do not claim production experience with a pattern you have only studied.
