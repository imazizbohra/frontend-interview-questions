# Intersection With

The `intersectionWith` function takes a custom `comparator` function and multiple arrays as arguments. It compares the elements of the arrays using the `comparator` function to determine equality. The function returns a new array containing the elements that are present in all given arrays.

Implement `intersectionWith(comparator, ...arrays)` so it returns the matching values from the first array in the same order they appear there. If no arrays are provided, or any array is empty, return [].

## Argument(s)

- comparator (Function): The function used to compare two elements. It is invoked with (arrVal, othVal) and should return true when the values should be treated as equal.
- arrays (...Array): The arrays to inspect.

## Returns

(Array): Returns the intersecting values from the first array.

## Example(s)

```ts
const arr1 = [
  { x: 1, y: 2 },
  { x: 2, y: 3 },
];
const arr2 = [
  { y: 2, x: 1 },
  { x: 3, y: 4 },
];

const result = intersectionWith(
  (a, b) => a.x === b.x && a.y === b.y,
  arr1,
  arr2,
); // => [{ x: 1, y: 2 }]

intersectionWith((a, b) => a === b, [1, 2, 3], [2, 3, 4], [3, 4, 5]);
// => [3]
```

## Notes

- In Lodash, comparator is optional and is the last parameter, but in this question it is a required parameter for simplicity.
- The order of elements in the resulting array is determined by the order in which they appear in the first array.
- If no arrays are provided, the function will return an empty array.
- If any of the arrays are empty, the function will return an empty array.

## Reference(s)

- Lodash [_.intersintersectionWithectionBy](https://lodash.com/docs/4.18.1#intersectionWith)

## Solution(s)

```ts
export default function intersectionWith<T>(
  comparator: (a: T, b: T) => boolean,
  ...arrays: Array<Array<T>>
): Array<T> {
  if (arrays.length === 0) {
    return [];
  }

  if (arrays.length === 1) {
    return Array.from(new Set(arrays[0]));
  }

  if (arrays.some((arr) => arr.length === 0)) {
    return [];
  }

  const set = new Set<T>();

  const firstArray = arrays[0];
  for (let i = 0; i < firstArray.length; i++) {
    const elemA: T = firstArray[i];

    const isExist = arrays.every((array) => {
      return array.some((elemB) => comparator(elemA, elemB));
    });

    if (isExist) {
      set.add(elemA);
    }
  }

  return Array.from(set);
}
```
