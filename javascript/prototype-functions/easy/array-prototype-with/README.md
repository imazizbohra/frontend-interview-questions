# Array.prototype.with

`Array.prototype.with` returns a new array with the element at the given index replaced, without mutating the original array.

Implement `Array.prototype.with`. To avoid overwriting the actual `Array.prototype.with` which is being used by the autograder, implement it as Array.prototype.myWith.

Negative indices count from the end of the array. If the index is out of range, `myWith` should throw a `RangeError`.

## Example(s)

```ts
[1, 2, 3].myWith(1, 9); // [1, 9, 3]
[1, 2, 3].myWith(-1, 9); // [1, 2, 9]

const values = [1, 2, 3];
const result = values.myWith(0, 7);

result; // [7, 2, 3]
values; // [1, 2, 3]
```

## Notes

- Assume the input is a normal dense array and index is already an integer.
- Array-like objects, sparse-array edge cases, and specification-heavy coercion behavior are out of scope.
- Throw a RangeError when index is smaller than -array.length or greater than or equal to array.length.

## Reference(s)

- Array [with](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/with)

## Solution(s)

```ts
Array.prototype.myWith = function <T>(this: T[], index: number, value: T): T[] {
  const array = this;
  const length = array.length;

  // thrown error if index is out-of-bound
  if (index < -length || index >= length) {
    throw new RangeError(`Invalid index : ${index}`);
  }

  // normalized index
  const nIndex =
    index < 0 ? Math.max(index + length, 0) : Math.min(index, length);

  const returnValue = Array.from(array);
  returnValue[nIndex] = value;
  return returnValue;
};
```
