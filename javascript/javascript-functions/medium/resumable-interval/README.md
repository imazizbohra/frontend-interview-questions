# Resumable Interval

**Note**: It is recommended to have completed the [Cancellable Interval](../../easy//cancellable-interval/) question before attempting this question.

Implement a function `createResumableInterval` that acts like `setInterval` and has the same signature. However, instead of returning a timer ID, it returns an object that contains three methods:

- `start`: Runs the callback `immediately` and every `delay` milliseconds.
- `pause`: Pauses the interval so that it stops running. Execution can be resumed by calling `start()` again.
- `stop`: Stops the interval permanently; it cannot be restarted.

A newly created interval starts paused. Calling `start()` while it is already running should do nothing: it must not invoke the callback again or create another timer.

## Examples

```ts
let i = 0;
// t = 0:
const interval = createResumableInterval(() => {
  i++;
}, 10);
// t = 10:
interval.start(); // i is now 1.
// t = 20: callback executes and i is now 2.
// t = 25:
interval.pause();
// t = 30: i remains at 2 because interval.pause() was called.
// t = 35:
interval.start(); // i is now 3.
// t = 45: callback executes and i is now 4.
// t = 50:
interval.stop(); // i remains at 4.
```

## Notes

- A created interval starts paused; no callback runs until `start()` is called.
- Calling `start()` while already running should not create a second interval.
- Calling `stop()` should be terminal; later `start()` calls should do nothing.
- `pause()` should stop future ticks without discarding the ability to resume with `start()`.
- Extra arguments passed to `createResumableInterval(callback, delay, ...args)` are forwarded to the immediate callback invocation and to `setInterval()`.
- this does not need special handling within the callback function because there is no option to pass a `thisArg` to `setInterval`, unlike `Array.prototype.forEach()` or `Array.prototype.reduce()`. Read more about this on MDN.

## References

Window [setInterval](https://developer.mozilla.org/en-US/docs/Web/API/Window/setInterval)

## Solutions

```ts
// Approach 1 

interface Resumable {
  start: () => void;
  pause: () => void;
  stop: () => void;
}

export default function createResumableInterval(
  callback: Function,
  delay?: number,
  ...args: Array<any>
): Resumable {
  let isRunning: boolean = false;
  let isStoped: boolean = false;
  let timerId: number | undefined = undefined;

  const clear = () => {
    clearInterval(timerId);
    timerId = undefined;
  };

  return {
    start(this: any) {
      if (isRunning || isStoped) return;

      isRunning = true;
      callback.call(this, ...args);
      timerId = setInterval(() => {
        callback.call(this, ...args);
      }, delay);
    },
    pause() {
      isRunning = false;
      clear();
    },
    stop() {
      isStoped = true;
      clear();
    },
  };
}
```

```ts
// Approach 2

interface Resumable {
  start: () => void;
  pause: () => void;
  stop: () => void;
}

export default function createResumableInterval(
  callback: Function,
  delay?: number,
  ...args: Array<any>
): Resumable {
  let timerId: number | null;
  let state: State = 'paused';

  function nextState(action: Action) {
    // Table-driven transitions keep invalid actions as explicit no-ops.
    const newState: State = stateMachine[state][action];
    if (newState === state) {
      return;
    }

    state = newState;
    switch (state) {
      case 'paused':
      case 'stopped':
        clearInterval(timerId ?? undefined);
        timerId = null;
        return;
      case 'running':
        // Entering `running` always triggers an immediate tick before scheduling repeats.
        callback(...args);
        timerId = setInterval(callback, delay, ...args);
        return;
    }
  }

  return {
    start: () => nextState('start'),
    pause: () => nextState('pause'),
    stop: () => nextState('stop'),
  };
}

type State = 'stopped' | 'paused' | 'running';
type Action = 'pause' | 'start' | 'stop';
type StateMachine = Record<State, Record<Action, State>>;

const stateMachine: StateMachine = {
  stopped: {
    pause: 'stopped',
    start: 'stopped',
    stop: 'stopped',
  },
  paused: {
    pause: 'paused',
    start: 'running',
    stop: 'stopped',
  },
  running: {
    pause: 'paused',
    start: 'running',
    stop: 'stopped',
  },
};
```
