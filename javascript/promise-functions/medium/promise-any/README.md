# Promise.any

**Note**: If you haven't completed the [Promise.all](../promise-all/) question, you should attempt that first.
`Promise.all()` combines multiple values or promises into one promise, preserving input order in its fulfilled results.

After implementing `Promise.all`, this is a useful contrast: `Promise.any` resolves on the first fulfillment and rejects only if every input rejects.

Implement a custom version of `Promise.any()`, a `promiseAny` function, except that the function takes an array instead of an iterable, and returned `AggregateErrors` only need to contain an array of error reasons; the message doesn't have to be set. Refer to the `AggregateError` constructor examples on [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any).

Be sure to read the description carefully and implement accordingly!

## Example(s)

```ts
const p0 = Promise.resolve(42);
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(21);
  }, 100);
});

await promiseAny([p0, p1]); // 42
```

```ts
const p0 = new Promise((resolve) => {
  setTimeout(() => {
    resolve(42);
  }, 100);
});
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('Err!');
  }, 400);
});

await promiseAny([p0, p1]); // 42
```

```ts
const p0 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(42);
  }, 400);
});
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('Err!');
  }, 100);
});

try {
  await promiseAny([p0, p1]);
} catch (err) {
  console.log(err instanceof AggregateError); // true
  console.log(err.errors); // [42, 'Err!']
}
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- Promise [any](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)
- Error [AggregateError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AggregateError)

## Techniques

- Constructing promises
- Concurrent async coordination
- Preserving input order for aggregated errors

## Solution(s)

```ts
export default function promiseAny<T>(iterable: Array<T>): Promise<T> {
  return new Promise<T>((resolve, reject) => {
    if (iterable.length === 0) {
      reject(new AggregateError([]));
    }

    let rejectedCount: number = 0;
    const errors = new Array<Error>(iterable.length);

    function rejected(index: number, error: Error) {
      rejectedCount++;
      errors[index] = error;

      // if all the promises rejects
      if (rejectedCount === iterable.length) {
        reject(new AggregateError(errors));
      }
    }

    for (let i = 0; i < iterable.length; i++) {
      const elem = iterable[i];
      const promise = Promise.resolve(elem);

      promise.then(resolve).catch(rejected.bind(null, i));
    }
  });
}
```
