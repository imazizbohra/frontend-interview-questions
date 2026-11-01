# Make Counter

Implement a function makeCounter that accepts an optional integer value and returns a function. When the returned function is called initially, it returns the initial value if provided, otherwise 0. The returned function can be called repeatedly to return one more than the return value of the previous invocation.

## Argument(s)

- initialValue (number): Value of counter to start with.

## Returns

(number): Returns the value of counter at the moment.

## Example(s)

```ts
const counter = makeCounter();
counter(); // 0
counter(); // 1
counter(); // 2
```

```ts
const counter = makeCounter(5);
counter(); // 5
counter(); // 6
counter(); // 7
```

## Reference(s)

- JavaScript [Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- Operators [Increment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Increment)

## Solution(s)

```ts
export default function makeCounter(initialValue: number = 0): () => number {
  let value: number = initialValue;

  const counterFn = function () {
    return value++;
  }

  return counterFn;
}
```
