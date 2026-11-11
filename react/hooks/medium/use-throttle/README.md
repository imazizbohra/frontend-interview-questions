# useThrottle

Implement a `useThrottle` hook that returns a throttled version of value that updates at most once every interval milliseconds.

## Argument(s)

- `value`: The value to throttle
- `interval`: number: The interval in milliseconds. This defaults to 500 milliseconds

## Returns

The hook returns the throttled value.

## Notes

- The first render should return the input `value` immediately.
- Later updates should publish at most once per `interval`.
- If several values arrive during the same throttle window, the trailing update should publish the latest one.
- Clean up any pending timeout when the hook unmounts.

## Example(s)

```tsx
export default function Component() {
  const [position, setPosition] = useState([0, 0]);
  const throttledPosition = useThrottle(position, 1000);

  return (
    <div
      style={{
        position: 'fixed',
        top: throttledPosition[0],
        left: throttledPosition[1],
      }}
      onMouseMove={(e) => setPosition([e.clientY, e.clientX])}
    />
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useEffect, useRef } from "react";

export default function useThrottle<T>(value: T, interval = 500) {
  const [state, setState] = useState<T>(value);
  const latestValueRef = useRef<T>(value);
  const lastPublishedAt = useRef(Date.now());
  const mountRef = useRef<boolean>(true);
  const timeroutIdRef = useRef<ReturnType<typeof setTimeout>>(undefined);

  useEffect(() => {
    // record latest value
    latestValueRef.current = value;

    // skip on first mount
    if (mountRef.current) {
      mountRef.current = false;
      return;
    }

    const publish = () => {
      lastPublishedAt.current = Date.now();
      setState(latestValueRef.current);
    };

    // find remaining time
    const remaining = interval - (Date.now() - lastPublishedAt.current);
    if (remaining <= 0) {
      publish();
      return;
    }

    timeroutIdRef.current = setTimeout(() => {
      publish();
    }, remaining);

    return () => clearTimeout(timeroutIdRef.current);
  }, [value]);

  return state;
}
```
