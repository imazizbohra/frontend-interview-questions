# Array.prototype.reduce

`Array.prototype.reduce` is a way of "reducing" elements in an array by calling a "reducer" callback function on each element in order and passing along the return value from the previous callback. The final result of running the reducer across all elements of the array is a single value.

Implement `Array.prototype.reduce`. To avoid overwriting the actual `Array.prototype.reduce`, which is being used by the autograder, implement it as `Array.prototype.myReduce`.

## Example(s)

```ts
[1, 2, 3].myReduce((prev, curr) => prev + curr, 0); // 6
[1, 2, 3].myReduce((prev, curr) => prev + curr, 4); // 10
```

## Notes

There are some nuances in how `Array.prototype.reduce` works and what values are passed to the reducer callback. Read the specification for `Array.prototype.reduce` on MDN Docs before attempting.

## Reference(s)

- Array [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

## Solution(s)

```ts
// Approach 1

Array.prototype.myReduce = function (callbackFn, initialValue) {
  const array = this;
  const length = array.length;

  if (length === 0 && arguments.length === 1) {
    throw new TypeError("Reduce of empty array with no initial value");
  }

  if (Object.keys(array).length === 0 && arguments.length === 1) {
    throw new TypeError("Reduce of empty array with no initial value");
  }

  let acc = initialValue as any;
  let noInitialValue = arguments.length <= 1;

  for (let i = 0; i < length; i++) {
    if (i in array) {
      if (noInitialValue) {
        acc = array[i];
        noInitialValue = false;
      } else {
        acc = callbackFn.call(null, acc, array[i], i, array);
      }
    }
  }

  return acc;
};
```

```ts
// Approach 2

Array.prototype.myReduce = function (callbackFn, initialValue) { 
  const len = this.length;
  const noInitialValue = arguments.length <= 1;

  let k = 0;
  let acc: any;

  if (noInitialValue) {
    // ignore empty slots
    while (k < len && !(k in this)) k++;
    if (k >= len) {
      throw new TypeError('Reduce of empty array with no initial value');
    }

    acc = this[k];
    k++;
  } else {
    acc = initialValue;
  }

  for (; k < len; k++) {
    if (k in this) {
      acc = callbackFn(acc, this[k], k, this);
    }
  }

  return acc;
};
```
