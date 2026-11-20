# Intersection By

The `intersectionBy` function takes an iteratee function and multiple arrays as arguments. It first applies the iteratee function to transform the values in all arrays. Then, it identifies the set of transformed values that are common across all arrays. Finally, it returns an array containing the original values from the first array that correspond to these common transformed values.

Note: The comparison to find common elements uses the values after the iteratee is applied, but the final returned array contains the original values from the first array (before the iteratee was applied).

The iteratee function is invoked with one argument: value, where value is the current value being iterated.

## Argument(s)

- iteratee (Function): The iteratee invoked per element.
- arrays (Array): The arrays to inspect.

## Returns

(Array): Returns the new array of intersecting values.

## Example(s)

```ts
// Get the intersection based on the floor value of each number
const result = intersectionBy(Math.floor, [1.2, 2.4], [2.5, 3.6]);
// Compares floored values ([1, 2] vs [2, 3]). The common floor value is 2.
// Original value from the first array that floors to 2 is 2.4.
// => [2.4]

// Get the intersection based on the lowercase value of each string
const result2 = intersectionBy(
  (str) => str.toLowerCase(),
  ['apple', 'banana', 'ORANGE', 'orange'],
  ['Apple', 'Banana', 'Orange'],
);
// Common lowercase results are 'apple', 'banana', 'orange'.
// Returns the corresponding first-occurrence originals from the first array: 'apple', 'banana', 'ORANGE'.
// => ['apple', 'banana', 'ORANGE']

// Single array case
intersectionBy(Math.floor, [1, 2.5, 3]); // => [1, 2.5, 3]
```

## Constraints

- The input arrays may contain any type of values.
- The input arrays may have varying lengths.
- The input arrays may be empty.
- The function should not modify the original arrays.
- 0 <= number of arrays <= 20.

## Reference(s)

- Lodash [_.intersectionBy](https://lodash.com/docs/4.18.1#intersectionBy)

## Solution(s)

```ts
// Approach 1

export default function intersectionBy<T, R>(
  iteratee: (value: T) => R,
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

  // transform the values in all arrays
  const transformedArrays = arrays.reduce((acc: R[][], array: T[]) => {
    const transformedArray: R[] = array.map((elem) => iteratee(elem));
    acc.push(transformedArray);
    return acc;
  }, []);

  const set = new Set<T>();
  const visited = new Set<R>();

  const firstArray = transformedArrays[0];
  for (let i = 0; i < firstArray.length; i++) {
    const elem: R = firstArray[i];

    const isExist = transformedArrays.every((transformedArray) => {
      return transformedArray.includes(elem);
    });

    if (isExist && !visited.has(elem)) {
      visited.add(elem);
      set.add(arrays[0][i]);
    }
  }

  return Array.from(set);
}
```

```ts
// Approach 2

export default function intersectionBy<T, R>(
  iteratee: (value: T) => R,
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

  // transform the values in all arrays
  const transformedArrays = arrays.reduce((acc: Set<R>[], array: T[]) => {
    const transformedArray: R[] = array.map((elem) => iteratee(elem));
    acc.push(new Set(transformedArray));
    return acc;
  }, []);

  const set = new Set<T>();
  const visited = new Set<R>();

  const firstArray = arrays[0];
  for (let i = 0; i < firstArray.length; i++) {
    const elem: R = iteratee(firstArray[i]);

    const isExist = transformedArrays.every((transformedSet) => {
      return transformedSet.has(elem);
    });

    if (isExist && !visited.has(elem)) {
      visited.add(elem);
      set.add(firstArray[i]);
    }
  }

  return Array.from(set);
}
```
