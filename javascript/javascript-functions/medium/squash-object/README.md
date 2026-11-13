# Squash Object

Implement a function that returns a new object with the input object squashed to a single level, where nested keys are joined with a period delimiter `(.)`.

## Example(s)

```ts
const object = {
  a: 5,
  b: 6,
  c: {
    f: 9,
    g: {
      m: 17,
      n: 3,
    },
  },
};

squashObject(object); // { a: 5, b: 6, 'c.f': 9, 'c.g.m': 17, 'c.g.n': 3 }
```

Any keys with nullish values (null and undefined) are still included in the returned object.

```ts
const object = {
  a: { b: null, c: undefined },
};
squashObject(object); // { 'a.b': null, 'a.c': undefined }
```

It should also work with properties that have arrays as the value:

```ts
const object = { a: { b: [1, 2, 3], c: ['foo'] } };
squashObject(object); // { 'a.b.0': 1, 'a.b.1': 2, 'a.b.2': 3, 'a.c.0': 'foo' }
```

Empty keys should be treated as if that "layer" does not exist.

```ts
const object = {
  foo: {
    '': { '': 1, bar: 2 },
  },
};
squashObject(object); // { foo: 1, 'foo.bar': 2 }
```

## Reference(s)

- Glossary [Recursion](https://developer.mozilla.org/en-US/docs/Glossary/Recursion)

## Techniques

- Convert array to Object `{ ...array }`

## Solution(s)

```ts
// Approach 1: Recursion

export default function squashObject(obj: Object): Object {
  const buildKey = (prefix: string, key: string) => {
    if (!prefix) {
      return key;
    }

    if (!key) {
      return prefix;
    }

    return `${prefix}.${key}`;
  };

  function squash(obj: Object, prefix: string) {
    const copyObj: Record<string, any> = { ...obj };
    const returnObj: Record<string, any> = {};

    for (let key in copyObj) {
      const value = copyObj[key];

      if (value !== null && typeof value === "object") {
        const squashedObject = squash(value, key);

        for (let squashedKey in squashedObject) {
          const squashedValue = squashedObject[squashedKey];

          returnObj[buildKey(prefix, squashedKey)] = squashedValue;
        }
      } else {
        returnObj[buildKey(prefix, key)] = value;
      }
    }

    return returnObj;
  }

  return squash(obj, "");
}
```

```ts
// Approach 2: Recursion + Collect Path

export default function squashObject(obj: Object): Object {
  function squashImpl(
    obj_: Object,
    path: Array<string>,
    output: Record<string, any>,
  ) {
    for (const [key, value] of Object.entries(obj_)) {
      if (typeof value !== 'object' || value === null) {
        // Build the dotted key from the recursion path; empty segments are skipped.
        output[path.concat(key).filter(Boolean).join('.')] = value;
      } else {
        squashImpl(value, path.concat(key), output);
      }
    }
  }

  const out = {};
  squashImpl(obj, [], out);
  return out;
}
```
