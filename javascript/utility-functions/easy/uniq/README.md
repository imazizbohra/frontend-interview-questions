# Unique Array

Implement a function `uniqueArray` that takes in an array and returns a duplicate-free version of the array where only the first occurrence of each element is kept. The order of result values is determined by the order in which they occur in the array.

Uniqueness uses SameValueZero equality: repeated NaN values are treated as equal, while objects are equal only when they are the same reference.

## Argument(s)

array (Array): The array to inspect.

## Returns

(Array): Returns the new duplicate free array.

## Example(s)

```ts
uniqueArray([1, 2, 3]); // [1, 2, 3]
uniqueArray([1, 1, 2]); // [1, 2]
uniqueArray([2, 1, 2]); // [2, 1]
```

## Reference(s)

- Lodash [_.uniq](https://lodash.com/docs/4.18.1#uniq)

## Solution(s)

```ts
// Approach 1

export default function uniqueArray<T extends ReadonlyArray<unknown>>(
  array: T,
): Array<T[number]> {
  return Array.from(new Set(array));
}
```

```ts
// Approach 2

export default function uniqueArray<T extends ReadonlyArray<unknown>>(
  array: T,
): Array<T[number]> {
  const result: Array<T[number]> = [];

  array.forEach((item) => {
    if (!result.includes(item)) {
      result.push(item);
    }
  });

  return result;
}
```
