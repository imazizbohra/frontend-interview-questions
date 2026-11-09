# Throttle

Throttling is a technique used to control how many times a function is allowed to execute over time. When a JavaScript function is said to be throttled with a `wait` time of X milliseconds, it can only be invoked at most once every X milliseconds. The callback is invoked immediately and cannot be invoked again for the rest of the `wait` duration.

Implement `throttle(func, wait)` so that func can run at most once every `wait` milliseconds. The first call should invoke func immediately. Calls made during the throttling window should be ignored. When `func` runs, it should receive the call's arguments and preserve `this`.

## Arguments

- func (Function): The callback to throttle.
- wait (number): The number of milliseconds in each throttling window.

## Returns

(Function): Returns the throttled function.

## Example(s)

```ts
let i = 0;
function increment() {
  i++;
}
const throttledIncrement = throttle(increment, 100);

// t = 0: Call throttledIncrement(). i is now 1.
throttledIncrement(); // i = 1

// t = 50: Call throttledIncrement() again.
//  i is still 1 because 100ms have not passed.
throttledIncrement(); // i = 1

// t = 101: Call throttledIncrement() again. i is now 2.
//  i is incremented because it has been more than 100ms
//  since the last throttledIncrement() call at t = 0.
throttledIncrement(); // i = 2
```

## Reference(s)

- Lodash [throttle](https://lodash.com/docs/4.18.1#throttle)

## Techniques

- Using `setTimeout`
- Closures
- How `this` works
- Deferring a call without losing its receiver or argument list

## Solution(s)

```ts
type ThrottleFunction<T extends any[]> = (...args: T) => any;

export default function throttle<T extends any[]>(
  func: ThrottleFunction<T>,
  wait: number,
): ThrottleFunction<T> {
  let isOpen: boolean = true;

  return function (this: any, ...args: T) {
    if (isOpen) {
      isOpen = false;
      func.call(this, ...args);

      setTimeout(() => {
        isOpen = true;
      }, wait);
    }
  };
}
```
