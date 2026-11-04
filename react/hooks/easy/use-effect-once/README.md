# useEffectOnce

Implement a `useEffectOnce` hook that runs an effect once for the mounted component lifecycle, using the same setup and cleanup behavior as `useEffect`.

React Strict Mode can intentionally run an extra setup and cleanup cycle in development. Do not suppress that cycle; preserving React's cleanup semantics is more important than hiding development-only diagnostics.

## Argument(s)

effect: The function that will be executed once. This function has the same parameters and behavior as the first argument of useEffect

## Returns

Nothing, just like useEffect.

## Example(s)

```ts
export default function Component() {
  useEffectOnce(() => {
    console.log('Running effect once on mount');

    return () => {
      console.log('Running cleanup of effect on unmount');
    };
  });

  return null;
}
```

## Reference(s)

- React [useEffect](https://react.dev/reference/react/useEffect)
- React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useEffect, EffectCallback } from "react";

export default function useEffectOnce(effect: EffectCallback) {
  useEffect(() => {
    const cleanupFn = effect();

    return cleanupFn;
  }, []);
}

```
