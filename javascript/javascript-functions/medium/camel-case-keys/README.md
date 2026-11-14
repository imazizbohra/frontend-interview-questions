# Camel Case Keys

Implement a function `camelCaseKeys` that takes an object and returns a new object with all its keys converted to camel case.

Camel case is a format where word boundaries are represented by capitalizing each word after the first, and the first letter is lowercase. Some examples:

## Example(s)

```ts
camelCaseKeys({ foo_bar: true });
// { fooBar: true }

camelCaseKeys({ foo_bar: true, bar_baz: { baz_qux: '1' } });
// { fooBar: true, barBaz: { bazQux: '1' } }

camelCaseKeys([{ baz_qux: true }, { foo: true, bar: [{ foo_bar: 'hello' }] }]);
// [{ bazQux: true }, { foo: true, bar: [{ fooBar: 'hello' }] }]
```

## Reference(s)

- Github [camelcase-keys](https://github.com/sindresorhus/camelcase-keys)

## Solution(s)

```ts
// Approach 1

export default function camelCaseKeys(object: unknown): unknown {
  if (Array.isArray(object)) {
    return object.map((elem) => camelCaseKeys(elem));
  }

  if (typeof object === "object" && object !== null) {
    return Object.fromEntries(
      Object.entries(object).map(([key, value]) => {
        if (key.includes("_")) {
          const camelCaseKey = String(key)
            .split("_")
            .map((elem, index) => {
              if (index === 0) {
                return elem.toLowerCase();
              }

              return (
                elem.slice(0, 1).toUpperCase() + elem.slice(1).toLowerCase()
              );
            })
            .join("");

          return [camelCaseKey, camelCaseKeys(value)];
        }

        return [key, camelCaseKeys(value)];
      }),
    );
  }

  return object;
}
```

```ts
// Approach 2

function camelCase(str: string): string {
  return str
    .toLowerCase()
    .replace(/([_])([a-z])/g, (_match, _p1, p2) => p2.toUpperCase());
}

export default function camelCaseKeys(object: unknown): unknown {
  // Arrays keep their shape; only the nested values need recursive processing.
  if (Array.isArray(object)) {
    return object.map((item) => camelCaseKeys(item));
  }

  if (typeof object !== 'object' || object === null) {
    return object;
  }

  // Rebuild plain objects so both keys and nested values are transformed.
  return Object.fromEntries(
    Object.entries(object).map(([key, value]) => [
      camelCase(key),
      camelCaseKeys(value),
    ]),
  );
}
```
