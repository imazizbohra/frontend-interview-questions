# useTimeout

Implement a `useTimeout` hook that invokes a callback function after a specified delay.

Note that the hook can be called again with different values after the initial call:

- Different callback: The pending timer should invoke the latest callback. If the timer has already expired, changing the callback should not execute it or set a new timer
- Different delay: The previous timeout should be canceled if the timer has not expired, and a new timer is set with the new delay value

The primary benefit of `useTimeout` is that you do not have to manually call `clearTimeout()` if the component unmounts before the timer expires.

## Argument(s)

- callback: () => void: A function to be called after the specified delay
- delay: number | null: The delay in milliseconds before invoking the callback function. If null, the timeout is cleared

## Returns

Nothing.

## Notes

- The pending timeout should call the latest callback if the callback changes before the timeout fires.
- Changing only the callback should not restart the countdown.
- Passing null for delay should clear the timeout and avoid scheduling a new one.
- Clean up the timeout when the component unmounts or when delay changes.

## Example(s)

```tsx
export default function Component() {
  const [loading, setLoading] = useState(true);

  useTimeout(() => setLoading(false), 1000);

  return (
    <div>
      <p>{loading ? 'Loading' : 'Ready'}</p>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useEffect, useRef } from "react";

export default function useTimeout(callback: () => void, delay: number | null) {
  const latestCallback = useRef(callback);
  latestCallback.current = callback;

  useEffect(() => {
    if (delay === null) {
      return;
    }

    const timeoutId = setTimeout(() => {
      latestCallback.current();
    }, delay);

    return () => {
      clearTimeout(timeoutId);
    };
  }, [delay]);
}
```
