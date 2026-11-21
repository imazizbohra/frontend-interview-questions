# Deep Clone II

**Note**: This is an advanced version of the [Deep Clone](../deep-clone/) question, which you should complete before attempting this question.

It is not realistic to expect candidates to produce a complete deep clone solution in typical interview settings, though the interviewer might ask you for a simpler version like [Deep Clone](../deep-clone/) and discuss the edge cases and how you would handle them.

Implement a `deepClone` function that performs a deep clone as thoroughly as possible, while also handling the following:

- The input object can contain any data type.
- If the input object is cyclic, clone its circular references as well.

## Behavior guide

Use this as the interview scope for the clone:

| Value kind | Expected behavior |
| --- | --- |
| Primitives and `null` | Return the value directly |
| Functions | Return the same function reference |
| Arrays | Return new arrays and recursively clone their contents |
| Plain objects and other ordinary object instances | Return new objects with the same prototype and recursively clone their own properties |
| `Date` and `RegExp` | Return equivalent new instances |
| `Map` and `Set` | Return new containers; clone Set entries and Map values while preserving Map keys |
| Symbol keys on objects | Preserve the symbol-keyed properties |
| Circular object references | Preserve the cycle in the cloned object graph |

You do not need to fully support every browser or platform object. Values such as DOM nodes, promises, weak collections, errors, and typed arrays are outside the intended interview scope.

## Examples

```ts
const obj1 = {
  num: 0,
  str: '',
  boolean: true,
  unf: undefined,
  nul: null,
  obj: { name: 'foo', id: 1 },
  arr: [0, 1, 2],
  date: new Date(),
  reg: new RegExp('/bar/ig'),
  [Symbol('s')]: 'baz',
};

const clonedObj1 = deepClone(obj1);
clonedObj1.arr.push(3);
obj1.arr; // Should still be [0, 1, 2]

const obj2 = { a: {} };
obj2.a.b = obj2; // Circular reference

const clonedObj2 = deepClone(obj2); // Should not recurse infinitely or cause a stack overflow.

clonedObj2.a.b = 'something new';

obj2.a.b === obj2; // This should still be true
```

## References

- Concept [deep copy](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy)
- Window [structuredClone](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone)
- Lodash [_.cloneDeep](https://lodash.com/docs/4.18.1#cloneDeep)

## Solutions

```ts
export default function deepClone<T>(value: T): T {
  function clone<T>(value: T, cache: Map<T, T>): T {
    // cache hit
    if (cache.has(value)) {
      return cache.get(value) as T;
    }

    // handle date
    if (value instanceof Date) {
      return new Date(value) as T;
    }

    // handle RegExp
    if (value instanceof RegExp) {
      return new RegExp(value) as T;
    }

    // handle array
    if (Array.isArray(value)) {
      const clonedArray = [] as T[];
      cache.set(value, clonedArray as T);

      value.forEach((value_) => {
        clonedArray.push(clone(value_, cache));
      });

      return clonedArray as T;
    }

    // handle set
    if (value instanceof Set) {
      const clonedSet = new Set<T>();
      cache.set(value, clonedSet as T);

      value.forEach((value_) => {
        clonedSet.add(clone(value_, cache));
      });

      return clonedSet as T;
    }

    // handle map
    if (value instanceof Map) {
      const clonedMap = new Map<T, T>();
      cache.set(value, clonedMap as T);

      value.forEach((value_, key) => {
        clonedMap.set(key, clone(value_, cache));
      });

      return clonedMap as T;
    }

    // handle object
    if (typeof value === "object" && value !== null) {
      const clonedObj = Object.create(Object.getPrototypeOf(value));
      cache.set(value, clonedObj);

      // `Reflect.ownKeys()` includes symbol keys too.
      for (const key of Reflect.ownKeys(value as Object)) {
        const value_ = (value as any)[key];
        clonedObj[key] = clone(value_, cache);
      }
      return clonedObj;
    }

    // return as-is if the value primitive or function
    return value;
  }

  return clone(value, new Map());
}
```
