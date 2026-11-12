# Debounce 2

**Note:** This is an advanced version of [Debounce](../debounce/); you should complete that first before attempting this question.

Debouncing controls how often a function can execute over time. When a JavaScript function is debounced with a wait time of `wait` milliseconds, it runs only after `wait` milliseconds have elapsed since the debounced function was last called.

You have probably encountered debouncing in daily life before, such as when entering an elevator. Only after some time passes without pressing the "Door open" button (the debounced function not being called) will the elevator door actually close (the callback function is executed).

Implement a `debounce` function that accepts a callback function and a wait duration. Calling `debounce()` returns a function that debounces invocations of the callback function following the behavior described above.

The returned function accepts the same arguments as the callback. When the callback eventually runs, it receives the `this` value and arguments from the most recent invocation of the debounced function.

Additionally, the debounced function comes with two extra methods:

- `cancel()` method to cancel pending invocations.
- `flush()` method to immediately invoke any delayed invocations.

## Example(s)

```ts
let i = 0;
function increment() {
  i++;
}
const debouncedIncrement = debounce(increment, 100);

// t = 0: Call debouncedIncrement().
debouncedIncrement(); // i = 0

// t = 50: Cancel the delayed increment.
debouncedIncrement.cancel();

// t = 100: increment() was not invoked and i is still 0.
```

Flush to immediately call the debounced function.

```ts
let i = 0;
function increment() {
  i++;
}
const debouncedIncrement = debounce(increment, 100);

// t = 0: Call debouncedIncrement().
debouncedIncrement(); // i = 0

// t = 50: i is still 0 because 100ms have not passed.
// t = 51:
debouncedIncrement.flush(); // i is now 1 because flush() causes the callback to be invoked immediately.

// t = 100: i is already 1. The callback has been called before
// and won't be called again.
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
interface DebouncedFunction extends Function {
  cancel: () => void;
  flush: () => void;
}

export default function debounce(
  func: Function,
  wait?: number,
): DebouncedFunction {
  let timeoutId: ReturnType<typeof setTimeout> | undefined = undefined;
  let context: any;
  let args: any[];

  const clearTimer = () => {
    clearTimeout(timeoutId);
    timeoutId = undefined;
  };

  function invoke() {
    if (timeoutId == null) {
      return;
    }

    clearTimer();
    func.call(context, ...args);
  }

  function debounced(this: any, ...rest: any[]) {
    // keep the latest context and args
    context = this;
    args = rest;

    clearTimer();

    timeoutId = setTimeout(() => {
      invoke();
    }, wait);
  }

  debounced.cancel = clearTimer;
  debounced.flush = invoke;

  return debounced;
}
```
