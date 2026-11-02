# useCounter

Implement a `useCounter` hook that manages counter state with additional convenience utility methods.

## Argument(s)

initialValue: number: Initial value of the counter state. If not provided, it should default to 0.

## Returns

The useCounter hook returns an object with the following properties:

- count: number: The current counter value
- increment: () => void: A function to increment the counter value
- decrement: () => void: A function to decrement the counter value
- reset: () => void: A function to reset the counter value to initialValue, or 0 if not provided
- setCount: A function with the same behavior as the setter returned by useState; it accepts either a number or an updater function (previousCount: number) => number

## Example(s)

```ts
export default function Component() {
  const { count, increment, decrement, reset, setCount } = useCounter();

  return (
    <div>
      <p>Counter: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useCallback, Dispatch, SetStateAction } from "react";

interface UseCounterReturn {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  setCount: Dispatch<SetStateAction<number>>;
}

export default function useCounter(initialValue?: number): UseCounterReturn {
  const [state, setState] = useState<number>(initialValue ?? 0);

  const incrementState = useCallback(() => {
    setState((prev) => prev + 1);
  }, []);

  const decrementState = useCallback(() => {
    setState((prev) => prev - 1);
  }, []);

  const resetState = useCallback(() => {
    setState(initialValue ?? 0);
  }, []);

  const setCounterState: Dispatch<SetStateAction<number>> = useCallback(
    (stateAction: SetStateAction<number>) => {
      setState(stateAction);
    },
    [],
  );

  const returnValue = {
    count: state,
    increment: incrementState,
    decrement: decrementState,
    reset: resetState,
    setCount: setCounterState,
  };

  return returnValue;
}
```
