# Drop Right While

Given a predicate function and an array, implement a function dropRightWhile(array, predicate). This function should create a slice of array excluding elements dropped from the end. Elements are dropped until predicate returns falsy. Your function should not modify the original array. The array may or may not be in sorted order.

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
// Example 1: Basic usage
dropRightWhile([1, 2, 3, 4, 5], (value, _index, _array) => value > 3);
// => [1, 2, 3]
// Explanation: Starts from right (5). 5 > 3 (true, drop). 4 > 3 (true, drop). 3 > 3 (false, stop). Returns [1, 2, 3].

// Example 2: Predicate always true
dropRightWhile([1, 2, 3], (value, _index, _array) => value < 6);
// => []
// Explanation: 3 < 6 (true, drop). 2 < 6 (true, drop). 1 < 6 (true, drop). Returns empty array.

// Example 3: Predicate always false
dropRightWhile([1, 2, 3, 4, 5], (value, _index, _array) => value > 6);
// => [1, 2, 3, 4, 5]
// Explanation: 5 > 6 (false, stop immediately). Returns the original array slice.

// Example 4: Using the `index` argument
dropRightWhile([1, 2, 3, 4, 5], (_value, index, _array) => index > 2);
// => [1, 2, 3]
// Explanation: Starts at index 4. 4 > 2 (true, drop). Index 3. 3 > 2 (true, drop). Index 2. 2 > 2 (false, stop). Returns [1, 2, 3].

// Example 5: Using the `array` argument
dropRightWhile([10, 11, 12, 4, 5], (value, _index, array) => value < array[1]);
// => [10, 11, 12]
// Explanation: array[1] = 11. 5 < 11 (true, drop). 4 < 11 (true, drop). 12 < 11 (false, stop). Returns [10, 11, 12].
```

## Note

- If every element satisfies the predicate, the function should return an empty array.
- If the rightmost element fails immediately, the function should return a shallow copy of the whole array.
- The input array is never modified; slice() creates the returned array.
- This is a suffix operation. If matching values appear earlier in the kept prefix, they remain in the output.

## Reference(s)

- Array [slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
- Lodash [_.dropRightWhile](https://lodash.com/docs/4.18.1#dropRightWhile)

## Solution(s)

```ts
export default function dropRightWhile<T>(
  array: Array<T>,
  predicate: (value: T, index: number, array: Array<T>) => boolean,
): Array<T> {
  if (array.length === 0) return [];

  let endIdx: number = array.length - 1;
  while (endIdx >= 0) {
    const value: T = array[endIdx];

    if (predicate(value, endIdx, array)) {
      endIdx--;
    } else {
      break;
    }
  }

  return array.slice(0, endIdx + 1);
}
```
