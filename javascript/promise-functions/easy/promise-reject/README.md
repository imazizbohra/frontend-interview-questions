# Promise.reject

Unlike `Promise.resolve()`, `Promise.reject()` always wraps reason in a new Promise object, even when reason is already a Promise.

Implement the `Promise.reject()` function as `promiseReject`. You can ignore the case where this is referenced within the implemented function.

## Argument(s)

reason (any): Reason of rejection.

## Returns

(promise): Returns a new rejected promise.

## Example(s)

```ts
try {
  await promiseReject('Mayday!');
} catch (err) {
  console.log(err); // Mayday!
}

```

`promiseReject(reason)` returns a rejected promise; it does not throw synchronously. The rejection only reaches a catch clause once the promise is await-ed (or via .catch(...) on the returned promise).

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- Promise [reject](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject)

## Solution(s)

```ts
export default function promiseReject<T = never>(reason: any): Promise<T> {
  return new Promise((_, reject) => reject(reason));
}
```
