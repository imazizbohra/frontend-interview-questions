# Cancellable Interval

Implement a function setCancellableInterval that acts like setInterval but, instead of returning a timer ID, returns a function that cancels the interval when called. The setCancellableInterval function should have the same signature as setInterval:

## Example(s)

```ts
let i = 0;
// t = 0:
const cancel = setCancellableInterval(() => {
  i++;
}, 10);
// t = 10: i is 1
// t = 20: i is 2
cancel(); // Called at t = 25
// t = 30: i is still 2 because cancel() was called and the interval callback has stopped running.
```

## Reference(s)

Window [setInterval](https://developer.mozilla.org/en-US/docs/Web/API/Window/setInterval)

## Solution(s)

```ts
export default function setCancellableInterval(
  callback: Function,
  delay?: number,
  ...args: Array<any>
): () => void {
  let intervalId: ReturnType<typeof setInterval> | undefined = undefined;

  intervalId = setInterval(() => {
    callback.apply(null, args);
  }, delay ?? 0);

  return () => {
    clearInterval(intervalId);
    intervalId = undefined;
  };
}
```
