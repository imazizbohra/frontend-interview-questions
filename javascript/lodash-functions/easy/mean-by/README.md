# Mean By

Implement a function `meanBy(array, iteratee)` that calculates the mean of the values produced by iteratee for the elements in array.

## Argument(s)

- array (Array): The array to iterate over.
- iteratee (Function): The iteratee invoked per element, which accepts one argument: (value).

## Returns

(Number): Returns the mean value.

## Example(s)

```ts
meanBy([{ n: 1 }, { n: 3 }], (o) => o.n); // => 2

meanBy([2, 3, 1, 4], (num) => num); // => 2.5
```

The function should ignore elements where iteratee produces null or undefined and return NaN when array is empty or has no matching values.

```ts
meanBy([], (o) => o); // => NaN

meanBy([{ n: 1 }, { n: 2 }], (o) => o.m); // => NaN

meanBy([{ n: 1 }, { m: 2 }], (o) => o.m); // => 2
```

## Reference(s)

- Lodash [_.meanBy](https://lodash.com/docs/4.18.1#meanBy)

## Solution(s)

```ts
export default function meanBy<T>(
  array: Array<T>,
  iteratee: (value: T) => number | null | undefined,
): number {
  const length = array.length;
  let sum: number = 0;
  let count: number = 0;

  for (let i = 0; i < length; i++) {
    // skip empty slots
    if (i in array) {
      const value = iteratee(array[i]);

      // zero is valid value; only null and undefined should be skipped
      if (value != null) {
        sum = sum + value;
        count++;
      }
    }
  }

  return sum / count;
}
```
