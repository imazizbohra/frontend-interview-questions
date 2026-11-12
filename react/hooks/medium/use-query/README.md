# useQuery

Implement a `useQuery` hook that manages a promise that can be used to fetch data.

## Argument(s)

- `fn: () => Promise`: A function that returns a promise
- `deps: DependencyList`: An array of dependencies, similar to the second argument of `useEffect`. Unlike `useEffect`, this defaults to `[]`

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

## Notes

- The hook should return `{ status: 'loading' }` when it starts a new request.
- Changing `deps` should start a new request and ignore stale results from earlier requests.
- Unmounting should prevent pending promise callbacks from updating state.
- The hook does not need caching, retries, request deduplication, or cancellation of the underlying async operation.

## Example(s)

```tsx
export default function Component({ param }) {
  const request = useQuery(async () => {
    const response = await getDataFromServer(param);
    return response.data;
  }, [param]);

  if (request.status === 'loading') {
    return <p>Loading...</p>;
  }

  if (request.status === 'error') {
    const message =
      request.error instanceof Error
        ? request.error.message
        : String(request.error);

    return <p>Error: {message}</p>;
  }

  return <p>Data: {request.data}</p>;
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import {
  DependencyList,
  useState,
  useEffect,
  useCallback,
  useRef,
} from "react";

type AsyncState<T> =
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

export default function useQuery<T>(
  fn: () => Promise<T>,
  deps: DependencyList = [],
): AsyncState<T> {
  const mountRef = useRef<boolean>(true);
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
    if (mountRef.current) {
      mountRef.current = false;
    } else {
      setQueryState({ status: "loading" });
    }

    const controller = new AbortController();
    const signal = controller.signal;

    new Promise<T>((resolve, reject) => {
      let isAborted = false;

      signal.addEventListener("abort", () => {
        isAborted = true;
      });

      fn()
        .then((result: T) => {
          if (isAborted) return;

          resolve(result);
        })
        .catch((error: Error) => {
          if (isAborted) return;

          reject(error);
        });
    }).then(resolve, reject);

    return () => controller.abort();
  }, deps);

  return queryState;
}
```
