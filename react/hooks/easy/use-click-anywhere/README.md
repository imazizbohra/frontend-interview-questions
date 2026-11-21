# useClickAnywhere

Implement a `useClickAnywhere` hook that handles click events anywhere in the browser window.

## Arguments

handler: (event: MouseEvent) => void: The function to be called when a click event is detected anywhere in the window

## Returns

Nothing.

## Examples

```ts
export default function Component() {
  const [count, setCount] = useState(0);

  useClickAnywhere(() => {
    setCount((prev) => prev + 1);
  });

  return <p>Click count: {count}</p>;
}
```

## References

- EventTarget [addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- React [useEffect](https://react.dev/reference/react/useEffect)
- React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solutions

```ts
import { useEffect } from "react";

export default function useClickAnywhere(handler: (event: MouseEvent) => void) {
  useEffect(() => {
    const handleClick = (event: MouseEvent) => {
      handler(event);
    };

    window.addEventListener("click", handleClick);

    return () => window.removeEventListener("click", handleClick);
  }, [handler]);
}
```
