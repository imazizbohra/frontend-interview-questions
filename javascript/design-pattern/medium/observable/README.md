# Observable

Observables are a way to model values that arrive over time. Unlike an event emitter, a basic observable is usually cold: each call to `subscribe()` starts a fresh execution of the source.

Implement a simplified `Observable` class with support for `next`, `error`, `complete`, and unsubscribing.

`Observable` API
`new Observable(subscribeFn)`
Creates an `Observable` instance.

`subscribeFn` receives a `subscriber` object with the following methods:

- `next(value)`
- `error(error)`
- `complete()`

Do not call `subscribeFn` inside the constructor. It should run only when `subscribe()` is called, and it should run once per subscription.

`observable.subscribe(observer)`
Subscribes to the observable and returns a subscription object with an `unsubscribe()` method.

`observer` can be either:

- A function, treated as the `next` handler
- An object with optional `next`, `error`, and `complete` methods

After a subscription has been closed by `error()`, `complete()`, or `unsubscribe()`, later notifications should be ignored.

## Examples

This example shows that values emitted after complete() should be ignored.

```ts
const observable = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.complete();
  subscriber.next(3);
});

const events = [];

observable.subscribe({
  next(value) {
    events.push(`next:${value}`);
  },
  complete() {
    events.push('complete');
  },
});

events; // ['next:1', 'next:2', 'complete']
```

This example shows that the observable is cold, so each subscription runs the source independently.

```ts
let runs = 0;

const observable = new Observable((subscriber) => {
  runs += 1;
  subscriber.next(runs);
});

const first = [];
const second = [];

observable.subscribe((value) => first.push(value));
observable.subscribe((value) => second.push(value));

first; // [1]
second; // [2]
```

This example shows that calling unsubscribe() prevents future asynchronous notifications from being delivered.

```ts
const observable = new Observable((subscriber) => {
  setTimeout(() => subscriber.next('later'), 10);
});

const values = [];
const subscription = observable.subscribe((value) => values.push(value));

subscription.unsubscribe();

// After the timeout:
values; // []
```

## Notes

- This question focuses on subscription state and notification delivery.
- `unsubscribe()` only needs to stop future notifications for that subscription. Source cleanup is out of scope here and is added in the follow-up question.
- You do not need to support `pipe()`, `operators`, `Observable.from()`, `Subject`, or callback overloads like `subscribe(next, error, complete)`.
- You do not need to handle exceptions thrown by `subscribeFn` or observer callbacks.

## References

- Observable [rxjs.dev](https://rxjs.dev/guide/observable)

## Solutions

```ts
export interface Observer<T = unknown> {
  next?(value: T): void;
  error?(error: unknown): void;
  complete?(): void;
}

export interface Subscriber<T = unknown> {
  next(value: T): void;
  error(error: unknown): void;
  complete(): void;
}

export interface Subscription {
  unsubscribe(): void;
}

export type SubscribeInput<T = unknown> = Observer<T> | ((value: T) => void);
export type SubscribeFn<T = unknown> = (subscriber: Subscriber<T>) => void;

function normalizeObserver<T>(observer?: SubscribeInput<T>): Observer<T> {
  if (typeof observer === "function") {
    return { next: observer };
  }

  return observer ?? {};
}

export default class Observable<T = unknown> {
  _subscribeFn: SubscribeFn<T>;

  constructor(subscribeFn: SubscribeFn<T>) {
    this._subscribeFn = subscribeFn;
  }

  subscribe(observer?: SubscribeInput<T>): Subscription {
    const normalizedObserver = normalizeObserver(observer);
    let closed = false;

    // Once the stream is closed, every callback becomes a no-op.
    const subscriber: Subscriber<T> = {
      next: (value: T) => {
        if (closed) {
          return;
        }

        normalizedObserver.next?.(value);
      },

      error: (error: unknown) => {
        if (closed) {
          return;
        }

        closed = true;
        normalizedObserver.error?.(error);
      },

      complete: () => {
        if (closed) {
          return;
        }

        closed = true;
        normalizedObserver.complete?.();
      },
    };

    this._subscribeFn(subscriber);

    return {
      unsubscribe() {
        closed = true;
      },
    };
  }
}
```
