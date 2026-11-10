# Promise Timeout

When fetching data or performing other async operations, it is sometimes useful to set a timeout duration, i.e. require a response before the timeout and otherwise treat the request as failed.

Implement a `promiseTimeout` function that accepts a promise and a timeout duration (in milliseconds) and returns a `Promise`. If the promise argument settles within the timeout period, the returned promise settles with the promise argument's fulfillment value or rejection reason. Otherwise, the returned promise rejects with the string "Promise timeout".

## Example(s)

```ts
function fakeFetch(latency) {
  return new Promise((resolve, reject) => {
    // Simulate an asynchronous operation that resolves after `latency`.
    setTimeout(() => {
      resolve('Data successfully fetched!');
    }, latency);
  });
}

const response = await promiseTimeout(fakeFetch(1000), 2000);
console.log(response); // Data successfully fetched!

await promiseTimeout(fakeFetch(5000), 2000);
// "Promise timeout" thrown.
```

## Notes

- The original promise is not canceled when the timeout wins. Its later result is ignored by the returned promise.
- If the original promise rejects before the timeout, the returned promise should reject with that original reason.
- A duration of 0 still schedules a timer; do not reject synchronously.

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)

## Solution(s)

```ts
// Approach 1

export default function promiseTimeout<T>(
  promise: Promise<T>,
  duration: number,
): Promise<T> {
  const returnValue = new Promise<T>((res, rej) => {
    let timerId: number;

    promise.then(res, rej).finally(() => {
      clearTimeout(timerId);
    });

    timerId = setTimeout(() => {
      rej("Promise timeout");
    }, duration);
  });

  return returnValue;
}
```

```ts
// Approach 2

export default function promiseTimeout<T>(
  promise: Promise<T>,
  duration: number,
): Promise<T> {
  // Race the original promise against a promise that rejects after the deadline.
  const timeout = new Promise((_, reject) => {
    setTimeout(() => {
      reject('Promise timeout');
    }, duration);
  });

  return Promise.race([promise, timeout]) as Promise<T>;
}
```
