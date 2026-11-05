# useToggle

Implement a `useToggle` hook that manages a boolean toggle state in React components.

## Argument(s)

defaultValue: boolean (optional): The initial value of the toggle state. If not provided, it should default to false

## Returns

A tuple containing the following elements:

- value: boolean: The current state
- toggle: () => void: A function that toggles the state between true and false
- setValue: A function that sets the state to the provided value. It has the same signature as the second argument of the useState hook

## Example(s)

```ts
export default function Component() {
  const [enabled, toggle, setEnabled] = useToggle();

  return (
    <div>
      <p>Enabled: {enabled ? 'On' : 'Off'}</p>
      <button onClick={toggle}>Toggle</button>
      <button onClick={() => setEnabled(true)}>Enable</button>
      <button onClick={() => setEnabled(false)}>Disable</button>
    </div>
  );
}
```

## Reference(s)

- React [useRef](https://react.dev/reference/react/useRef)
- React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { Dispatch, SetStateAction, useState } from "react";

export default function useToggle(
  defaultValue?: boolean,
): [boolean, () => void, Dispatch<SetStateAction<boolean>>] {
  const [enabled, setEnabled] = useState(defaultValue ?? false);

  const toggle = () => setEnabled((prev) => !prev);

  return [enabled, toggle, setEnabled] as const;
}
```

