# Cancellable Timeout

Implement a function setCancellableTimeout that acts like setTimeout but, instead of returning a timer ID, returns a function that cancels the pending callback function when called. The setCancellableTimeout function should have the same signature as setTimeout:

## Example(s)

```ts
let i = 0;
// t = 0:
const cancel = setCancellableTimeout(() => {
  i++;
}, 100);
// t = 50:
cancel();
// t = 100: i is still 0 because cancel() was called.
```

## Reference(s)

Window [setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout)

## Solution(s)

```ts
export default function setCancellableTimeout(
  callback: Function,
  delay?: number,
  ...args: Array<any>
): () => void {
  let intervalId: ReturnType<typeof setTimeout> | undefined = undefined;

  intervalId = setTimeout(() => {
    callback.apply(null, args);
  }, delay ?? 0);

  return () => {
    clearTimeout(intervalId);
    intervalId = undefined;
  };
}
```
