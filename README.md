# React Native / React / TypeScript — Coding Interview Questions & Answers (5+ Years)

A practical coding-preparation companion for senior React Native and Expo interviews. Each question includes a problem, reference solution, explanation, complexity where it makes sense, and follow-up points an interviewer may explore.

> **How to practice:** First solve each problem without looking at the answer. Explain your assumptions, edge cases, complexity, and tests aloud. In a senior interview, a correct solution is only part of the answer; clarity, robustness, and trade-offs matter too.

## Contents

1. JavaScript fundamentals and array/string problems
2. Algorithms and data transformations
3. Closures, functions, and asynchronous JavaScript
4. TypeScript coding questions
5. React hooks and component exercises
6. React Native and Expo scenarios
7. Senior-level discussion checklist

---

# 1. JavaScript Fundamentals and Array/String Problems

## Q1. Reverse a string

**Problem:** Write a function that reverses a string.

```ts
function reverseString(value: string): string {
  return [...value].reverse().join("");
}

console.log(reverseString("hello")); // "olleh"
```

**Explanation:** Convert the string into an array of Unicode code points, reverse it, then join it.

**Complexity:** `O(n)` time and `O(n)` extra space.

**Senior follow-up:** `[...value]` handles many Unicode code points better than `value.split("")`, but it does not correctly preserve every user-perceived character made from multiple code points (for example, some emoji and combining characters). For grapheme-aware reversal, use `Intl.Segmenter` where supported or a suitable Unicode library.

## Q2. Check whether a string is a palindrome

**Problem:** Ignore case, spaces, and punctuation.

```ts
function isPalindrome(value: string): boolean {
  const normalized = value.toLowerCase().replace(/[^\p{L}\p{N}]/gu, "");
  return normalized === [...normalized].reverse().join("");
}

console.log(isPalindrome("A man, a plan, a canal: Panama!")); // true
console.log(isPalindrome("React Native")); // false
```

**Explanation:** Normalize the string before comparing it with its reverse. Unicode property escapes (`\p{L}` and `\p{N}`) retain letters and numbers across many writing systems in modern JavaScript runtimes.

**Complexity:** `O(n)` time and space.

**Follow-up:** Clarify whether accents should be treated as equivalent, whether whitespace matters, and which Unicode behavior the product requires.

## Q3. Count character frequency

```ts
function characterFrequency(value: string): Map<string, number> {
  const frequency = new Map<string, number>();

  for (const character of value) {
    frequency.set(character, (frequency.get(character) ?? 0) + 1);
  }

  return frequency;
}

console.log([...characterFrequency("banana")]);
// [["b", 1], ["a", 3], ["n", 2]]
```

**Complexity:** `O(n)` time; `O(k)` space for `k` distinct characters.

**Follow-up:** Decide whether matching is case-sensitive and whether spaces/punctuation should be counted.

## Q4. Find the first non-repeating character

```ts
function firstUniqueCharacter(value: string): string | null {
  const counts = new Map<string, number>();

  for (const character of value) {
    counts.set(character, (counts.get(character) ?? 0) + 1);
  }

  for (const character of value) {
    if (counts.get(character) === 1) return character;
  }

  return null;
}

console.log(firstUniqueCharacter("swiss")); // "w"
console.log(firstUniqueCharacter("aabb")); // null
```

**Why two passes?** The first establishes frequencies; the second preserves original order to find the first unique character.

**Complexity:** `O(n)` time and `O(k)` space.

## Q5. Check whether two strings are anagrams

```ts
function areAnagrams(left: string, right: string): boolean {
  const normalize = (value: string) =>
    value.toLowerCase().replace(/[^\p{L}\p{N}]/gu, "");

  const a = normalize(left);
  const b = normalize(right);
  if (a.length !== b.length) return false;

  const counts = new Map<string, number>();

  for (const character of a) {
    counts.set(character, (counts.get(character) ?? 0) + 1);
  }

  for (const character of b) {
    const count = counts.get(character);
    if (!count) return false;
    if (count === 1) counts.delete(character);
    else counts.set(character, count - 1);
  }

  return counts.size === 0;
}

console.log(areAnagrams("Dormitory", "Dirty room!!")); // true
```

**Complexity:** `O(n + m)` time and `O(k)` space.

**Follow-up:** Explain whether normalization, accents, whitespace, and case should affect the result.

## Q6. Remove duplicates from an array while preserving order

```ts
function unique<T>(items: readonly T[]): T[] {
  return [...new Set(items)];
}

console.log(unique([1, 2, 2, 3, 1])); // [1, 2, 3]
```

**Complexity:** Expected `O(n)` time and `O(n)` space.

**Important:** `Set` compares objects by reference, not by deep structural equality. Two separate `{ id: 1 }` objects are not duplicates unless they are the same object reference.

**Follow-up — deduplicate objects by ID:**

```ts
type Identified = { id: string | number };

function uniqueById<T extends Identified>(items: readonly T[]): T[] {
  const seen = new Set<T["id"]>();
  return items.filter(item => {
    if (seen.has(item.id)) return false;
    seen.add(item.id);
    return true;
  });
}
```

## Q7. Find the second-largest distinct number

```ts
function secondLargest(values: readonly number[]): number | null {
  let largest = -Infinity;
  let second = -Infinity;

  for (const value of values) {
    if (!Number.isFinite(value)) continue;

    if (value > largest) {
      second = largest;
      largest = value;
    } else if (value < largest && value > second) {
      second = value;
    }
  }

  return second === -Infinity ? null : second;
}

console.log(secondLargest([7, 3, 7, 2, 5])); // 5
console.log(secondLargest([4, 4])); // null
```

**Complexity:** `O(n)` time and `O(1)` extra space.

**Follow-up:** State whether “second largest” means second-largest distinct value; clarify how `NaN`, infinities, and empty arrays should be handled.

## Q8. FizzBuzz

```ts
function fizzBuzz(limit: number): string[] {
  const result: string[] = [];

  for (let value = 1; value <= limit; value++) {
    if (value % 15 === 0) result.push("FizzBuzz");
    else if (value % 3 === 0) result.push("Fizz");
    else if (value % 5 === 0) result.push("Buzz");
    else result.push(String(value));
  }

  return result;
}
```

**Complexity:** `O(n)` time and `O(n)` output space.

**Follow-up:** Explain why checking divisibility by 15 first avoids the combined case being consumed by the 3 or 5 conditions.

## Q9. Flatten a nested array

```ts
type NestedArray<T> = Array<T | NestedArray<T>>;

function flatten<T>(values: NestedArray<T>): T[] {
  const result: T[] = [];

  for (const value of values) {
    if (Array.isArray(value)) {
      result.push(...flatten(value as NestedArray<T>));
    } else {
      result.push(value as T);
    }
  }

  return result;
}

console.log(flatten([1, [2, [3, 4]], 5])); // [1, 2, 3, 4, 5]
```

**Alternative:** `array.flat(Infinity)` is concise when the runtime and input types allow it.

**Complexity:** `O(n)` time for `n` total elements; recursion uses stack space proportional to nesting depth, plus the output array.

**Follow-up:** Discuss stack overflow for extremely deep input and an iterative stack-based approach.

## Q10. Split an array into chunks

```ts
function chunk<T>(values: readonly T[], size: number): T[][] {
  if (!Number.isInteger(size) || size <= 0) {
    throw new RangeError("size must be a positive integer");
  }

  const result: T[][] = [];
  for (let index = 0; index < values.length; index += size) {
    result.push(values.slice(index, index + size));
  }
  return result;
}

console.log(chunk([1, 2, 3, 4, 5], 2)); // [[1, 2], [3, 4], [5]]
```

**Use cases:** API batching, upload batches, or controlled work scheduling.

**Complexity:** `O(n)` time and `O(n)` space for the result.

---

# 2. Algorithms and Data Transformations

## Q11. Group objects by a property

```ts
function groupBy<T, K extends PropertyKey>(
  items: readonly T[],
  getKey: (item: T) => K
): Partial<Record<K, T[]>> {
  return items.reduce<Partial<Record<K, T[]>>>((groups, item) => {
    const key = getKey(item);
    (groups[key] ??= []).push(item);
    return groups;
  }, {});
}

const users = [
  { name: "Asha", role: "admin" },
  { name: "Ben", role: "user" },
  { name: "Chen", role: "admin" }
];

const grouped = groupBy(users, user => user.role);
// admin -> Asha, Chen; user -> Ben
```

**Complexity:** `O(n)` time and `O(n)` space.

**Follow-up:** Why accept a `getKey` callback instead of a property name? It works for derived keys and avoids complex generic constraints on arbitrary property paths.

## Q12. Merge two arrays of records by ID

**Problem:** The second array should override the first array's item when IDs match.

```ts
type RecordWithId = { id: string };

function mergeById<T extends RecordWithId>(
  first: readonly T[],
  second: readonly T[]
): T[] {
  const byId = new Map<string, T>();
  for (const item of first) byId.set(item.id, item);
  for (const item of second) byId.set(item.id, item);
  return [...byId.values()];
}
```

**Behavior:** Existing IDs keep their map position while later values replace their contents; new IDs are appended in encounter order.

**Complexity:** Expected `O(n + m)` time and `O(n + m)` space.

**Follow-up:** Discuss whether a matching record should be replaced or shallow-merged field by field.

## Q13. Sort products by price without mutating input

```ts
type Product = { id: string; name: string; price: number };

function sortByPrice(products: readonly Product[]): Product[] {
  return [...products].sort((a, b) => a.price - b.price);
}
```

**Why copy first?** `sort()` mutates the array. Directly sorting a prop or Redux-derived array can violate immutability expectations.

**Complexity:** Usually `O(n log n)` time and `O(n)` for the copied array.

## Q14. Implement a safe nested-property getter

```ts
function getPath(source: unknown, path: readonly (string | number)[]): unknown {
  let current: unknown = source;

  for (const key of path) {
    if (current === null || typeof current !== "object") return undefined;
    current = (current as Record<string | number, unknown>)[key];
  }

  return current;
}

const data = { account: { profile: { name: "Surendra" } } };
console.log(getPath(data, ["account", "profile", "name"])); // "Surendra"
console.log(getPath(data, ["account", "missing", "name"])); // undefined
```

**Why does it return `unknown`?** Arbitrary paths cannot safely guarantee the resulting type. Callers must validate/narrow the result before using it.

## Q15. Calculate cart totals

```ts
type CartItem = {
  productId: string;
  unitPrice: number;
  quantity: number;
};

type CartTotals = { subtotal: number; itemCount: number };

function calculateCartTotals(items: readonly CartItem[]): CartTotals {
  return items.reduce<CartTotals>(
    (totals, item) => ({
      subtotal: totals.subtotal + item.unitPrice * item.quantity,
      itemCount: totals.itemCount + item.quantity
    }),
    { subtotal: 0, itemCount: 0 }
  );
}
```

**Complexity:** `O(n)` time and `O(1)` auxiliary space, excluding the new accumulator objects created by `reduce`.

**Production follow-ups:** Validate non-negative quantities and prices; account for currency precision (integer minor units or a decimal library); distinguish subtotal, tax, shipping, discounts, and final total.

## Q16. Find the intersection of two arrays

```ts
function intersection<T>(left: readonly T[], right: readonly T[]): T[] {
  const rightSet = new Set(right);
  const emitted = new Set<T>();
  const result: T[] = [];

  for (const item of left) {
    if (rightSet.has(item) && !emitted.has(item)) {
      emitted.add(item);
      result.push(item);
    }
  }
  return result;
}

console.log(intersection([1, 2, 2, 3], [2, 3, 4])); // [2, 3]
```

**Complexity:** Expected `O(n + m)` time and `O(m + n)` space.

**Follow-up:** Clarify whether duplicates should be preserved and how object equality should work.

## Q17. Implement binary search

**Assumption:** Input is sorted in ascending order.

```ts
function binarySearch(values: readonly number[], target: number): number {
  let low = 0;
  let high = values.length - 1;

  while (low <= high) {
    const middle = low + Math.floor((high - low) / 2);
    const value = values[middle];

    if (value === target) return middle;
    if (value < target) low = middle + 1;
    else high = middle - 1;
  }

  return -1;
}
```

**Complexity:** `O(log n)` time and `O(1)` space.

**Follow-up:** What if duplicates exist? Specify whether any matching index, first occurrence, or insertion position is required.

---

# 3. Closures, Functions, and Asynchronous JavaScript

## Q18. Implement debounce

**Problem:** Only invoke the callback after calls stop for the specified delay.

```ts
function debounce<TArgs extends unknown[]>(
  callback: (...args: TArgs) => void,
  delayMs: number
): (...args: TArgs) => void {
  let timer: ReturnType<typeof setTimeout> | undefined;

  return (...args: TArgs) => {
    if (timer !== undefined) clearTimeout(timer);
    timer = setTimeout(() => callback(...args), delayMs);
  };
}
```

**Use cases:** Search input, validation, saving drafts after typing.

**Complexity:** `O(1)` work per call, excluding the callback.

**Senior follow-ups:** How would you add `cancel()` and `flush()`? How would you preserve `this`? Should the callback fire on the leading edge, trailing edge, or both? In React, who clears the timer on unmount?

## Q19. Implement a cancellable debounce

```ts
type Debounced<TArgs extends unknown[]> = ((...args: TArgs) => void) & {
  cancel: () => void;
};

function debounceWithCancel<TArgs extends unknown[]>(
  callback: (...args: TArgs) => void,
  delayMs: number
): Debounced<TArgs> {
  let timer: ReturnType<typeof setTimeout> | undefined;

  const debounced = ((...args: TArgs) => {
    if (timer !== undefined) clearTimeout(timer);
    timer = setTimeout(() => {
      timer = undefined;
      callback(...args);
    }, delayMs);
  }) as Debounced<TArgs>;

  debounced.cancel = () => {
    if (timer !== undefined) clearTimeout(timer);
    timer = undefined;
  };

  return debounced;
}
```

**React reminder:** A debounced function often needs stable identity (`useMemo` or a ref) and cleanup in an effect. Also ensure the callback sees the intended/latest values.

## Q20. Implement throttle

**Problem:** Allow a callback to run at most once per time window.

```ts
function throttle<TArgs extends unknown[]>(
  callback: (...args: TArgs) => void,
  intervalMs: number
): (...args: TArgs) => void {
  let lastRun = 0;

  return (...args: TArgs) => {
    const now = Date.now();
    if (now - lastRun < intervalMs) return;
    lastRun = now;
    callback(...args);
  };
}
```

This is a **leading-edge throttle**; calls that occur during the interval are dropped.

**Follow-up:** Implement trailing-edge behavior if the last call in the window must eventually run. Use a monotonic clock where available if clock adjustments matter.

## Q21. Implement memoization for a single primitive argument

```ts
function memoizeNumberResult(
  callback: (value: number) => number
): (value: number) => number {
  const cache = new Map<number, number>();

  return value => {
    if (cache.has(value)) return cache.get(value)!;
    const result = callback(value);
    cache.set(value, result);
    return result;
  };
}

const square = memoizeNumberResult(value => value * value);
square(4); // computes and caches 16
square(4); // returns cached 16
```

**Important limitations:** This example only handles a numeric argument and numeric result. Real memoization needs a key strategy, cache size/eviction policy, and clarity about side effects. Do not memoize impure functions blindly.

## Q22. Implement currying

```ts
const add = (a: number) => (b: number) => a + b;

const addFive = add(5);
console.log(addFive(3)); // 8
```

**Interview explanation:** Currying transforms a function that accepts multiple arguments into a chain of functions, each accepting one argument. It is useful for partial application and reusable configuration.

## Q23. Compose functions from right to left

```ts
type Unary<T> = (value: T) => T;

function compose<T>(...functions: Unary<T>[]): Unary<T> {
  return (initialValue: T) =>
    functions.reduceRight((value, fn) => fn(value), initialValue);
}

const double = (n: number) => n * 2;
const increment = (n: number) => n + 1;
const composed = compose(double, increment);
console.log(composed(3)); // 8: double(increment(3))
```

**Follow-up:** This simplified version assumes every function uses the same input/output type. A fully generic composition type is more involved.

## Q24. Retry a failed asynchronous operation

```ts
async function retry<T>(
  operation: () => Promise<T>,
  attempts: number,
  delayMs = 0
): Promise<T> {
  if (!Number.isInteger(attempts) || attempts < 1) {
    throw new RangeError("attempts must be at least 1");
  }

  let lastError: unknown;

  for (let attempt = 1; attempt <= attempts; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;
      if (attempt < attempts && delayMs > 0) {
        await new Promise<void>(resolve => setTimeout(resolve, delayMs));
      }
    }
  }

  throw lastError;
}
```

**Senior follow-ups:** Use exponential backoff and jitter for distributed systems; honor cancellation; retry only transient failures; avoid retrying non-idempotent operations without an idempotency strategy.

## Q25. Execute async tasks with limited concurrency

```ts
async function mapWithConcurrency<T, R>(
  items: readonly T[],
  concurrency: number,
  mapper: (item: T, index: number) => Promise<R>
): Promise<R[]> {
  if (!Number.isInteger(concurrency) || concurrency < 1) {
    throw new RangeError("concurrency must be a positive integer");
  }

  const results = new Array<R>(items.length);
  let nextIndex = 0;

  async function worker(): Promise<void> {
    while (true) {
      const index = nextIndex++;
      if (index >= items.length) return;
      results[index] = await mapper(items[index]!, index);
    }
  }

  const workerCount = Math.min(concurrency, items.length);
  await Promise.all(Array.from({ length: workerCount }, () => worker()));
  return results;
}
```

**Behavior:** Runs up to `concurrency` operations at once and preserves result order by writing to the original index.

**Complexity:** Total work depends on the mapper; orchestration memory is `O(n)` for output.

**Follow-up:** Decide how errors behave. As written, a worker rejection rejects `Promise.all`; other already-running tasks may still continue. Production variants may support cancellation and settled results.

## Q26. Run tasks sequentially vs concurrently

**Sequential:** Use when each operation depends on the previous result or ordering matters.

```ts
async function sequential<T>(tasks: readonly (() => Promise<T>)[]): Promise<T[]> {
  const results: T[] = [];
  for (const task of tasks) results.push(await task());
  return results;
}
```

**Concurrent:** Use when tasks are independent and resource usage is acceptable.

```ts
async function concurrent<T>(tasks: readonly (() => Promise<T>)[]): Promise<T[]> {
  return Promise.all(tasks.map(task => task()));
}
```

**Senior answer:** Concurrency is not always better. Consider rate limits, backend capacity, memory, ordering, partial failure, and cancellation.

## Q27. Build a timeout wrapper for a promise

```ts
function withTimeout<T>(promise: Promise<T>, timeoutMs: number): Promise<T> {
  return new Promise<T>((resolve, reject) => {
    const timer = setTimeout(
      () => reject(new Error(`Operation timed out after ${timeoutMs}ms`)),
      timeoutMs
    );

    promise.then(
      value => {
        clearTimeout(timer);
        resolve(value);
      },
      error => {
        clearTimeout(timer);
        reject(error);
      }
    );
  });
}
```

**Important:** This rejects the wrapper but does not automatically cancel the underlying operation. If cancellation is required, pass an `AbortSignal` to an API that supports it.

## Q28. Fetch JSON safely using `unknown`

```ts
async function fetchJson(url: string): Promise<unknown> {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  return response.json() as Promise<unknown>;
}
```

**Why `unknown` instead of `any`?** The caller must validate the response shape before trusting it.

```ts
type User = { id: string; name: string };

function isUser(value: unknown): value is User {
  if (value === null || typeof value !== "object") return false;
  const candidate = value as Record<string, unknown>;
  return typeof candidate.id === "string" && typeof candidate.name === "string";
}
```

**Follow-up:** In production, use a schema validator (for example, Zod) when richer validation, useful error messages, and nested schemas are required.

---

# 4. TypeScript Coding Questions

## Q29. Write a generic function to get the first array item

```ts
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}

const firstNumber = first([10, 20, 30]); // number | undefined
const firstName = first(["Asha", "Ben"]); // string | undefined
```

**Key point:** The generic preserves the input element type instead of returning `any`.

## Q30. Use `keyof` to safely read a property

```ts
function getProperty<T, K extends keyof T>(object: T, key: K): T[K] {
  return object[key];
}

const user = { id: 1, name: "Asha", active: true };
const name = getProperty(user, "name"); // string
const active = getProperty(user, "active"); // boolean
// getProperty(user, "email"); // compile-time error
```

**What interviewer is checking:** Generic constraints, `keyof`, and indexed access types.

## Q31. Make selected properties optional with a mapped type

```ts
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type Profile = {
  id: string;
  name: string;
  avatarUrl: string;
};

type ProfileUpdate = PartialBy<Profile, "avatarUrl">;
// id and name required; avatarUrl optional
```

**Follow-up:** Explain the roles of `Omit`, `Pick`, `Partial`, and the intersection (`&`).

## Q32. Model request status with a discriminated union

```ts
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

function describeRequest<T>(state: RequestState<T>): string {
  switch (state.status) {
    case "idle": return "Not started";
    case "loading": return "Loading";
    case "success": return "Loaded";
    case "error": return state.error.message;
  }
}
```

**Why it is better than several booleans:** It prevents contradictory states such as `isLoading === true` and `isSuccess === true`, and TypeScript narrows the fields based on `status`.

## Q33. Create a typed `Result` helper

```ts
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

async function safeCall<T>(operation: () => Promise<T>): Promise<Result<T>> {
  try {
    return { ok: true, value: await operation() };
  } catch (error) {
    return {
      ok: false,
      error: error instanceof Error ? error : new Error("Unknown error")
    };
  }
}
```

**Follow-up:** Why not catch and ignore the exception? Returning a typed result forces the caller to handle success and failure deliberately. Choose one error-handling convention consistently in the codebase.

## Q34. Type a reusable button component

```tsx
import type { ReactNode } from "react";
import { Pressable, Text, type PressableProps } from "react-native";

type AppButtonProps = Omit<PressableProps, "children"> & {
  title: string;
  icon?: ReactNode;
};

export function AppButton({ title, icon, ...pressableProps }: AppButtonProps) {
  return (
    <Pressable accessibilityRole="button" {...pressableProps}>
      {icon}
      <Text>{title}</Text>
    </Pressable>
  );
}
```

**What this demonstrates:** Reuse platform props, remove `children` from the inherited prop contract because the component controls its own content, and type custom props explicitly.

**Follow-up:** In a production design system, include disabled/loading states, accessibility labels, variants, sizing, and theme tokens.

## Q35. Type an API response and avoid trusting raw JSON

```ts
type ProductDto = {
  id: string;
  title: string;
  price: number;
};

function isProductDto(value: unknown): value is ProductDto {
  if (value === null || typeof value !== "object") return false;
  const record = value as Record<string, unknown>;
  return typeof record.id === "string"
    && typeof record.title === "string"
    && typeof record.price === "number";
}

async function loadProduct(url: string): Promise<ProductDto> {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);

  const payload: unknown = await response.json();
  if (!isProductDto(payload)) throw new Error("Invalid product response");
  return payload;
}
```

**Senior follow-up:** Keep transport DTOs separate from domain models when backend representation and UI/domain needs differ. Validation at runtime is still necessary because TypeScript types are erased at runtime.

## Q36. Type a `groupBy` utility

See Q11 for the implementation. Be prepared to explain:

- `T` represents each item type.
- `K extends PropertyKey` limits keys to strings, numbers, or symbols.
- `getKey: (item: T) => K` derives the grouping key safely.
- `Partial<Record<K, T[]>>` expresses that not every possible key must exist.

## Q37. Use `satisfies` for configuration validation

```ts
type Environment = "development" | "qa" | "production";
type Config = { apiUrl: string; enableAnalytics: boolean };

const config = {
  development: { apiUrl: "http://localhost:3000", enableAnalytics: false },
  qa: { apiUrl: "https://qa-api.example.com", enableAnalytics: true },
  production: { apiUrl: "https://api.example.com", enableAnalytics: true }
} satisfies Record<Environment, Config>;
```

**Why `satisfies`?** It checks that the object conforms to the required shape while retaining more specific inferred types than a broad annotation may retain. These URLs are illustrative placeholders; never put private credentials in client-side config.

## Q38. Type navigation route parameters

For React Navigation, route types are commonly represented with a param list:

```ts
type RootStackParamList = {
  Home: undefined;
  ProductDetails: { productId: string };
  Search: { query?: string };
};
```

A `ProductDetails` route requires a `productId`; `Home` accepts no parameters. Wire this param list to the navigator and screen props using the types for the React Navigation version in the project.

**Senior follow-up:** Handle deep links and untrusted external parameters by validating inputs at runtime rather than assuming static route types validate an incoming URL.

---

# 5. React Hooks and Component Exercises

## Q39. Create a `usePrevious` hook

```tsx
import { useEffect, useRef } from "react";

function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T | undefined>(undefined);

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}
```

**Behavior:** During a render, it returns the value saved by the previous effect, so it is useful for comparing render-to-render values.

**Follow-up:** Explain that refs persist across renders but changing `ref.current` does not itself cause a render. Be aware of effect timing when using the hook for layout-sensitive logic.

## Q40. Create a debounced-value hook

```tsx
import { useEffect, useState } from "react";

function useDebouncedValue<T>(value: T, delayMs: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delayMs);
    return () => clearTimeout(timer);
  }, [value, delayMs]);

  return debouncedValue;
}
```

Example:

```tsx
const [query, setQuery] = useState("");
const debouncedQuery = useDebouncedValue(query, 300);

useEffect(() => {
  if (debouncedQuery.trim()) {
    // Start search for debouncedQuery.
  }
}, [debouncedQuery]);
```

**Important:** Debouncing reduces the frequency of work; it does not cancel an API request that has already started. Add request cancellation or stale-response protection when needed.

## Q41. Fetch data with cleanup and stale-response protection

```tsx
import { useEffect, useState } from "react";

type User = { id: number; name: string };

type State =
  | { status: "loading" }
  | { status: "success"; user: User }
  | { status: "error"; message: string };

function useUser(userId: number): State {
  const [state, setState] = useState<State>({ status: "loading" });

  useEffect(() => {
    const controller = new AbortController();
    let active = true;
    setState({ status: "loading" });

    async function load() {
      try {
        const response = await fetch(`https://example.com/users/${userId}`, {
          signal: controller.signal
        });
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const user = (await response.json()) as User;
        if (active) setState({ status: "success", user });
      } catch (error) {
        if (!active || controller.signal.aborted) return;
        setState({
          status: "error",
          message: error instanceof Error ? error.message : "Unknown error"
        });
      }
    }

    void load();
    return () => {
      active = false;
      controller.abort();
    };
  }, [userId]);

  return state;
}
```

**Important limitation:** This example assumes the response matches `User`; production code should validate runtime data as in Q35. Replace the example URL with your API endpoint.

**Follow-up:** For larger applications, prefer a server-state library such as RTK Query or another established query/cache layer instead of reimplementing caching, deduplication, refetching, and invalidation for every endpoint.

## Q42. Implement a search filter without mutating data

```tsx
import { useMemo, useState } from "react";
import { FlatList, Text, TextInput, View } from "react-native";

type Item = { id: string; name: string };

function SearchableList({ items }: { items: readonly Item[] }) {
  const [query, setQuery] = useState("");

  const filteredItems = useMemo(() => {
    const normalizedQuery = query.trim().toLowerCase();
    if (!normalizedQuery) return items;
    return items.filter(item => item.name.toLowerCase().includes(normalizedQuery));
  }, [items, query]);

  return (
    <View>
      <TextInput value={query} onChangeText={setQuery} placeholder="Search" />
      <FlatList
        data={filteredItems}
        keyExtractor={item => item.id}
        renderItem={({ item }) => <Text>{item.name}</Text>}
      />
    </View>
  );
}
```

**Follow-up:** `useMemo` is useful only if the calculation/reference behavior warrants it. For remote search, debounce requests and cancel or ignore stale responses.

## Q43. Implement load-more pagination safely

A simplified pattern:

```tsx
import { useCallback, useRef, useState } from "react";

type Page<T> = { items: T[]; nextPage: number | null };

function usePaginatedItems<T>(fetchPage: (page: number) => Promise<Page<T>>) {
  const [items, setItems] = useState<T[]>([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const inFlight = useRef(false);

  const loadMore = useCallback(async () => {
    if (inFlight.current || !hasMore) return;
    inFlight.current = true;
    setLoading(true);

    try {
      const result = await fetchPage(page);
      setItems(previous => [...previous, ...result.items]);
      setPage(result.nextPage ?? page + 1);
      setHasMore(result.nextPage !== null);
    } finally {
      inFlight.current = false;
      setLoading(false);
    }
  }, [fetchPage, hasMore, page]);

  return { items, loading, hasMore, loadMore };
}
```

**Important:** Adapt the meaning of `nextPage` to the API contract. This sample assumes `null` means no next page and that `fetchPage` identity is stable. Consider error state, retry behavior, duplicate-page prevention, refresh/reset behavior, and cancellation in production. Many query libraries already support infinite queries.

## Q44. Avoid rerendering every row in a product list

```tsx
import { memo, useCallback } from "react";
import { FlatList, Pressable, Text } from "react-native";

type Product = { id: string; name: string; price: number };

const ProductRow = memo(function ProductRow({
  item,
  onPress
}: {
  item: Product;
  onPress: (id: string) => void;
}) {
  return (
    <Pressable onPress={() => onPress(item.id)}>
      <Text>{item.name}</Text>
      <Text>{item.price.toFixed(2)}</Text>
    </Pressable>
  );
});

function ProductList({ products, openProduct }: {
  products: Product[];
  openProduct: (id: string) => void;
}) {
  const renderItem = useCallback(
    ({ item }: { item: Product }) => <ProductRow item={item} onPress={openProduct} />,
    [openProduct]
  );

  return (
    <FlatList
      data={products}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
}
```

**Why this may help:** `ProductRow` can skip a render when its `item` and `onPress` references are unchanged. It is not a guarantee: if each item object is recreated, the row props may still differ.

**Senior follow-up:** Profile first; keep row components light; avoid mutating item objects; use stable IDs; consider `getItemLayout` for fixed row sizes; optimize images and selectors; tune list virtualization only based on evidence.

## Q45. Create a typed cart reducer

```ts
type CartItem = { productId: string; title: string; price: number; quantity: number };

type CartAction =
  | { type: "add"; item: Omit<CartItem, "quantity"> }
  | { type: "remove"; productId: string }
  | { type: "setQuantity"; productId: string; quantity: number }
  | { type: "clear" };

function cartReducer(state: CartItem[], action: CartAction): CartItem[] {
  switch (action.type) {
    case "add": {
      const existing = state.find(item => item.productId === action.item.productId);
      if (existing) {
        return state.map(item => item.productId === action.item.productId
          ? { ...item, quantity: item.quantity + 1 }
          : item);
      }
      return [...state, { ...action.item, quantity: 1 }];
    }
    case "remove":
      return state.filter(item => item.productId !== action.productId);
    case "setQuantity":
      if (!Number.isInteger(action.quantity) || action.quantity < 1) {
        return state.filter(item => item.productId !== action.productId);
      }
      return state.map(item => item.productId === action.productId
        ? { ...item, quantity: action.quantity }
        : item);
    case "clear":
      return [];
  }
}
```

**What to discuss:** Immutability, union action types, quantity validation, inventory limits, persistence, derived totals, and whether Redux Toolkit is warranted. In Redux Toolkit reducers, mutation-style syntax is safe inside `createSlice` because Immer manages immutable updates.

## Q46. Build an accessible loading/error/content UI

```tsx
import { ActivityIndicator, Text, View } from "react-native";
import type { ReactNode } from "react";

type ScreenState<T> =
  | { status: "loading" }
  | { status: "error"; message: string }
  | { status: "success"; data: T };

function ContentState<T>({
  state,
  renderData
}: {
  state: ScreenState<T>;
  renderData: (data: T) => ReactNode;
}) {
  switch (state.status) {
    case "loading":
      return <ActivityIndicator accessibilityLabel="Loading" />;
    case "error":
      return <Text accessibilityRole="alert">{state.message}</Text>;
    case "success":
      return <View>{renderData(state.data)}</View>;
  }
}
```

**Interview note:** This uses a discriminated union to make UI states explicit. Add retry actions, empty states, skeletons, and accessibility semantics according to the screen's requirements.

---

# 6. React Native and Expo Scenario Coding Questions

## Q47. Prevent a search screen from showing an older response

**Problem:** The user searches for `ca`, then `cat`; the slower `ca` response arrives last and incorrectly overwrites `cat` results.

**Approach A — Abort the old request:** Use an `AbortController` where the HTTP client supports it, abort the previous request when the query changes, and ignore abort errors.

**Approach B — Request sequence number:**

```ts
let latestRequestId = 0;

async function search(query: string) {
  const requestId = ++latestRequestId;
  const results = await fetchSearchResults(query);
  if (requestId !== latestRequestId) return; // ignore stale result
  showResults(results);
}
```

`fetchSearchResults` and `showResults` are application functions to implement.

**Senior follow-up:** Also consider caching by query, debounce, empty queries, network errors, cancellation on unmount, and whether the data-fetching layer already handles stale requests.

## Q48. Calculate a safe responsive width

```tsx
import { useWindowDimensions } from "react-native";

function useContentWidth(horizontalPadding = 16) {
  const { width } = useWindowDimensions();
  return Math.max(0, width - horizontalPadding * 2);
}
```

**Why `useWindowDimensions`?** It updates when the window dimensions or font scale changes, making it more appropriate than reading window dimensions once at module initialization for responsive UI.

**Follow-up:** Consider split-screen, rotation, tablets, safe areas, and accessibility font scaling.

## Q49. Convert an API error into a user-safe message

```ts
function getUserFacingError(error: unknown): string {
  if (error instanceof Error) {
    if (error.name === "AbortError") return "Request cancelled.";
    return "Something went wrong. Please try again.";
  }
  return "An unexpected error occurred.";
}
```

**Important:** Log technical diagnostics through an approved logging/crash-reporting system, but do not expose tokens, private server messages, or sensitive data in user-facing errors.

## Q50. Prevent duplicate submit actions

```tsx
import { useState } from "react";
import { Button, View } from "react-native";

function SaveButton({ save }: { save: () => Promise<void> }) {
  const [saving, setSaving] = useState(false);

  const handleSave = async () => {
    if (saving) return;
    setSaving(true);
    try {
      await save();
    } finally {
      setSaving(false);
    }
  };

  return (
    <View>
      <Button title={saving ? "Saving…" : "Save"} disabled={saving} onPress={() => void handleSave()} />
    </View>
  );
}
```

**Senior caveat:** UI-level disabling reduces accidental duplicates, but it does not guarantee exactly-once backend processing. For payments/orders, use server-side idempotency keys and authoritative backend validation.

## Q51. Build a safe app configuration resolver

```ts
type AppAction = "OPEN_PRODUCTS" | "OPEN_PROFILE";

type HomeBlock =
  | { type: "banner"; title: string; imageUrl: string; action: AppAction }
  | { type: "productCarousel"; title: string; productIds: string[] }
  | { type: "categoryGrid"; categoryIds: string[] };

function isAppAction(value: unknown): value is AppAction {
  return value === "OPEN_PRODUCTS" || value === "OPEN_PROFILE";
}
```

**Architecture:** Validate remote configuration at runtime, map known `type` values to an allowlisted component registry, and map known actions to internal navigation behavior. Never evaluate arbitrary JavaScript received from a server.

**Senior follow-up:** Version the schema, define defaults/fallbacks, handle unknown block types, validate URLs, control rollout with feature flags, and track config failures.

## Q52. Write a test for a pure utility

Example utility:

```ts
export function formatPrice(amount: number, currency = "USD"): string {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency
  }).format(amount);
}
```

Example Jest tests:

```ts
import { formatPrice } from "./formatPrice";

describe("formatPrice", () => {
  it("formats USD amounts", () => {
    expect(formatPrice(12)).toBe("$12.00");
  });

  it("supports another currency", () => {
    expect(formatPrice(12, "EUR")).toContain("12.00");
  });
});
```

**Note:** Currency formatting can vary by runtime/locale data. Prefer stable locale configuration and assertions aligned to the project's supported environments.

## Q53. What is the difference between a unit test and a component behavior test?

**Unit test:** Verifies a small isolated function or module, for example `calculateCartTotals`.

**Component behavior test:** Renders a component, interacts with it, and asserts visible behavior, for example entering a search query and seeing matching products.

**Senior answer:** Prefer tests that verify public behavior. Mock external boundaries such as network, storage, and native device capabilities, but avoid mocking so much internal implementation that the test cannot catch real regressions.

---

# 7. More Questions to Practice Without Looking at Solutions

Use these as timed exercises. For each, explain edge cases and time/space complexity.

1. Implement `once(fn)` so the wrapped function runs only once.
2. Implement `pipe(fn1, fn2, fn3)` from left to right.
3. Write a function that compares two shallow objects.
4. Find the most frequent element in an array.
5. Find all pairs whose sum equals a target.
6. Move all zeroes to the end while preserving other item order.
7. Rotate an array by `k` positions.
8. Find the longest substring without repeated characters.
9. Validate balanced brackets using a stack.
10. Implement a queue using two stacks.
11. Implement a basic LRU cache and explain its data structures.
12. Create a function that retries only on selected HTTP status codes.
13. Implement `Promise.all` and explain how rejection and ordering work.
14. Build a request deduplication layer for identical GET requests.
15. Implement a cache with a time-to-live (TTL) and explain stale entries.
16. Create a generic `pick<T, K extends keyof T>()` function.
17. Create a type-safe event map for analytics event names and payloads.
18. Model a payment flow with a discriminated union.
19. Implement a typed form field component with validation errors.
20. Build a custom hook that tracks online/offline state and cleans up subscriptions.
21. Implement pull-to-refresh and pagination without duplicating items.
22. Implement optimistic cart updates and rollback on API failure.
23. Write a selector that derives cart subtotal without mutating state.
24. Debug an effect that causes an infinite API request loop.
25. Explain how you would profile a slow `FlatList` before optimizing it.
26. Design a typed route parameter model for nested navigators.
27. Build a runtime validator for a remote feature-flag payload.
28. Prevent a stale token refresh response from overwriting a newer session.
29. Handle a failed image request with a fallback image and accessible text.
30. Design a test strategy for a login flow, cart, and checkout.

---

# 8. Senior-Level Coding Interview Checklist

When you receive a coding question, use this sequence:

1. **Clarify requirements:** Input constraints, output, sorting/order, duplicates, nullability, and failure behavior.
2. **Give a simple correct solution first:** Make assumptions explicit.
3. **Discuss complexity:** Time and space, including output storage.
4. **Cover edge cases:** Empty input, one item, duplicate values, invalid arguments, rejected promises, cancellation, and stale data.
5. **Improve only where justified:** Explain the trade-off instead of adding abstractions automatically.
6. **Write tests:** Include typical cases, boundaries, and failure cases.
7. **Connect to mobile production:** Consider responsiveness, lifecycle cleanup, memory, accessibility, secure storage, offline behavior, and backend consistency.

## Common edge cases to mention

- Empty and single-element arrays
- Duplicate values and stable ordering
- `null`, `undefined`, and unexpected API values
- Unicode and locale-sensitive text
- Invalid page/size/count parameters
- Rejected requests and timeouts
- Stale asynchronous responses
- Component unmount during async work
- Duplicate submit events
- Runtime validation of server data
- Accessibility and loading/error/empty states
- Offline mode and retry semantics

## What interviewers assess beyond the final code

- Does the candidate clarify ambiguous requirements?
- Can they explain each important line?
- Do they understand closures, immutability, async scheduling, and type narrowing?
- Can they distinguish UI state from server state?
- Do they handle cleanup, cancellation, and stale results?
- Do they know when a concise built-in is preferable to custom code?
- Can they reason about production failure modes and test the solution?

## Final practice rule

Do not memorize these implementations line by line. Re-solve each from a blank file, narrate the reasoning, test edge cases, and then compare your solution. A 5+ years answer demonstrates not only that the code works, but also that you understand its assumptions, complexity, and behavior under production conditions.
