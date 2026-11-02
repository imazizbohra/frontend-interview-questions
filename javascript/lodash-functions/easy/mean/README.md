# Mean

Implement a function `mean(array)` that returns the mean (also known as the average) of the values in array, an array of numbers.

## Argument(s)

array (Array): The array of numbers.

## Returns

(Number): Returns the mean of the values in array.

## Example(s)

```ts
mean([4, 2, 8, 6]); // => 5
mean([1, 2, 3, 4]); // => 2.5
mean([1, 2, 2]); // => 1.6666666666666667
```

```ts
mean([]); // => NaN
```

## Reference(s)

- Array [Reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- Lodash [Mean](https://lodash.com/docs/4.18.1#mean)

## Solution(s)

```ts
// Approach 1

export default function mean(array: number[]): number {
  if (array.length === 0) {
    return NaN;
  }

  const total: number = values.reduce((acc, value) => acc + value, 0);
  const mean: number = total / values.length;

  return mean;
}
```

```ts
// Approach 2

export default function mean(array: number[]): number {
  // because 0 / 0 = NaN
  return array.reduce((acc, value) => acc + value, 0) / array.length;
}
```
