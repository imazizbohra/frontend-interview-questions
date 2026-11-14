# Console Log History

Sometimes it is useful to inspect what has been logged to the console earlier in a page's lifetime.

Implement `consoleLogHistory()`, a function that installs console log tracking and returns a function for reading the recorded history.

Assume `consoleLogHistory()` is called once before any `console.log()` calls that you want to track. When called, it may replace the default `console` with a custom implementation.

Only `console.log()` and `console.clear()` matter for this question:

- `console.log(...args)` should record one history entry containing the exact argument list for that call.
- `console.clear()` should clear the stored history.
- Wrapped `console.log()` and `console.clear()` should still call through to the original console methods.

## Example(s)

```ts
import consoleLogHistory from './console-log-history';

const getHistory = consoleLogHistory();

console.log('hello');
console.log('count', 2);

getHistory();
// [['hello'], ['count', 2]]
```

Calling console.clear() should reset the history.

```ts
const getHistory = consoleLogHistory();

console.log('first');
console.clear();
console.log('second');

getHistory();
// [['second']]
```

## Reference(s)

- Javascript [function-borrowing](https://javascript.plainenglish.io/function-borrowing-in-javascript-275e336a27f6)
- Function [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

## Solution(s)

```ts
// Approach 1

export default function consoleLogHistory(): () => Array<Array<unknown>> {
  let history: Array<Array<unknown>> = [];
  const originalLog = console.log;
  const originalClear = console.clear;

  console.log = (...args) => {
    history.push(args);
    originalLog.call(console, ...args);
  };

  console.clear = () => {
    history = [];
    originalClear.call(console);
  };

  return () => history.slice();
}
```

```ts
// Approach 2

type ConsoleMethod = (...args: Array<unknown>) => void;

export default function consoleLogHistory(): () => Array<Array<unknown>> {
  let history: Array<Array<unknown>> = [];

  const originalConsole = globalThis.console;
  // Native console methods expect the original console object as `this`.
  const originalLog = originalConsole.log.bind(
    originalConsole,
  ) as ConsoleMethod;
  const originalClear = originalConsole.clear.bind(
    originalConsole,
  ) as ConsoleMethod;

  // Inherit every other console method without reimplementing them.
  const wrappedConsole = Object.create(originalConsole) as Console & {
    log: ConsoleMethod;
    clear: ConsoleMethod;
  };

  wrappedConsole.log = (...args: Array<unknown>): void => {
    // Store each call as the exact argument list that was logged.
    history.push(args);
    originalLog(...args);
  };

  wrappedConsole.clear = (...args: Array<unknown>): void => {
    // Clearing the console also resets the captured log history.
    history = [];
    originalClear(...args);
  };

  globalThis.console = wrappedConsole;

  return function getHistory(): Array<Array<unknown>> {
    // Return a copy so callers cannot mutate the internal history array.
    return history.slice();
  };
}
```
