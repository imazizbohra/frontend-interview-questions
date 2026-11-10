# useCountdown

Implement a `useCountdown` hook that manages a countdown.

## Argument(s)

The hook accepts an object with the following properties:

- countStart: number: The initial value of the countdown
- countStop: number: The value at which the countdown should stop. This defaults to 0
- intervalMs: number: The interval (in milliseconds) at which the countdown should decrease. This defaults to 1000
- isIncrement: boolean: A flag indicating whether the countdown should increment instead of decrement. Defaults to false

## Returns

The hook returns an object with the following properties:

- count: number: The current value of the countdown
- start: () => void: A function that starts the countdown
- stop: () => void: A function that stops the countdown
- reset: () => void: A function that resets the countdown to countStart

## Notes

- start() should begin ticking only when the countdown is not already running.
- stop() should pause the countdown without resetting the current count.
- reset() should stop the countdown and return count to countStart.
- The interval should clean itself up when the hook unmounts or when the countdown stops.

## Example(s)

```tsx
export default function Component() {
  const { count, start, stop, reset } = useCountdown({ countStart: 10 });

  return (
    <div>
      <p>Countdown: {count}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useCallback, useEffect, useState } from "react";

interface UseCountdownOptions {
  countStart: number;
  countStop?: number;
  intervalMs?: number;
  isIncrement?: boolean;
}

interface UseCountdownReturn {
  count: number;
  start: () => void;
  stop: () => void;
  reset: () => void;
}

export default function useCountdown({
  countStart,
  countStop = 0,
  intervalMs = 1000,
  isIncrement = false,
}: UseCountdownOptions): UseCountdownReturn {
  const [count, setCount] = useState(countStart);
  const [running, setRunning] = useState(false);

  const reset: UseCountdownReturn["reset"] = useCallback(() => {
    setRunning(false);
    setCount(countStart);
  }, [countStart]);

  const start: UseCountdownReturn["start"] = useCallback(() => {
    setRunning(true);
  }, []);

  const stop: UseCountdownReturn["stop"] = useCallback(() => {
    setRunning(false);
  }, []);

  useEffect(() => {
    if (!running) return;
    
    const id = setInterval(() => {
      if (count === countStop) return stop();

      if (isIncrement) {
        setCount((prev) => prev + 1);
      } else {
        setCount((prev) => prev - 1);
      }
    }, intervalMs);

    return () => clearInterval(id);
  }, [count, countStop, intervalMs, isIncrement, running]);

  return { count, start, stop, reset };
}
```
