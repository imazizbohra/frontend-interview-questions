# Find Last Index

Implement a function `findLastIndex(array, predicate, [fromIndex=array.length-1])` that takes an array of values, a function predicate, and an optional fromIndex number argument. `findLastIndex` iterates over elements of the array from right to left.

The function returns the index of the first element that satisfies the predicate when iterating backwards from fromIndex.

## Argument(s)

- array (Array): The array to inspect.
- predicate (Function): The function invoked per iteration.
- [fromIndex=array.length-1] (number): The index to start searching from (toward the left).

Predicate signature: The predicate function is invoked with three arguments: (value, index, array).

- value: The current element being iterated.
- index: The index of the current element.
- array: The original input array.

## Returns

(number): Returns the index of the found element, or -1 otherwise.

## Example(s)

```ts
const arr = [5, 4, 3, 2, 1]; // Indices: 0, 1, 2, 3, 4

// Search for the element > 3, starting from the end (index 4).
findLastIndex(arr, (num) => num > 3); // => 1

// Start searching backwards from index 3.
// Check index 3: 2 > 1 is true. Stop.
findLastIndex(arr, (num) => num > 1, 3); // => 3

// Start searching backwards from index 3.
// Check index 3: 2 < 1 is false.
// Check index 2: 3 < 1 is false.
// Check index 1: 4 < 1 is false.
// Check index 0: 5 < 1 is false. Reached start and no element found.
findLastIndex(arr, (num) => num < 1, 3); // => -1
```

## Edge cases

Your function should handle negative and out-of-bounds fromIndex values.

- Empty arrays return -1 without calling the predicate.
- Fractional fromIndex values are truncated toward zero before resolving the starting index. NaN is treated as 0.
- Negative fromIndex: negative integers count back from the last item in the array. -1 means the last element in the array, -2 means the second-to-last element, and so on.
- Negative out-of-bounds: if the resolved index from a negative fromIndex is less than 0 (e.g., fromIndex is -10 for an array of length 5), the search starts from index 0.
- Positive out-of-bounds: if fromIndex >= array.length, the search starts from the last index (array.length - 1).

```ts
const arr = [5, 4, 3, 2, 1]; // Indices: 0, 1, 2, 3, 4

// fromIndex = -3 resolves to index 2.
// Check index 2: 3 > 2 is true. Stop.
findLastIndex(arr, (num) => num > 2, -3); // => 2

// fromIndex = -3 resolves to index 2.
// Check index 2: 3 % 2 === 0 is false.
// Check index 1: 4 % 2 === 0 is true. Stop.
findLastIndex(arr, (num) => num % 2 === 0, -3); // => 1

// Start from the last index if fromIndex >= array.length.
findLastIndex(arr, (num) => num > 0, 10); // => 4

// Negative out-of-bounds: start searching from the first item in the array.
// Check index 0: 5 > 2 is true.
findLastIndex(arr, (num) => num > 0, -10); // => 0
```

## Reference(s)

Lodash [_.findLastIndex](https://lodash.com/docs/4.18.1#findLastIndex)

## Solution(s)

```ts
export default function findLastIndex<T>(
  array: Array<T>,
  predicate: (value: T, index: number, array: Array<T>) => boolean,
  fromIndex = array.length - 1,
): number {
  const length: number = array.length;

  if (length === 0) return -1;

  // normalizing index
  // NaN || 0 => 0
  // -0 || 0 => 0
  let nFromIndex: number = Math.trunc(fromIndex) || 0;
  
  nFromIndex =
    nFromIndex < 0
      ? Math.max(nFromIndex + length, 0)
      : Math.min(nFromIndex, length - 1);

  for (let i = nFromIndex; i >= 0; i--) {
    if (predicate(array[i], i, array)) {
      return i;
    }
  }

  return -1;
}
```
