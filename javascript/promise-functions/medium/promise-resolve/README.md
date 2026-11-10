# Promise.resolve

The `Promise.resolve()` static method "resolves" a given value to a Promise. If the value is:

- A native promise, return that promise.
- A non-thenable, return a promise that is already fulfilled with that value.
- A thenable, `Promise.resolve()` will call the `then()` method and pass a pair of resolving functions as arguments. A promise that has the same state as the thenable is returned.

A "thenable" is an interface that implements the `.then()` method, which is called with two callbacks: one for when the promise is fulfilled, one for when it's rejected. Promises are also thenables.

Implement the `Promise.resolve()` function as `promiseResolve`. You can ignore the case where this is referenced within the implemented function.

## Example(s)

```ts
// Resolving a non-promise.
const p = promiseResolve(42);
await p; // 42

// Resolving a Promise.
const original = new Promise((resolve) => resolve(42));
const cast = promiseResolve(original);
await cast; // 42

// Resolving a thenable.
const resolvedThenable = promiseResolve({
  then(resolve, reject) {
    resolve(42);
  },
});
await resolvedThenable; // 42
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)

## Solution(s)

```ts
export default function promiseResolve(value: any): Promise<any> {
  // if value is an instance of promise
  if (value instanceof Promise) {
    return value;
  }

  // if a value is theneable
  if (
    value !== null &&
    typeof value === "object" &&
    typeof value.then === "function"
  ) {
    return new Promise((res, rej) => {
      value.then(res, rej);
    });
  }

  // any other value
  return new Promise((res) => res(value));
}
```
