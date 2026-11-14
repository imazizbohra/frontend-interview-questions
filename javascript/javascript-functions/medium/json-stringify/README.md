# JSON.stringify

Implement a `jsonStringify` function that converts a JavaScript value into a JSON string, similar to `JSON.stringify`.

Only JSON-serializable values (i.e. `boolean`, `number`, `null`, `array`, `object`) will be present in the input value.
Ignore the second and third optional parameters in the original API.

## Example(s)

```ts
jsonStringify({ foo: 'bar' }); // '{"foo":"bar"}'
jsonStringify({ foo: 'bar', bar: [1, 2, 3] }); // '{"foo":"bar","bar":[1,2,3]}'
jsonStringify({ foo: true, bar: false }); // '{"foo":true,"bar":false}'
```

Other types:

```ts
jsonStringify(null); // 'null'
jsonStringify(true); // 'true'
jsonStringify(false); // 'false'
jsonStringify(1); // '1'
jsonStringify('foo'); // '"foo"'
```

## Reference(s)

- JSON [stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

## Solution(s)

```ts
export default function jsonStringify(value: unknown): string {
  if (
    value == null ||
    typeof value === "boolean" ||
    typeof value === "number"
  ) {
    return String(value);
  }

  if (typeof value === "string") {
    return `"${value}"`;
  }

  if (Array.isArray(value)) {
    const arr = value.map((elem) => jsonStringify(elem));

    return "".concat("[").concat(arr.join(",")).concat("]");
  }

  if (value !== null && typeof value === "object") {
    const arr = Object.entries(value).map(([key, value]) => {
      return `"${key}":${jsonStringify(value)}`;
    });

    return "".concat("{").concat(arr.join(",")).concat("}");
  }
}
```
