# Debounce

Debouncing controls how often a function can execute over time. When a JavaScript function is debounced with a `wait` time of `wait` milliseconds, it runs only after `wait` milliseconds have elapsed since the debounced function was last called.

You have probably encountered debouncing in daily life before, such as when entering an elevator. Only after some time passes without pressing the "Door open" button (the debounced function not being called) will the elevator door actually close (the callback function is executed).

Implement `debounce(func, wait)` so that `func` is called only after `wait` milliseconds have passed since the most recent call. The returned function should not invoke `func` immediately. When the delayed call finally runs, it should use the latest arguments and preserve the `this` value from the most recent call.

## Arguments

- func (Function): The callback to debounce.
- wait (number): The number of milliseconds to wait after the latest call.

## Returns

(Function): Returns the debounced function.

## Example(s)

```ts
let i = 0;
function increment() {
  i++;
}
const debouncedIncrement = debounce(increment, 100);

// t = 0: Call debouncedIncrement().
debouncedIncrement(); // i = 0

// t = 50: i is still 0 because 100ms have not passed.

// t = 100: increment() was invoked and i is now 1.
```

`debouncedIncrement()` is called multiple times.

```ts
let i = 0;
function increment() {
  i++;
}
const debouncedIncrement = debounce(increment, 100);

// t = 0: Call debouncedIncrement().
debouncedIncrement(); // i = 0

// t = 50: i is still 0 because 100ms have not passed.
//  Call debouncedIncrement() again.
debouncedIncrement(); // i = 0

// t = 100: i is still 0 because it has only
//  been 50ms since the last debouncedIncrement() at t = 50.

// t = 150: Because 100ms have passed since
//  the last debouncedIncrement() at t = 50,
//  increment was invoked and i is now 1.
```

## Reference(s)

- Lodash [debounce](https://lodash.com/docs/4.18.1#debounce)

## Techniques

- Using `setTimeout`
- Closures
- How `this` works
- Deferring a call without losing its receiver or argument list

## Solution(s)

```ts
// Approach 1

export default function debounce(func: Function, wait: number): Function {
  let timeoutId: ReturnType<typeof setTimeout> | null = null;

  return function (this: any, ...args: any[]) {
    const context = this;

    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    // with regular function
    timeoutId = setTimeout(function () {
      timeoutId = null;
      func.call(context, ...args);
    }, wait);
  };
}
```

```ts
// Approach 2

export default function debounce(func: Function, wait: number): Function {
  let timeoutId: ReturnType<typeof setTimeout> | null = null;

  return function (this: any, ...args: any[]) {
    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    // with arrow function
    timeoutId = setTimeout(() => {
      timeoutId = null;
      func.call(this, ...args);
    }, wait);
  };
}
```
