# Drop While

Given a predicate function and an array, implement a function `dropWhile(array, predicate)`. This function should create a slice of array excluding elements dropped from the start of the list. Elements are dropped until predicate returns falsy. Your function should not modify the original array. The array may or may not be in sorted order.

## Argument(s)

- array (Array): The array to query.
- predicate (Function): The function invoked per iteration.

Predicate signature: The predicate function is invoked with three arguments: (value, index, array). You must invoke it with all three arguments.

- value: The current element being iterated.
- index: The index of the current element.
- array: The original input array.

## Returns

(Array): Returns the slice of array containing the kept elements.

## Example(s)

```ts
dropWhile([1, 2, 3, 4, 5], (value, _index, _array) => value < 3);
// => [3, 4, 5]
// Explanation: Starts from left (1). 1 < 3 (true, drop). 2 < 3 (true, drop). 3 < 3 (false, stop). Returns [3, 4, 5].

dropWhile([1, 2, 3], (value, _index, _array) => value < 6);
// => []
// Explanation: Starts from left (1). 1 < 6 (true, drop). 2 < 6 (true, drop). 3 < 6 (true, drop). Reaches end. Returns [].

dropWhile([1, 2, 3, 4, 5], (value, _index, _array) => value > 6);
// => [1, 2, 3, 4, 5]
// Explanation: Starts from left (1). 1 > 6 (false, stop immediately). Returns [1, 2, 3, 4, 5].

dropWhile([1, 2, 3, 4, 5], (_value, index, _array) => index < 3);
// => [4, 5]
// Explanation: Starts at index 0. 0 < 3 (true, drop). Index 1. 1 < 3 (true, drop). Index 2. 2 < 3 (true, drop). Index 3. 3 < 3 (false, stop). Returns [4, 5].

dropWhile([4, 5, 12, 10, 11], (value, _index, array) => value < array[2]);
// => [12, 10, 11]
// Explanation: array[2] is 12. Starts from left (4). 4 < 12 (true, drop). Index 1 (5). 5 < 12 (true, drop). Index 2 (12). 12 < 12 (false, stop). Returns [12, 10, 11].
```

## Note

- If the predicate fails on the first element, return a shallow copy of the whole array.
- If the predicate stays truthy for every element, return an empty array.
- An empty input array should also return an empty array.
- The input array does not need to be sorted because the scan is strictly left to right until the first falsy result.
- Sparse arrays work through normal JavaScript indexing: a hole reads as undefined when visited, and slice() preserves holes in the returned suffix.

## Reference(s)

- Array [slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
- Lodash [_.dropWhile](https://lodash.com/docs/4.18.1#dropWhile)

## Solution(s)

```ts
export default function dropWhile<T>(
  array: Array<T>,
  predicate: (value: T, index: number, array: Array<T>) => boolean,
): Array<T> {
  if (array.length === 0) return [];

  const length: number = array.length;
  let startIdx: number = 0;
  while (startIdx < length) {
    const value: T = array[startIdx];

    if (predicate(value, startIdx, array)) {
      startIdx++;
    } else {
      break;
    }
  }

  return array.slice(startIdx, length);
}
```
