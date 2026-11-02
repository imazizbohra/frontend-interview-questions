# Array.prototype.square

Implement a custom Array function, Array.prototype.square(), which creates a new array with the results of squaring every element in the array that .square() is called on.

## Example(s)

```ts
[-2].square(); // [4]
[1, 2, 3, 4].square(); // [1, 4, 9, 16]
```

## Reference(s)

- Array [constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Array)
- Array [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- Statements [for](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for)

## Solution(s)

```ts
// Approach 1

interface Array<T> {
  square(): Array<number>;
}

Array.prototype.square = function () {
  const originalArray = this;
  const newArray = new Array(originalArray.length);

  for (let i = 0; i < originalArray.length; i++) {
    newArray[i] = originalArray[i] * originalArray[i];
  }

  return newArray;
};
```

```ts
// Approach 2

interface Array<T> {
  square(): Array<number>;
}

Array.prototype.square = function () {
  return this.map((el) => el * el);
};
```
