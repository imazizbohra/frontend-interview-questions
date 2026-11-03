# Array.prototype.filter

`Array.prototype.filter` creates a new array containing the elements that pass the provided predicate function.

For sparse arrays (e.g. [1, 2, , 4]), the empty values should be ignored while traversing the array (i.e. they should not be in the resulting array).

Implement `Array.prototype.filter`. To avoid overwriting the actual `Array.prototype.filter`, which is being used by the autograder, implement it as `Array.prototype.myFilter`.

## Example(s)

```ts
[1, 2, 3, 4].myFilter((value) => value % 2 == 0); // [2, 4]
[1, 2, 3, 4].myFilter((value) => value < 3); // [1, 2]
```

## Notes

The filter callback function receives more than just the element. `Array.prototype.filter` also accepts a second parameter. Read the specification for `Array.prototype.filter` on MDN Docs before attempting.

## Reference(s)

- Array [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

## Solution(s)

```ts
Array.prototype.myFilter = function (callbackFn, thisArg) {
  const array = this;
  const context = thisArg == null ? globalThis : Object(thisArg);
  const length = array.length;

  const returnValue = [];

  for (let i = 0; i < length; i++) {
    // ignore empty slot
    if (i in array) {
      const elem = array[i];

      if (callbackFn.call(context, elem, i, array)) {
        returnValue.push(elem);
      }
    }
  }

  return returnValue;
};
```
