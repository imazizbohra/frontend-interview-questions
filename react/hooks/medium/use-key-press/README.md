# useKeyPress

Implement a `useKeyPress` hook that detects keyboard events and runs a callback.

## Argument(s)

- `key: string`: The key to detect. It must be a valid key value that is usually obtained from KeyboardEvent.key
- `callback: (event: KeyboardEvent) => void`: A function that is called when the key identified by key is pressed
- `options`: An optional object with the following properties:
  - `target: EventTarget`: The target element to listen for the key press. This defaults to window
  - `event: 'keydown' | 'keyup'`: The event to listen for. This defaults to 'keydown'

## Returns

Nothing.

## Notes

- Listen to keydown by default, not the deprecated keypress event.
- Match against `KeyboardEvent.key`.
- Pass the original keyboard event to callback so callers can call `preventDefault()` or inspect modifiers.
- Clean up the event listener when the component unmounts or when the key, callback, event type, or target changes.

## Example(s)

```tsx
export default function Component() {
  useKeyPress('a', (e) => {
    e.preventDefault();
    console.log('The "a" key was pressed');
  });

  return null;
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useEffect } from "react";

export default function useKeyPress(
  key: string,
  callback: (e: KeyboardEvent) => void,
  {
    event = "keydown",
    target = window,
  }: { event?: "keydown" | "keyup"; target?: EventTarget } = {
    event: "keydown",
    target: window,
  },
) {
  useEffect(() => {
    const listener = (event: KeyboardEvent) => {
      if (event.key === key) {
        callback(event);
      }
    };

    target.addEventListener(event, listener as EventListener);

    return () => target.removeEventListener(event, listener as EventListener);
  }, [key, callback, event, target]);
}
```
