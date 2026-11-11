# useDebounce

Implement a `useDebounce` hook that delays state updates until a specified `delay` has passed without any further changes to the provided `value`.

The observable outcome of using `useDebounce` is quite similar to React's `useDeferredValue`; the former returns an updated value after a fixed duration, while the latter always returns the updated value but relies on React's priority system for DOM updates.

## Argument(s)

- value: The value to debounce
- delay: number: The delay in milliseconds

## Returns

The hook returns the debounced value.

## Notes

- The returned value should start as the input value.
- When value changes, wait for delay milliseconds before publishing the new value.
- If value changes again before the delay expires, cancel the previous timeout and restart the wait.
- Clean up the pending timeout when the hook unmounts.

## Example(s)

```tsx
export default function Component() {
  const [keyword, setKeyword] = useState('');
  const debouncedKeyword = useDebounce(keyword, 1000);

  return (
    <div>
      <input value={keyword} onChange={(e) => setKeyword(e.target.value)} />
      <p>Debounced keyword: {debouncedKeyword}</p>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useEffect, useRef } from "react";

export default function useDebounce<T>(value: T, delay: number) {
  const [state, setState] = useState<T>(value);
  const mount = useRef<boolean>(true);
  const timerIdRef = useRef<ReturnType<typeof setTimeout>>(undefined);

  useEffect(() => {
    // ignore on 1st mount
    if (mount.current) {
      mount.current = false;
      return;
    }

    timerIdRef.current = setTimeout(() => {
      setState(value);
    }, delay);

    return () => clearTimeout(timerIdRef.current);
  }, [value]);

  return state;
}
```
