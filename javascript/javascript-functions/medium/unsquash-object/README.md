# Unsquash Object

Implement a function that returns a new object after unsquashing a flat object where keys use period delimiters `(.)` to represent nesting.

This is the reverse operation of the [Squash Object](../squash-object/) question.

## Example(s)

```ts
const object = {
  a: 5,
  b: 6,
  'c.f': 9,
  'c.g.m': 17,
  'c.g.n': 3,
};

unsquashObject(object);
// {
//   a: 5,
//   b: 6,
//   c: {
//     f: 9,
//     g: {
//       m: 17,
//       n: 3,
//     },
//   },
// }
```

Any keys with nullish values (null and undefined) are still included in the returned object.

```ts
const object = {
  'a.b': null,
  'a.c': undefined,
};

unsquashObject(object); // { a: { b: null, c: undefined } }
```

It should also work with properties that represent arrays:

```ts
const object = {
  'a.b.0': 1,
  'a.b.1': 2,
  'a.b.2': 3,
  'a.c.0': 'foo',
};

unsquashObject(object); // { a: { b: [1, 2, 3], c: ['foo'] } }
```

Empty path segments should be treated as if that layer does not exist.

A key containing only empty segments, such as `''` or `'...'`, should be ignored.

```ts
const object = {
  'foo..bar': 2,
};

unsquashObject(object); // { foo: { bar: 2 } }
```

## Notes

- An empty input object should return an empty object.
- Keys with no delimiters should stay at the top level.
- Empty path segments, such as `foo..bar`, should be ignored.
- Nullish values (`null` and `undefined`) should be preserved.
- Array-like paths (`a.0`, `a.1`) should produce arrays.
- Conflicting keys (for example both `a` and `a.b`) are ambiguous. In JavaScript object insertion order, later assignments can overwrite earlier structures.

## Reference(s)

- Glossary [Recursion](https://developer.mozilla.org/en-US/docs/Glossary/Recursion)

## Solution(s)

```ts
// Approach 1: Recursion

type pojo = Record<string, any>;

export default function unsquashObject(obj: pojo): pojo {
  function unsquash(path: string, value: any, acc: pojo) {
    if (path.length === 0) {
      return value;
    }

    const pathArr = path.split(".").filter(Boolean);
    const key = pathArr[0];
    const restPathStr = pathArr.slice(1).join(".");

    if (acc && key in acc) {
      // reuse existing container
      unsquash(restPathStr, value, acc[key]);
      return acc;
    } else {
      // reuse existing container
      const newObj = acc ?? (Number.isNaN(Number(key)) ? {} : []);
      newObj[key] = unsquash(restPathStr, value, newObj[key]);
      return newObj;
    }
  }

  let returnObj: pojo = {};

  for (let path in obj) {
    // ignore empty path
    const pathStr = path.split(".").filter(Boolean).join("");
    if (pathStr.length === 0) {
      continue;
    }

    const value = obj[path];
    if (path.includes(".")) {
      const unsquashedObject = unsquash(path, value, returnObj);
      returnObj = { ...returnObj, ...unsquashedObject };
      continue;
    }

    returnObj[path] = value;
  }

  return returnObj;
}
```

```ts
// Approach 2: Iterative

function isArrayIndex(segment: string): boolean {
  return /^\d+$/.test(segment);
}

export default function unsquashObject(obj: Object): Object {
  const output: Record<string, any> = {};

  for (const [rawKey, value] of Object.entries(obj)) {
    const path = rawKey.split('.').filter(Boolean);

    if (path.length === 0) {
      continue;
    }

    let current: any = output;

    for (let i = 0; i < path.length - 1; i += 1) {
      const segment = path[i];
      const nextSegment = path[i + 1];

      // The upcoming path segment indicates whether this missing container
      // should behave like an object branch or an array slot holder.
      if (current[segment] == null || typeof current[segment] !== 'object') {
        current[segment] = isArrayIndex(nextSegment) ? [] : {};
      }

      current = current[segment];
    }

    current[path[path.length - 1]] = value;
  }

  return output;
}
```
