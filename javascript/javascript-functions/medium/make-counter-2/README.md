# Make Counter 2

Implement a function `makeCounter` that accepts an optional integer value (defaults to 0) and returns an object that contains the following methods:

- `get()`: returns the current value.
- `increment()`: increments the current value and returns it.
- `decrement()`: decrements the current value and returns it.
- `reset()`: resets the current value to the initial value and returns it.

## Example(s)

```ts
const counter = makeCounter();
counter.get(); // 0
counter.increment(); // 1
counter.increment(); // 2
counter.get(); // 2
counter.reset(); // 0
counter.decrement(); // -1
```

With a custom initial value:

```ts
const counter = makeCounter(5);
counter.get(); // 5
counter.decrement(); // 4
counter.decrement(); // 3
counter.get(); // 3
counter.reset(); // 5
counter.increment(); // 6
```

Each call to makeCounter() should create an independent counter.

## Reference(s)

- JavaScript [Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- Operators [Increment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Increment)
- Operators [Decrement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Decrement)

## Solution(s)

```ts
// Approach 1

interface Counter {
  get: () => number;
  increment: () => number;
  decrement: () => number;
  reset: () => number;
}

export default function makeCounter(initialValue: number = 0): Counter {
  let value = initialValue;

  const returnValue: Counter = {
    get: () => value,
    increment: () => ++value,
    decrement: () => --value,
    reset: () => {
      value = initialValue;
      return value;
    },
  };

  return returnValue;
}
```

```ts
// Approach 2

interface CounterValue {
  get: () => number;
  increment: () => number;
  decrement: () => number;
  reset: () => number;
}

class Counter implements CounterValue {
  initialValue: number;
  value: number;

  constructor(initialValue: number = 0) {
    this.initialValue = initialValue;
    this.value = initialValue;
  }

  get(): number {
    return this.value;
  }

  increment(): number {
    return ++this.value;
  }

  decrement(): number {
    return --this.value;
  }

  reset(): number {
    this.value = this.initialValue;
    return this.value;
  }
}

export default function makeCounter(initialValue: number = 0): CounterValue {
  return new Counter(initialValue);
}
```
