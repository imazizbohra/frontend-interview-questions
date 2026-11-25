# Promise II

This is a follow-up to [Promise](../../medium/promise/). In the first part, `MyPromise` could settle and notify `.then()` handlers asynchronously. In this part, you will add the behavior that makes promises composable: `.then()` should return a new promise.

Implement a class named `MyPromise`.

## Requirements

Keep the behavior from Part I:

- The executor runs immediately.
- An executor error rejects the promise if it has not already settled.
- A promise can be pending, fulfilled, or rejected.
- `resolve` and `reject` settle the promise only once.
- `.then()` handlers run asynchronously.
- Multiple and late `.then()` handlers are supported.

Add the following behavior:

`promise.then(onFulfilled, onRejected)`
`.then()` should return a new `MyPromise`.

- If `onFulfilled` returns a value, the returned promise fulfills with that value.
- If `onRejected` returns a value, the returned promise fulfills with that value.
- If either handler throws, the returned promise rejects with the thrown error.
- If a handler is omitted, the previous fulfillment or rejection should propagate.
- If a handler returns another `MyPromise`, the returned promise should adopt that MyPromise's eventual state.

`promise.catch(onRejected)`
`.catch(onRejected)` should behave like `.then(undefined, onRejected)`.

## Notes

- This is still a simplified interview implementation.
- Generic thenable assimilation and full Promises/A+ self-resolution handling are out of scope.
- Static methods are introduced in [Promise III](../promise-3/).

## Examples

```ts
const result = await new MyPromise((resolve) => {
  resolve(2);
})
  .then((value) => value + 3)
  .then((value) => value * 4);

console.log(result); // 20
```

```ts
const result = await new MyPromise((resolve, reject) => {
  reject('failed');
}).catch((reason) => `Recovered from ${reason}`);

console.log(result); // 'Recovered from failed'
```

## References

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)

## Solutions

```ts
type PromiseCallback = (value: any) => unknown;

type Handler = {
  onFulfilled: PromiseCallback;
  onRejected: PromiseCallback;
  resolve: PromiseCallback;
  reject: PromiseCallback;
};

const PROMISE_STATE = {
  PENDING: "PENDING",
  FULFILLED: "FULFILLED",
  REJECTED: "REJECTED",
};

export default class MyPromise {
  #state: (typeof PROMISE_STATE)[keyof typeof PROMISE_STATE];
  #result: any;
  #handlers: Handler[];

  constructor(
    executor: (resolve: PromiseCallback, reject: PromiseCallback) => void,
  ) {
    this.#state = PROMISE_STATE.PENDING;
    this.#result = undefined;
    this.#handlers = [];

    try {
      executor(this.#resolve.bind(this), this.#reject.bind(this));
    } catch (error) {
      this.#reject(error);
    }
  }

  #resolve(value: any) {
    if (this.#state !== PROMISE_STATE.PENDING) {
      return;
    }

    this.#state = PROMISE_STATE.FULFILLED;
    this.#result = value;
    this.#executeHandlers();
  }

  #reject(reason: any) {
    if (this.#state !== PROMISE_STATE.PENDING) {
      return;
    }

    this.#state = PROMISE_STATE.REJECTED;
    this.#result = reason;
    this.#executeHandlers();
  }

  #executeHandlers() {
    if (this.#state === PROMISE_STATE.PENDING) {
      return;
    }

    this.#handlers.forEach(({ onFulfilled, onRejected, resolve, reject }) => {
      queueMicrotask(() => {
        try {
          const handler =
            this.#state === PROMISE_STATE.FULFILLED ? onFulfilled : onRejected;
          const valueOrPromise = handler(this.#result);

          if (valueOrPromise instanceof MyPromise) {
            valueOrPromise.then(resolve, reject);
          } else {
            resolve(valueOrPromise);
          }
        } catch (error) {
          reject(error);
        }
      });
    });
    this.#handlers = [];
  }

  then(onFulfilled?: PromiseCallback, onRejected?: PromiseCallback): MyPromise {
    const promise = new MyPromise((resolve, reject) => {
      const handler: Handler = {
        onFulfilled:
          onFulfilled ??
          ((value: any) => {
            return value;
          }),
        onRejected:
          onRejected ??
          ((error: any) => {
            throw error;
          }),
        resolve: resolve,
        reject: reject,
      };

      this.#handlers.push(handler);
      this.#executeHandlers();
    });

    return promise;
  }

  catch(onRejected?: PromiseCallback): MyPromise {
    return this.then(undefined, onRejected);
  }
}
```
