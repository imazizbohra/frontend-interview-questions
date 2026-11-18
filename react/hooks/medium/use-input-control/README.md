# useInputControl

Implement a `useInputControl` hook that manages a controlled input value and tracks additional form input states like:

| Property | Tracks | When it becomes true | When it becomes false |
| --- | --- | --- | --- |
| Touched | Whether the input has been focused and then blurred | When the user blurs the input (focus → blur) | Never resets automatically |
| Dirty | Whether the value has changed at least once | When the user types something | Never resets automatically |
| Different | Whether the value is different from the original | When the value is different from the initial value | When the value is the same as the initial value |

The handleX functions returned by the hook are meant to be passed to the relevant `<input>` event handlers for the hook to work as intended.

## Argument(s)

`initialValue: string`: The initial value of the input

## Returns

The hook returns an object with the following properties:

- `value: string`: The current value of the input
- `dirty: boolean`: Whether the input value has been modified at least once
- `touched: boolean`: Whether the input was focused and blurred
- `different: boolean`: Whether the value is different from the initial value
- `handleChange: (event: React.ChangeEvent<HTMLInputElement>) => void`: A function that updates the value of the input
- `handleBlur: () => void`: A function that should be called when the input is blurred
- `reset: () => void`: A function that resets the input value and all flags back to their initial state

## Notes

- Compare `different` against the initial value from the first render, not against later `initialValue` arguments.
- `dirty` stays `true` once the user has changed the value, even if the value later matches the initial value again.
- `touched` becomes `true` after blur and does not reset unless `reset()` is called.
- `reset()` should restore the first-render value and clear both `dirty` and `touched`.

## Example(s)

```tsx
export default function Component() {
  const nameInput = useInputControl('Oliver');

  return (
    <form>
      <div>
        <label htmlFor="name">Name</label>
        <input
          id="name"
          value={nameInput.value}
          onChange={nameInput.handleChange}
          onBlur={nameInput.handleBlur}
        />
      </div>
      <p>Touched: {nameInput.touched.toString()}</p>
      <p>Dirty: {nameInput.dirty.toString()}</p>
      <p>Different: {nameInput.different.toString()}</p>
      <button type="submit" disabled={!nameInput.different}>
        Submit
      </button>
      <button type="button" onClick={nameInput.reset}>
        Reset
      </button>
    </form>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { ChangeEvent, useState, useCallback, useRef } from "react";

interface UseInputValueReturn {
  value: string;
  dirty: boolean;
  touched: boolean;
  different: boolean;
  handleChange: (event: ChangeEvent<HTMLInputElement>) => void;
  handleBlur: () => void;
  reset: () => void;
}

interface InputState {
  value: string;
  dirty: boolean;
  touched: boolean;
}

export default function useInputControl(
  initialValue: string,
): UseInputValueReturn {
  const initialValueRef = useRef<string>(initialValue);
  const [state, setState] = useState<InputState>({
    value: initialValue,
    dirty: false,
    touched: false,
  });

  const handleChange = useCallback((event: ChangeEvent<HTMLInputElement>) => {
    const { target } = event;

    setState((prevState) => {
      return {
        ...prevState,
        value: target.value,
        dirty: true,
      };
    });
  }, []);

  const handleBlur = useCallback(() => {
    setState((prevState) => {
      return { ...prevState, touched: true };
    });
  }, []);

  const reset = useCallback(() => {
    setState({
      value: initialValue,
      dirty: false,
      touched: false,
    });
  }, []);

  return {
    value: state.value,
    dirty: state.dirty,
    touched: state.touched,
    different: initialValueRef.current !== state.value,
    handleChange: handleChange,
    handleBlur: handleBlur,
    reset: reset,
  };
}
```
