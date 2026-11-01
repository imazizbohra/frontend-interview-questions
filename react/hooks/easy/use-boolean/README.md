# useBoolean

Implement a useBoolean hook that manages boolean state with additional convenience utility methods.

## Argument(s)

initialValue: boolean: Initial value of the boolean state. If not provided, it should default to false.


## Returns

The useBoolean hook returns an object with the following properties:

- value: boolean: The current boolean state
- setTrue: () => void: A function to set the boolean state to true
- setFalse: () => void: A function to set the boolean state to false

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

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

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
