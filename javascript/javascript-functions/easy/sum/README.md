# Sum

Implement a sum function that accepts a number and can be called repeatedly with more numbers. Calling the function without an argument will sum up all the arguments so far and return the total.

## Example(s)

```ts
sum(1)(); // 1
sum(1)(2)(); // 3
sum(1)(2)(-3)(); // 0
```

A partially applied chain can be reused. Extending one branch should not change the result of another branch created from the same earlier call.

## Solution(s)

```ts
// Approach 1

type SumResult = (value?: number) => number | SumResult;

export default function sum(value: number): SumResult {
  let sum = value;

  return function inner(val?: number) {
    if (val == null) {
      const returnValue = sum;
      sum = value;
      return returnValue;
    }

    sum += val;

    return inner;
  };
}
```

```ts
// Approach 2

type SumResult = (value?: number) => number | SumResult;

export default function sum(value: number): SumResult {
  return function inner(val?: number) {
    if (val == null) {
      return value;
    }

    return sum(value + val);
  };
}
```
