# useCycle

Implement a `useCycle` hook that cycles through a sequence of values each time the returned function is called.

## Argument(s)

The `useCycle` hook should accept an indefinite number of arguments, each representing a value in the sequence to cycle through.

## Returns

A tuple containing the following elements:

- value: The current value
- cycle: A function that changes the current value to the next one in the sequence, or the first one if the current value is the last in the sequence

## Example(s)

```tsx
export default function Component() {
  const [mode, cycle] = useCycle('low', 'medium', 'high');

  return (
    <div>
      <p>State: {mode}</p>
      <button onClick={cycle}>Cycle</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useCallback } from "react";

export default function useCycle<T>(...args: T[]) {
  const [state, setState] = useState(0);

  const cycle = useCallback(() => {
    setState((prev) => (prev + 1) % args.length);
  }, []);

  if (args.length === 0) {
    return [undefined, undefined] as const;
  }

  return [args[state], cycle] as const;
}
```
