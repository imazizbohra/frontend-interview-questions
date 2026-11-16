# Promise.allSettled

**Note**: If you haven't completed the [Promise.all](../promise-all/) question, you should attempt that first.

`Promise.allSettled()` is frequently used when there are multiple independent asynchronous tasks and the result of each promise is needed, regardless of whether they were fulfilled or rejected.

Implement a custom version of `Promise.allSettled()`, a `promiseAllSettled` function, except that the function takes an array instead of an iterable. Be sure to read the description carefully and implement accordingly!

## Example(s)

```ts
const p0 = Promise.resolve(3);
const p1 = 42;
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('foo');
  }, 100);
});

await promiseAllSettled([p0, p1, p2]);
// [
//   { status: 'fulfilled', value: 3 },
//   { status: 'fulfilled', value: 42 },
//   { status: 'rejected', reason: 'foo' },
// ];
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- Promise [allSettled](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

## Solution(s)

```ts
interface PromiseFulfilledResult<T> {
  status: "fulfilled";
  value: T;
}

interface PromiseRejectedResult {
  status: "rejected";
  reason: any;
}

export default function promiseAllSettled<T>(
  iterable: Array<T>,
): Promise<Array<PromiseFulfilledResult<T> | PromiseRejectedResult>> {
  return new Promise((resolve, reject) => {
    if (iterable.length === 0) {
      resolve([]);
    }

    let settledCount: number = 0;
    const results = new Array<
      PromiseFulfilledResult<T> | PromiseRejectedResult
    >(iterable.length);

    function fulfilled(index: number, result: any) {
      results[index] = {
        status: "fulfilled",
        value: result,
      };

      settled();
    }

    function rejected(index: number, reason: any) {
      results[index] = {
        status: "rejected",
        reason: reason,
      };

      settled();
    }

    function settled() {
      settledCount++;

      if (settledCount === iterable.length) {
        resolve(results);
      }
    }

    for (let i = 0; i < iterable.length; i++) {
      Promise.resolve(iterable[i])
        .then(fulfilled.bind(null, i))
        .catch(rejected.bind(null, i));
    }
  });
}
```
