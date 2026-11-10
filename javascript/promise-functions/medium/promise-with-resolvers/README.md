# Promise.withResolvers

The `Promise.withResolvers()` static method returns an object containing a new Promise object and two functions to resolve or reject it, corresponding to the two parameters passed to the executor of the `Promise()` constructor.

This pattern can improve code readability and make it easier to manage asynchronous operations, especially when you need to resolve or reject the Promise from different parts of your code.

## Example(s)

```ts
function delayedGreeting(name) {
  const { promise, resolve, reject } = Promise.withResolvers();

  setTimeout(() => {
    if (name) {
      resolve(`Hello, ${name}!`);
    } else {
      reject(new Error('Name is required.'));
    }
  }, 1000);

  return promise;
}

delayedGreeting('Alice')
  .then((message) => console.log(message)) // Output: Hello, Alice!
  .catch((error) => console.error(error));

delayedGreeting()
  .then((message) => console.log(message))
  .catch((error) => console.error(error)); // Output: Error: Name is required.
```

```ts
const { promise, resolve, reject } = promiseWithResolvers();

// Later in your code
resolve('Success!');
promise.then((result) => console.log(result)); // Output: Success!
```

```ts
function timeoutPromise(duration) {
  const { promise, resolve, reject } = promiseWithResolvers();

  setTimeout(() => reject(new Error('Timeout')), duration);

  return promise;
}

timeoutPromise(2000).catch((error) => console.error(error.message)); // Output after 2 seconds: Timeout
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- Promise [withResolvers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/withResolvers)

## Solution(s)

```ts
export default function promiseWithResolvers<T>(): Readonly<{
  promise: Promise<T>;
  resolve: (value: T) => void;
  reject: (reason?: any) => void;
}> {
  let resolve: (value: T) => void = () => {};
  let reject: (reason?: any) => void = () => {};

  const promise: Promise<T> = new Promise<T>((res, rej) => {
    resolve = res;
    reject = rej;
  });

  const obj: Readonly<{
    promise: Promise<T>;
    resolve: (value: T) => void;
    reject: (reason?: any) => void;
  }> = {
    promise,
    resolve,
    reject,
  };

  return obj;
}
```
