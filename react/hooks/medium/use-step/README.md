# useStep

Implement a `useStep` hook that manages a step counter for a multi-step process.

The hook should provide utility methods to go to the next and previous steps, and allow the step counter to be reset. The step number is 1-indexed and goes up to at most `maxStep`.

## Argument(s)

`maxStep: number`: The maximum step number, i.e., the last step

## Returns

The hook returns an object with the following properties and methods:

- `step`: The current step number. It is initialized to 1
- `next: () => void`: A function that increments the step number
- `previous: () => void`: A function that decrements the step number
- `hasNext: boolean`: A boolean that indicates whether there is a next step
- `hasPrevious: boolean`: A boolean that indicates whether there is a previous step
- `reset: () => void`: A function that resets the step number to 1
- `setStep`: A function that sets the step number to a specific step. This should accept both a number and an updater function `(currentStep: number) => number`, similar to what the setter function of `useState` accepts

## Notes

- Valid step values are integers from `1` to `maxStep`, inclusive.
- `next()` and `previous()` should clamp at the boundaries.
- `setStep()` should ignore values outside the valid range.
- `hasNext` and `hasPrevious` should be derived from the current step instead of stored separately.

## Example(s)

```tsx
export default function Component() {
  const { step, next, previous, reset, setStep, hasNext, hasPrevious } =
    useStep(3);

  return (
    <div>
      <p>Step {step}</p>
      <button onClick={previous} disabled={!hasPrevious}>
        Previous
      </button>
      <button onClick={next} disabled={!hasNext}>
        Next
      </button>
      <button onClick={reset}>Reset</button>
      <button onClick={() => setStep(3)}>Go to step 3</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import {
  Dispatch,
  SetStateAction,
  useState,
  useCallback,
  useMemo,
} from "react";

interface UseStepReturn {
  step: number;
  next: () => void;
  previous: () => void;
  reset: () => void;
  hasNext: boolean;
  hasPrevious: boolean;
  setStep: Dispatch<SetStateAction<number>>;
}

export default function useStep(maxStep: number): UseStepReturn {
  const [step, setStep] = useState<number>(1);

  const next: () => void = useCallback(() => {
    setStep((prevState) => Math.min(maxStep, prevState + 1));
  }, []);

  const previous: () => void = useCallback(() => {
    setStep((prevState) => Math.max(1, prevState - 1));
  }, []);

  const reset: () => void = useCallback(() => {
    setStep(1);
  }, []);

  const hasNext: boolean = useMemo(() => {
    return step + 1 <= maxStep;
  }, [step]);

  const hasPrevious: boolean = useMemo(() => {
    return step - 1 >= 1;
  }, [step]);

  const set: Dispatch<SetStateAction<number>> = useCallback(
    (valueOrFunction) => {
      setStep((prevState) => {
        const newState =
          typeof valueOrFunction === "function"
            ? valueOrFunction(prevState)
            : valueOrFunction;

        return newState >= 1 && newState <= maxStep ? newState : prevState;
      });
    },
    [],
  );

  return { step, next, previous, reset, hasNext, hasPrevious, setStep: set };
}
```
