# Array.prototype.concat

The `Array.prototype.concat` method on JavaScript arrays is used to merge two or more arrays. This method does not change the existing arrays; instead, it returns a new array.

Implement `Array.prototype.concat`. To avoid overwriting the actual `Array.prototype.concat`, which is being used by the autograder, implement it as `Array.prototype.myConcat`.

## Example(s)

```ts
[1, 2, 3].myConcat([4, 5, 6]); // [1, 2, 3, 4, 5, 6]
[1, 2, 3].myConcat(4, 5, 6); // [1, 2, 3, 4, 5, 6]
[1, 2, 3].myConcat(4, [5, 6]); // [1, 2, 3, 4, 5, 6]
```

## Notes

As shown in the examples, `Array.prototype.concat` accepts a variable number of arguments, which are handled differently depending on whether each argument is a primitive or an array/array-like object. Arguments with the property `Symbol.isConcatSpreadable` set are out of scope, but read the specification for `Array.prototype.concat` on MDN Docs before attempting.

## Reference(s)

- Array [concat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)

## Solution(s)

```ts
Array.prototype.myConcat = function (...items) {
  const arr1 = this;
  const arr2 = items;

  const returnValue = new Array(arr1.length);

  // handle 1st array
  let i = 0;
  while (i < arr1.length) {
    // non-empty slot
    if (i in arr1) {
      returnValue[i] = arr1[i];
    } else {
      // empty slot
      returnValue.length++;
    }
    i++;
  }

  // handle 2nd array
  for (let j = 0; j < arr2.length; j++) {
    const elem = arr2[j];

    if (Array.isArray(elem)) {
      for (let k = 0; k < elem.length; k++) {
        // non-empty slot
        if (k in elem) {
          returnValue[i] = elem[k];
        } else {
          // empty slot
          returnValue.length++;
        }
        i++;
      }
    } else {
      returnValue[i] = elem;
      i++;
    }
  }

  return returnValue;
};
```
