# Map Async

By now you'd be familiar with mapping elements in an array. If you aren't, please do the `Array.prototype.map` question first.

What if the mapping function is not a synchronous function, i.e., it returns a promise? `Array.prototype.map` assumes the mapping function is synchronous and will fail to work properly.

Implement a function `mapAsync` that accepts an array of items and maps each element with an asynchronous mapping function. The function should return a `Promise` that resolves to the mapped results. If any promise returned by the mapping function rejects, the returned promise should reject with the same reason.

## Example(s)

```ts
const asyncDouble = (x: number) =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve(x * 2);
    }, 10);
  });

const doubled = await mapAsync([1, 2], asyncDouble);
console.log(doubled); // [2, 4]
```

## Reference(s)

- Array [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

## Solution(s)

```ts
export default function mapAsync<T, U>(
  iterable: Array<T>,
  callbackFn: (value: T) => Promise<U>,
): Promise<Array<U>> {
  return new Promise<U[]>((resolve, reject) => {
    if (iterable.length === 0) {
      resolve([]);
    }

    let resolvedCount: number = 0;
    const results = new Array<U>(iterable.length);

    function fulfilled(index: number, result: U) {
      resolvedCount++;
      results[index] = result;

      if (resolvedCount === iterable.length) {
        resolve(results);
      }
    }

    for (let i = 0; i < iterable.length; i++) {
      const elem = iterable[i];

      const promise = callbackFn(elem);

      promise.then(fulfilled.bind(null, i)).catch(reject);
    }
  });
}
```
