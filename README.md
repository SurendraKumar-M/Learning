# Axios Coding Interview Questions & Answers for React Native / Expo

# Axios Coding Questions and Answers

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

## Coding practice checklist

- [ ] Create an Axios instance with a base URL and timeout.
- [ ] Type an API service and its input/output contracts.
- [ ] Implement cancellation and protect against stale results.
- [ ] Implement a bounded retry policy and explain idempotency.
- [ ] Coordinate simultaneous 401 responses through a single refresh Promise.
- [ ] Upload a file with FormData on a real iOS and Android target.
- [ ] Test success, HTTP failure, network failure, timeout, and cancellation.
