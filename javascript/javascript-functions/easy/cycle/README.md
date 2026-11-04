# Cycle

Implement a function that takes one or more values and returns a function that cycles through those values each time it is called.

## Example(s)

```ts
const helloFn = cycle('hello');
console.log(helloFn()); // "hello"
console.log(helloFn()); // "hello"

const onOffFn = cycle('on', 'off');
console.log(onOffFn()); // "on"
console.log(onOffFn()); // "off"
console.log(onOffFn()); // "on"
```

## Notes

- The question says cycle() takes one or more values. The function should not be called without any arguments.
- With one value, the same value is returned on every call.
- Each call to cycle() creates a separate closure with its own index.
- The closure stores the argument values in the rest-parameter array. It does not deep-clone object values passed into the cycle.

## Solution(s)

```ts
export default function cycle<T>(...values: Array<T>): () => T {
  let index = 0;
  let length = values.length;

  return function () {
    const value = values[index];
    index = (index + 1) % length;
    return value;
  };
}
```
