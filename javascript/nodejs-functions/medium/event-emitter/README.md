# Event Emitter

An `EventEmitter` lets code subscribe to named events and invokes the registered callbacks when those events are emitted.

An upload component, for example, can emit a progress event while a progress bar and a logger each react independently. The upload code does not need direct references to every consumer. Subscriptions also have a lifecycle: when a consumer no longer needs updates, it should be able to remove its callback without affecting other listeners.

Implement an `EventEmitter` class similar to the one in [Node.js](https://nodejs.org/api/events.html). Preserve listener order, duplicate registrations, and instance isolation.

Example usage of the `EventEmitter` class:

## Example(s)

```ts
const emitter = new EventEmitter();

function addTwoNumbers(a, b) {
  console.log(`The sum is ${a + b}`);
}
emitter.on('foo', addTwoNumbers);
emitter.emit('foo', 2, 5);
// > "The sum is 7"

emitter.on('foo', (a, b) => console.log(`The product is ${a * b}`));
emitter.emit('foo', 4, 5);
// > "The sum is 9"
// > "The product is 20"

emitter.off('foo', addTwoNumbers);
emitter.emit('foo', -3, 9);
// > "The product is -27"
```

## Notes

- Invoke `listeners` in registration order.
- Registering the same listener more than once creates separate subscriptions, so it is invoked once per registration. One - `off()` call removes only one matching registration.
- Calling `off()` for a missing event or listener is a no-op.
- Event names are arbitrary strings and can match inherited object property names such as `toString`.

## Reference(s)

- NodeJs [events](https://nodejs.org/api/events.html)

## Solution(s)

```ts
interface IEventEmitter {
  on(eventName: string, listener: Function): IEventEmitter;
  off(eventName: string, listener: Function): IEventEmitter;
  emit(eventName: string, ...args: Array<any>): boolean;
}

type EventListenersMap = Map<string, Function[]>;

// You are free to use alternative approaches of
// instantiating the EventEmitter as long as the
// default export has the same interface.
export default class EventEmitter implements IEventEmitter {
  #map: EventListenersMap;

  constructor() {
    this.#map = new Map();
  }

  on(eventName: string, listener: Function): IEventEmitter {
    if (!eventName || !listener) {
      return this;
    }

    if (!this.#map.has(eventName)) {
      this.#map.set(eventName, []);
    }

    const listeners = this.#map.get(eventName) as Function[];
    listeners.push(listener);
    this.#map.set(eventName, listeners);

    return this;
  }

  off(eventName: string, listener: Function): IEventEmitter {
    if (!eventName || !listener || !this.#map.has(eventName)) {
      return this;
    }

    const listeners = this.#map.get(eventName) as Function[];
    const index = listeners.findIndex((elem) => elem === listener);
    if (index < 0) {
      return this;
    }
    listeners.splice(index, 1);
    if (listeners.length === 0) {
      this.#map.delete(eventName);
    } else {
      this.#map.set(eventName, listeners);
    }

    return this;
  }

  emit(eventName: string, ...args: Array<any>): boolean {
    if (!eventName || !this.#map.has(eventName)) {
      return false;
    }

    const listeners = this.#map.get(eventName) as Function[];
    listeners.forEach((listener) => {
      listener.call(this, ...args);
    });

    return true;
  }
}
```
