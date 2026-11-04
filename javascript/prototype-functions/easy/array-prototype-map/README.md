# Array.prototype.map

`Array.prototype.map` creates a new array populated with the results of calling a provided function on every element in the calling array.

Implement `Array.prototype.map`. To avoid overwriting the actual `Array.prototype.map`, which is being used by the autograder, implement it as `Array.prototype.myMap`.

## Example(s)

```ts
[1, 2, 3, 4].myMap((i) => i); // [1, 2, 3, 4]
[1, 2, 3, 4].myMap((i) => i * i); // [1, 4, 9, 16]
```

## Notes

The map callback function receives more than just the element. `Array.prototype.map` also accepts a second parameter. Read the specification for `Array.prototype.map` on MDN Docs before attempting.

## Reference(s)

- Array [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

## Solution(s)

```ts
Array.prototype.myMap = function (callbackFn, thisArg) {
  const array = this;
  const length = array.length;
  const context = thisArg == null ? globalThis : Object(thisArg);

  const returnValue = new Array(length);

  for (let i = 0; i < length; i++) {
    if (i in array) {
      returnValue[i] = callbackFn.call(context, array[i], i, array);
    }
  }

  return returnValue;
};
```
