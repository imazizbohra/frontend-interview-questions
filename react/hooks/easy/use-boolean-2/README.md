# useBoolean 2

This is a follow-up to `useBoolean`. In this version, the boolean behavior stays the same, but the returned methods should be memoized so that each method keeps the same function instance across re-renders.

## Argument(s)

initialValue: boolean: Initial value of the boolean state. If not provided, it should default to false.

## Returns

The `useBoolean` hook returns an object with the following properties:

- value: boolean: The current boolean state
- setTrue: () => void: A function to set the boolean state to true
- setFalse: () => void: A function to set the boolean state to false

`setTrue` and `setFalse` must be the same function instance across re-renders.

## Example(s)

```ts
export default function Component() {
  const { value, setTrue, setFalse } = useBoolean();

  return (
    <div>
      <p>{value ? 'enabled' : 'disabled'}</p>
      <button onClick={setTrue}>Enable</button>
      <button onClick={setFalse}>Disable</button>
    </div>
  );
}
```

## Reference(s)

- React [useCallback](https://react.dev/reference/react/useCallback)
- React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useCallback } from 'react';  

type UseBooleanReturn = {
  value: boolean;
  setTrue: () => void;
  setFalse: () => void;
};

export default function useBoolean(initialValue?: boolean): UseBooleanReturn {
  const [state, setState] = useState<boolean>(initialValue ?? false);

  const setTrue = useCallback(() => {
    setState(true);
  }, []);

  const setFalse = useCallback(() => {
    setState(false);
  }, []);

  const returnValue: UseBooleanReturn = {
    value: state,
    setTrue: setTrue,
    setFalse: setFalse
  };

  return returnValue;
}
```
