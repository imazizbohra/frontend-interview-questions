# usePrevious

Implement a `usePrevious` hook that returns the previous distinct state value. Unrelated re-renders with the same state value should not advance the previous value.

## Argument(s)

state: The state to track. This should be a generic value of any type

## Returns

The `usePrevious` hook should return the previous distinct value of the state argument, or undefined before the first change.

## Example(s)

```ts
export default function Component() {
  const [count, setCount] = useState(0);
  const previousCount = usePrevious(count);

  return (
    <div>
      <p>current: {count}</p>
      <p>previous: {previousCount}</p>
      <button onClick={() => setCount((count) => count + 1)}>Increase</button>
      <button onClick={() => setCount((count) => count - 1)}>Decrease</button>
    </div>
  );
}
```

## Reference(s)

- React [useEffect](https://react.dev/reference/react/useEffect)
- React [useRef](https://react.dev/reference/react/useRef)
- React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
// Approach 1

import { useRef } from "react";

export default function usePrevious<T>(newState: T): T | undefined {
  const prevState = useRef<T | undefined>(undefined);
  const currState = useRef<T | undefined>(newState);

  if (currState.current !== newState) {
    prevState.current = currState.current;
    currState.current = newState;
  }

  return prevState.current as T | undefined;
}

```

```ts
// Approach 2

import { useState, useEffect } from "react";

// cause extra render because of useState
export default function usePrevious<T>(newState: T): T | undefined {
  const [prevState, setPrevState] = useState<T | undefined>(undefined);
  const [currState, setCurrState] = useState<T | undefined>(newState);

  useEffect(() => {
    if (currState !== newState) {
      setPrevState(currState);
      setCurrState(newState);
    }
  }, [newState]);

  return prevState as T | undefined;
}
```
