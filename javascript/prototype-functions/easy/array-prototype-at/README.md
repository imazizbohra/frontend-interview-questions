# Array.prototype.at

`Array.prototype.at` takes an integer value and returns the item at that index, allowing for positive and negative integers. Negative integers count back from the last item in the array.

Implement `Array.prototype.at`. To avoid overwriting the actual `Array.prototype.at`, implement it as `Array.prototype.myAt`.

## Example(s)

```ts
const arr = [42, 79];
arr.myAt(0); // 42
arr.myAt(1); // 79
arr.myAt(2); // undefined

arr.myAt(-1); // 79
arr.myAt(-2); // 42
arr.myAt(-3); // undefined
```

## Notes

There are some tricky cases to handle. Read the specification for Array.prototype.at on MDN Docs before attempting.

## Reference(s)

- Array [at](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at)

## Solution(s)

```ts
Array.prototype.myAt = function (index: number) {
  const array = this;
  const length = array.length;

  // out-of-bound index
  if (index < -length || index >= length) {
    return undefined;
  }

  // normalized index
  const nIndex =
    index < 0 ? Math.max(index + length, 0) : Math.min(index, length);

  return array[nIndex];
};
```
