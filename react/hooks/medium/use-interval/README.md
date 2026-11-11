# useInterval

Implement a `useInterval` hook that creates an interval that invokes a callback function at a specified delay.

## Argument(s)

- `callback: () => void`: A function to be called at the specified interval
- `delay: number | null`: The delay in milliseconds between each invocation of the callback function. If `null`, the interval is cleared

## Returns

Nothing.

## Notes

- The interval should call the latest `callback` without recreating the interval solely because the callback identity changed.
- Passing `null` for delay should clear the interval and avoid scheduling a new one.
- The first `callback` invocation happens after the first interval `delay`, not immediately.
- Clean up the interval when the component unmounts or when `delay` changes.

## Example(s)

```tsx
export default function Component() {
  const [count, setCount] = useState(0);

  useInterval(() => setCount(count + 1), 1000);

  return (
    <div>
      <p>{count}</p>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useRef, useEffect } from "react";

export default function useInterval(
  callback: () => void,
  delay: number | null,
) {
  const callbackRef = useRef<() => void>(callback);
  callbackRef.current = callback;

  useEffect(() => {
    if (delay == null) return;

    const timerId = setInterval(() => {
      callbackRef.current();
    }, delay);

    return () => clearInterval(timerId);
  }, [delay]);
}
```
