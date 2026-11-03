# useStateWithReset

Implement a `useStateWithReset` hook that is similar to `useState` but with an additional reset function that resets the state to its initial value.

## Argument(s)

initialValue: The initial value of the state. This argument should be the same as the first argument of the `useState` hook

## Returns

The `useStateWithReset` hook should have the same return values as the `useState` hook, plus an additional function that resets the state to initialValue.

## Example(s)

```tsx
export default function Component() {
  const [value, setValue, resetValue] = useStateWithReset(10);

  return (
    <div>
      <div>Value: {value}</div>
      <input onChange={(e) => setValue(e.target.value)} />
      <button onClick={resetValue}>reset</button>
    </div>
  );
}
```

## Notes

Like `useState`, initialValue can be a value or a lazy initializer function.
Resolve the initial value only on the first render. Later initialValue arguments do not change the reset baseline, and resetting should not call a lazy initializer again.

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useRef, useCallback } from "react";

export default function useStateWithReset<T>(
  initialStateOrInitializer?: T | (() => T),
) {
  const [state, setState] = useState(initialStateOrInitializer);
  const initialState = useRef(state);

  const resetState = useCallback(() => {
    setState(initialState.current);
  }, []);

  return [state, setState, resetState] as const;
}
```
