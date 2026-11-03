# useDefault

Implement a `useDefault` hook that returns the default value when state is `null` or `undefined`.

## Argument(s)

- defaultValue: The default value to be returned when the state is null or undefined
- initialValue: The initial value of the state. This argument should be the same as the first argument of the `useState` hook

## Returns

The `useDefault` hook returns the same values as the `useState` hook.

## Example(s)

```tsx
export default function Component() {
  const initialUser = { name: 'Marshall' };
  const defaultUser = { name: 'Mathers' };
  const [user, setUser] = useDefault(defaultUser, initialUser);

  return (
    <div>
      <div>User: {user.name}</div>
      <input onChange={(e) => setUser({ name: e.target.value })} />
      <button onClick={() => setUser(null)}>reset</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, Dispatch, SetStateAction } from "react";

export default function useDefault<TStateType>(
  defaultValue: TStateType,
  initialValue: TStateType | (() => TStateType),
): readonly [
  TStateType,
  Dispatch<SetStateAction<TStateType | undefined | null>>,
] {
  const [state, setState] = useState<TStateType | undefined | null>(
    initialValue,
  );

  if (state === undefined || state === null) {
    return [defaultValue, setState] as const;
  }

  return [state, setState] as const;
}
```
