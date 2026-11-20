# Deep Map

Implement a function `deepMap(value, fn)` that returns a new value containing the results of calling a provided function on every value that is not an `Array` or plain `object`, including values within nested arrays and plain objects. The function `fn` is called with a single argument: the value being mapped or transformed. For every callback invocation, `this` should be the original `root` value passed to `deepMap`.

## Example(s)

```ts
const double = (x) => x * 2;

deepMap(2, double); // 4
deepMap([1, 2, 3], double); // [2, 4, 6]
deepMap({ a: 1, b: 2, c: 3 }, double); // { a: 2, b: 4, c: 6 }
deepMap(
  {
    foo: 1,
    bar: [2, 3, 4],
    qux: { a: 5, b: 6 },
  },
  double,
); // => { foo: 2, bar: [4, 6, 8], qux: { a: 10, b: 12 } }
```

## Reference(s)

- Concept [recursion](https://developer.mozilla.org/en-US/docs/Glossary/Recursion)
- Lodash [_.map](https://lodash.com/docs/4.18.1#map)

## Solution(s)

```ts
const isPlainObject = (value: unknown): boolean => {
  if (value === null || typeof value !== "object") {
    return false;
  }

  const proto = Object.getPrototypeOf(value);

  return proto === Object.prototype || proto === null;
};

export default function deepMap(value: unknown, fn: Function): unknown {
  const context = value;

  function map(value: unknown, fn: Function): unknown {
    if (Array.isArray(value)) {
      return value.map((elem) => map(elem, fn));
    }

    if (isPlainObject(value)) {
      return Object.fromEntries(
        Object.entries(value as Object).map(([k, v]) => {
          return [k, map(v, fn)];
        }),
      );
    }

    return fn.call(context, value);
  }

  return map(value, fn);
}
```
