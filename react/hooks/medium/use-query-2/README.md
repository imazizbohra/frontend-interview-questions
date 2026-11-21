# useQuery II

This is a follow-up to [useQuery](../use-query/), where each hook instance manages its own requests. When multiple components need the same data, fetching it independently can result in duplicate requests.

Data fetching libraries such as `SWR`, `TanStack Query`, and `fate` let components share cached data.

Implement `useQuery(queryKey, fn)` to share pending requests and cache successful results across hook instances. The string `queryKey`, such as `"user:42"`, replaces the dependency array from the first question and identifies which data to fetch or reuse.

Both components use the key "user-name:42", so only one fetch should start. Both receive its result. Mounting another component with that key after success should reuse the cached result.

## Argument(s)

- `queryKey: string`: The cache key. Include every input that changes the requested data. Equal strings identify the same query; different strings identify independent queries.
- `fn: () => Promise<T>`: A function that produces the data. Callers using the same key must request interchangeable data of the same type. The first caller's function starts a missing query.

Changing `queryKey` selects the corresponding cached or new request. Changing only the identity of fn must not start another request.

## Returns

The hook returns an object that has different properties depending on the state of the promise.

- Pending:
  - `status: 'loading'`: The promise is still pending
- Rejected:
  - `status: 'error'`: The promise was rejected
  - `error: unknown`: The value that caused the promise to be rejected
- Fulfilled:
  - `status: 'success'`: The promise was resolved
  - `data`: The data returned when the promise from fn resolves

Starting to observe a different query resets the state to `{ status: 'loading' }`, clearing previous data or errors. Cached results may be delivered asynchronously.

## Requirements

- Use a module-level cache shared by all instances of the hook.
- When a key is missing from the cache, call `fn()` once and immediately store the pending promise in the cache. Other hook instances requesting that key while the promise is pending must share it without calling `fn()` again.
- Keep successful results for the lifetime of the module, including after every consumer unmounts.
- Remove failed requests from the cache. A later mount or a change back to that key can try again. Existing consumers keep their error state until they select another key or remount; there are no automatic retries.
- Ignore late results for a hook that changed keys or unmounted. Cleaning up one consumer must not cancel or evict a request shared with other consumers.

## Notes

- Assume a client-side React application with one instance of this module.
- Cache expiration, invalidation, background refetching, cancellation, and persistence across page reloads are outside this question's scope.
- A Context-based cache is an optional alternative covered in the solution; a provider is not required for this implementation.

## Example(s)

```tsx
function UserName({ userId }) {
  const request = useQuery(`user-name:${userId}`, async () => {
    const response = await fetch(`/api/users/${userId}`);
    const user = await response.json();
    return user.name;
  });

  if (request.status === 'loading') {
    return <p>Loading...</p>;
  }

  if (request.status === 'error') {
    return <p>Could not load the user.</p>;
  }

  return <p>{request.data}</p>;
}

export default function App() {
  return (
    <>
      <UserName userId="42" />
      <UserName userId="42" />
    </>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { DependencyList, useState, useEffect, useCallback } from "react";

type AsyncState<T> =
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: unknown };

const cache = new Map<string, Promise<unknown>>();

export default function useQuery<T>(
  queryKey: string,
  fn: () => Promise<T>,
): AsyncState<T> {
  const [queryState, setQueryState] = useState<AsyncState<T>>({
    status: "loading",
  });

  const resolve = useCallback((result: T) => {
    setQueryState({ status: "success", data: result });
  }, []);

  const reject = useCallback((error: Error) => {
    setQueryState({ status: "error", error: error });
  }, []);

  useEffect(() => {
    let ignore = false;
    setQueryState({ status: "loading" });

    // cache hit
    if (cache.has(queryKey)) {
      const promise = cache.get(queryKey) as Promise<T>;

      promise.then(resolve).catch(reject);
      return;
    }

    try {
      const promise = fn();
      promise
        .then((result: T) => {
          if (!ignore) {
            resolve(result);
          }
        })
        .catch((error: Error) => {
          if (!ignore) {
            reject(error);
          }

          cache.delete(queryKey);
        });

      cache.set(queryKey, promise);
    } catch (error: unknown) {
      if (error instanceof Error) {
        reject(error);
      }
    }

    return () => {
      ignore = true;
    };
  }, [queryKey]);

  return queryState;
}
```
