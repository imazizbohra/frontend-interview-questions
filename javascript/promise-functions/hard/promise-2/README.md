# Promise

JavaScript `Promises` are the foundation for most asynchronous code. In this question, you will implement the first part of a simplified `Promise` class to understand the state machine behind fulfillment, rejection, and `.then()` callbacks.

Implement a class named `MyPromise`.

## Requirements

`new MyPromise(executor)`
Creates a promise-like object. The `executor` function receives two functions:

| Parameter | Type | Description |
| --- | --- | --- |
| `resolve` | Function | Fulfills the promise with a value. |
| `reject` | Function | Rejects the promise with a reason. |

The executor should run immediately. If it throws before settlement, reject with the thrown error. Calling `resolve` or `reject` should settle the promise only once, so a later call or executor error cannot replace the first result.

`promise.then(onFulfilled, onRejected)`
Registers callbacks that run after the promise settles.

| Parameter | Type | Description |
| --- | --- | --- |
| `onFulfilled` | Function | Called with the fulfilled value. |
| `onRejected` | Function | Called with the rejection reason. |

Handlers must run asynchronously, even if the promise has already settled. Multiple `.then()` calls should all be notified, and handlers added after settlement should still run.

Either handler may be omitted. Handler return values do not change the original promise's outcome. This part does not assess `.then()`'s return value or promise chaining; dependent promise chaining is introduced in Part II.

## Notes

- This is an interview-focused simplification, not a full Promises/A+ implementation.
- Chaining behavior is introduced in Promise II.
- Static helper methods are introduced in Promise III.

## Examples

```ts
const promise = new MyPromise((resolve) => {
  resolve(42);
});

promise.then((value) => {
  console.log(value); // 42
});
```

```ts
const promise = new MyPromise((resolve) => {
  resolve(42);
});

promise.then((value) => {
  console.log(value); // 42
});
```

## References

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)

## Solutions

```ts
type PromiseCallback = (value: any) => unknown;

const PROMISE_STATE = {
  PENDING: "PENDING",
  FULFILLED: "FULFILLED",
  REJECTED: "REJECTED",
};

type Handler = {
  onFulfilled: PromiseCallback;
  onRejected: PromiseCallback;
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

    this.#handlers.forEach(({ onFulfilled, onRejected }) => {
      queueMicrotask(() => {
        const handler =
          this.#state === PROMISE_STATE.FULFILLED ? onFulfilled : onRejected;
        handler(this.#result);
      });
    });
    this.#handlers = [];
  }

  then(onFulfilled?: PromiseCallback, onRejected?: PromiseCallback): void {
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
    };

    this.#handlers.push(handler);

    this.#executeHandlers();
  }
}
```
