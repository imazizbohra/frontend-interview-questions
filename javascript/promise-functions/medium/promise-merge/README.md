# Promise Merge

Implement a function that accepts two promises and returns a single `Promise`. This returned promise fulfills when both input promises fulfill, with a single value according to the order and types of the fulfillment values:

- Numbers should be added together.
- Strings should be concatenated.
- Arrays should be combined into a single array.
- Plain objects should be shallow-merged into a single object, with properties from the second value overriding matching - properties from the first.
- Other types aren't supported.

The returned promise should reject if one of the following happens:

- The types of the fulfilled results do not match; reject with the string 'Unsupported data types'.
- One of the promises fails; reject with the rejected promise's reason.

## Example(s)

```ts
await promiseMerge(Promise.resolve(1), Promise.resolve(2)); // 3
await promiseMerge(Promise.resolve('abc'), Promise.resolve('def')); // 'abcdef'
await promiseMerge(Promise.resolve([1, 2, 3]), Promise.resolve([4, 5, 6])); // [1, 2, 3, 4, 5, 6]
await promiseMerge(Promise.resolve({ foo: 1 }), Promise.resolve({ bar: 2 })); // { foo: 1, bar: 2}

await promiseMerge(Promise.resolve(1), Promise.resolve([])); // Rejected with 'Unsupported data types'
await promiseMerge(Promise.reject(1), Promise.resolve(2)); // Rejected with 1
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)

## Techniques

- Determine the type of value
- Determine if an object is POJO

## Solution(s)

```ts
// Approach 1

const isPlainObject = (value: unknown): boolean => {
  // For null and undefined.
  if (value == null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
};

export default function promiseMerge(
  p1: Promise<unknown>,
  p2: Promise<unknown>,
): Promise<unknown> {
  const promise = new Promise<unknown>((res, rej) => {
    Promise.all([p1, p2])
      .then(([r1, r2]) => {
        if (typeof r1 === "number" && typeof r2 === "number") {
          res(r1 + r2);
        }

        if (typeof r1 === "string" && typeof r2 === "string") {
          res(r1.concat(r2));
        }

        if (Array.isArray(r1) && Array.isArray(r2)) {
          res(r1.concat(r2));
        }

        if (isPlainObject(r1) && isPlainObject(r2)) {
          res({ ...(r1 as Object), ...(r2 as Object) });
        }

        rej("Unsupported data types");
      })
      .catch(rej);
  });

  return promise;
}
```

```ts
// Approach 2

const isPlainObject = (value: unknown): boolean => {
  // For null and undefined.
  if (value == null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
};

export default function promiseMerge(
  p1: Promise<unknown>,
  p2: Promise<unknown>,
): Promise<unknown> {
 // Await both inputs directly, then reuse the same type-based merge rules.
  const result1 = await p1;
  const result2 = await p2;

  try {
    if (typeof result1 === 'number' && typeof result2 === 'number') {
      return result1 + result2;
    }

    if (typeof result1 === 'string' && typeof result2 === 'string') {
      return result1 + result2;
    }

    if (Array.isArray(result1) && Array.isArray(result2)) {
      return [...result1, ...result2];
    }

    if (isPlainObject(result1) && isPlainObject(result2)) {
      return { ...result1, ...result2 };
    }

    throw 'Unsupported data types';
  } catch {
    throw 'Unsupported data types';
  }
}
```

```ts
// Approach 3

const isPlainObject = (value: unknown): boolean => {
  // For null and undefined.
  if (value == null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
};

export default function promiseMerge(
  p1: Promise<unknown>,
  p2: Promise<unknown>,
): Promise<unknown> {
 return Promise.all([p1, p2]).then(([result1, result2]) => {
    try {
      if (typeof result1 === 'number' && typeof result2 === 'number') {
        return result1 + result2;
      }

      if (typeof result1 === 'string' && typeof result2 === 'string') {
        return result1 + result2;
      }

      if (Array.isArray(result1) && Array.isArray(result2)) {
        return [...result1, ...result2];
      }

      if (isPlainObject(result1) && isPlainObject(result2)) {
        return { ...result1, ...result2 };
      }

      throw 'Unsupported data types';
    } catch {
      throw 'Unsupported data types';
    }
  });
}
```
