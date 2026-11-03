# Singleton

The Singleton pattern is used when you want to ensure that a class has only one instance and to provide a global point of access to that instance. The Singleton pattern is useful for:

- **Single point of control:** When you need a single point of control for actions, such as managing a configuration object, logging, or controlling access to a shared resource.
- **Global state:** When you want to maintain global state that should be accessible from different parts of your application. This is especially useful when sharing common data across various components.
- **Preventing duplicate initialization:** Initialize a shared resource only once by creating a single class instance.
- **Caching:** Implementing a caching mechanism where you want to maintain a single cache instance throughout the application.

Implement the GlobalMap module in JavaScript using the Singleton pattern. The GlobalMap module exports an object that has a single getInstance() method that returns a Map object that can be used as a key/value store for global caching/memoization.

## Example(s)

```ts
// fileA.js
import GlobalMap from './GlobalMap';

const gbMap = GlobalMap.getInstance();
gbMap.set('count', 42);
```

Somewhere else in another file, executed after the first:

```ts
// fileB.js
import GlobalMap from './GlobalMap';

const gbMap = GlobalMap.getInstance();
console.log(gbMap.get('count')); // 42
```

## Reference(s)

- patterns.dev [singleton-pattern](https://www.patterns.dev/vanilla/singleton-pattern/)

## Solution(s)

```ts
class GlobalMap {
  #instance: Map<any, any>;
  
  constructor () {
    this.#instance = new Map<any, any>();
  }

  getInstance() {
    return this.#instance;
  }
}

export default new GlobalMap();
```
