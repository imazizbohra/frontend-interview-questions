# Event Emitter II

In the observer pattern (also commonly known as the publish-subscribe model), subscribers observe events emitted by publishers and execute code whenever an event happens.

Implement an `EventEmitter` class similar to the one in `Node.js` that follows such an observer pattern. The difference between this question and the first Event Emitter question is how listeners are unsubscribed. In this version, there's no `emitter.off()` method available on the `EventEmitter` instance. Instead, `emitter.on()` returns an object that has an `off()` method.

## Example(s)

```ts
const emitter = new EventEmitter();

function addTwoNumbers(a, b) {
  console.log(`The sum is ${a + b}`);
}

// Returns a subscription object that has an .off() method.
const sub = emitter.on('foo', addTwoNumbers);
emitter.emit('foo', 2, 5);
// > "The sum is 7"

emitter.on('foo', (a, b) => {
  console.log(`The product is ${a * b}`);
});
emitter.emit('foo', 4, 5);
// > "The sum is 9"
// > "The product is 20"

sub.off(); // This unsubscribes the callback that logs the sum of the numbers.
emitter.emit('foo', -3, 9);
// > "The product is -27"
// (Only the multiplication callback is triggered; the first one was unsubscribed.)
```

## Notes

- Node.js's `EventEmitter` implementation allows eventNames to be symbols, which are not allowed here.
- Unlike the first `EventEmitter` question, `on()` returns a subscription object instead of the emitter instance, so method chaining through `on()` is no longer the focus.

## Reference(s)

- NodeJs [events](https://nodejs.org/api/events.html)

## Techniques

- Object-oriented programming
- Using the right data structures
- Closures
- Handling variadic arguments

## Solution(s)

```ts
interface IEventEmitter {
  on(eventName: string, listener: Function): { off: () => void };
  emit(eventName: string, ...args: Array<any>): boolean;
}

// You are free to use alternative approaches of
// instantiating the EventEmitter as long as the
// default export is correct.
export default class EventEmitter implements IEventEmitter {
  _key: number;
  _events: Record<string, Record<string, Function>>;

  constructor() {
    // Avoid creating objects via `{}` to exclude unwanted properties
    // on the prototype (such as `.toString`).
    this._events = Object.create(null);
    // Use an incrementing number to uniquely identify each listener.
    this._key = 0;
  }

  on(eventName: string, listener: Function): { off: () => void } {
    if (!Object.hasOwn(this._events, eventName)) {
      // It's ok to use `{}` here since the keys will just be numbers.
      this._events[eventName] = {};
    }

    // Each subscription gets its own id so duplicate listeners can be removed independently.
    const listenerId = this._key;
    this._events[eventName][listenerId] = listener;
    this._key++;

    return {
      // Use arrow function so that `this` is preserved.
      off: () => {
        delete this._events[eventName][listenerId];
      },
    };
  }

  emit(eventName: string, ...args: Array<any>): boolean {
    // Return false for non-existing eventNames or events without listeners.
    if (
      !Object.hasOwn(this._events, eventName) ||
      Object.keys(this._events[eventName]).length === 0
    ) {
      return false;
    }

    // Make a clone of the listeners in case one of the
    // listeners calls sub.off() and changes the listeners.
    const listeners = { ...this._events[eventName] };
    Object.values(listeners).forEach((listener) => {
      listener.apply(null, args);
    });

    return true;
  }
}
```
