# Promise III

This is a follow-up to [Promise II](../promise-2/). In this part, you will add convenience APIs that make the simplified MyPromise class closer to the native Promise API.

Implement a class named `MyPromise`.

## Requirements

Keep the behavior from the previous parts:

- The executor runs immediately.
- An executor error rejects the promise if it has not already settled.
- A promise settles only once.
- `.then()` handlers run asynchronously.
- `.then()` returns a new MyPromise and supports chaining.
- `.catch(onRejected)` behaves like `.then(undefined, onRejected)`.

Add the following behavior:

`promise.finally(onFinally)`
Runs `onFinally` when the promise settles.

- If the original promise fulfills, `.finally()` should fulfill with the original value.
- If the original promise rejects, `.finally()` should reject with the original reason.
- If `onFinally` returns a fulfilling `MyPromise`, wait for it before preserving the original outcome.
- If `onFinally` throws or returns a rejecting `MyPromise`, the promise returned by `.finally()` should reject with that new reason.
- If `onFinally` is omitted, `.finally()` should pass the original outcome through unchanged.

`MyPromise.resolve(value)`

Returns a fulfilled `MyPromise` for `value`. If `value` is already a `MyPromise`, return it unchanged.

`MyPromise.reject(reason)`

Returns a rejected `MyPromise` for `reason`.

`MyPromise.all(values)`

Accepts an array of values and promises. It returns a `MyPromise` that:

- fulfills with an array of fulfilled values in input order
- fulfills with `[]` for an empty array
- rejects as soon as any input rejects

`MyPromise.race(values)`

Accepts an array of values and promises. It returns a `MyPromise` that settles with the first input to settle.

## Notes

- The static collection helpers only need to accept arrays, not generic iterables.
- Full Promises/A+ compliance is still out of scope for this interview version.
- For additional static helpers, see the existing `Promise.allSettled`, `Promise.any`, and `Promise.withResolvers` questions.

## Examples

```ts
const value = await MyPromise.resolve(42).finally(() => {
  console.log('done');
});

console.log(value); // 42
```

```ts
const result = await MyPromise.all([
  MyPromise.resolve(1),
  2,
  new MyPromise((resolve) => {
    setTimeout(() => resolve(3), 100);
  }),
]);

console.log(result); // [1, 2, 3]
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

  static resolve(value?: any): MyPromise {
    if (value instanceof MyPromise) {
      return value;
    }

    return new MyPromise((resolve) => {
      resolve(value);
    });
  }

  static reject(reason?: any): MyPromise {
    return new MyPromise((_, reject) => {
      reject(reason);
    });
  }

  static all(values: Array<any>): MyPromise {
    return new MyPromise((resolve, reject) => {
      if (values.length === 0) {
        resolve([]);
        return;
      }

      let resolvedCount: number = 0;
      const results = new Array<any>(values.length);

      function fulfilled(index: number, result: any) {
        resolvedCount++;
        results[index] = result;

        if (resolvedCount === values.length) {
          resolve(results);
        }
      }

      for (let i = 0; i < values.length; i++) {
        const elem = values[i];
        const promise = MyPromise.resolve(elem);
        promise.then(fulfilled.bind(null, i)).catch(reject);
      }
    });
  }

  static race(values: Array<any>): MyPromise {
    return new MyPromise((resolve, reject) => {
      values
        .map((elem) => MyPromise.resolve(elem))
        .forEach((promise) => {
          promise.then(
            (value) => resolve(value),
            (reason) => reject(reason),
          );
        });
    });
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

  finally(onFinally?: () => unknown): MyPromise {
    // wait for the cleanup fun to finish then settle next promise in the chain
    // if onFinally() returns primitive -> immediate settle -> swallow
    // if onFinally() returns promise -> wait for it -> swallow
    // if onFinally() throws error -> catch by #executeHandlers catch block
    return this.then(
      (value: unknown) =>
        MyPromise.resolve(onFinally ? onFinally() : value).then(() => {
          return value;
        }),
      (reason: unknown) =>
        MyPromise.resolve(onFinally ? onFinally() : reason).then(() => {
          throw reason;
        }),
    );
  }
}
```
