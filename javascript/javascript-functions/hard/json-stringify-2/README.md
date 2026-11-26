# JSON.stringify II

Implement a `jsonStringify` function that converts a JavaScript value into a JSON string, similar to `JSON.stringify.`

- You may ignore the second and third optional parameters in the original API.
- The function should behave exactly like `JSON.stringify()` for any data type. Refer to the examples below.
- Other cases:
  - Cyclic references: `throw TypeError('Converting circular structure to JSON')`.
  - BigInt: `throw TypeError('Do not know how to serialize a BigInt')`.

## Behavior guide

The differences between top-level values, object properties, and array elements are easy to mix up. Use this table as the core behavior contract:

| Value kind | Top-level value | Object property value | Array element value |
| --- | --- | --- | --- |
| `undefined`, `Symbol`, or function | Return `undefined` | Omit the property | Serialize as `null` |
| `NaN`, `Infinity`, or `-Infinity` | Serialize as `null` | Serialize as `null` | Serialize as `null` |
| Date or custom `toJSON()` object | Serialize the `toJSON()` result | Serialize the `toJSON()` result | Serialize the `toJSON()` result |
| `BigInt` | Throw a `TypeError` | Throw a `TypeError` | Throw a `TypeError` |
| Cyclic object or array | Throw a `TypeError` | Throw a `TypeError` | Throw a `TypeError` |

## Examples

```ts
jsonStringify({ foo: 'bar' }); // '{"foo":"bar"}'
jsonStringify({ foo: 'bar', bar: [1, 2, 3] }); // '{"foo":"bar","bar":[1,2,3]}'
```

```ts
jsonStringify(); // undefined
jsonStringify(undefined); // undefined
jsonStringify(null); // 'null'
jsonStringify(true); // 'true'
jsonStringify(false); // 'false'
jsonStringify(1); // '1'
jsonStringify(Infinity); // 'null'
jsonStringify(NaN); // 'null'
jsonStringify('foo'); // '"foo"'
jsonStringify('"foo"') === '"\\"foo\\""'; // Double quotes present in the original input are escaped using backslashes
jsonStringify(Symbol('foo')); // undefined
jsonStringify(() => {}); // undefined
jsonStringify(['foo', 'bar']); // '["foo","bar"]'
jsonStringify(/foo/); // '{}'
jsonStringify(new Map()); // '{}'
jsonStringify(new Set()); // '{}'
```

## References

- JSON [stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

## Solutions

```ts
const ESCAPE_REGEX = /[\u0000-\u001f"\\]/g;
const ESCAPE_SEQUENCES: Record<string, string> = {
  '"': '\\"',
  "\\": "\\\\",
  "\b": "\\b",
  "\f": "\\f",
  "\n": "\\n",
  "\r": "\\r",
  "\t": "\\t",
};

function escapeString(value: string): string {
  return value.replace(ESCAPE_REGEX, (character) => {
    return (
      ESCAPE_SEQUENCES[character] ??
      `\\u${character.charCodeAt(0).toString(16).padStart(4, "0")}`
    );
  });
}

export default function jsonStringify(value: unknown): string {
  const cache = new Set<unknown>();

  function stringify(value: unknown): string | undefined {
    if (cache.has(value)) {
      throw new TypeError("Converting circular structure to JSON");
    }

    if (typeof value === "bigint") {
      throw new TypeError("Do not know how to serialize a BigInt");
    }

    if (Number.isNaN(value) || value === Infinity || value === -Infinity) {
      return String(null);
    }

    if (
      value === undefined ||
      typeof value === "symbol" ||
      typeof value === "function"
    ) {
      return undefined;
    }

    if (
      value === null ||
      typeof value === "boolean" ||
      typeof value === "number"
    ) {
      return String(value);
    }

    if (typeof value === "string") {
      return `"${escapeString(value)}"`;
    }

    if (
      value instanceof Date ||
      (typeof value === "object" && Object.hasOwn(value, "toJSON"))
    ) {
      return jsonStringify((value as any).toJSON());
    }

    if (value instanceof Number || value instanceof Boolean) {
      return value.toString();
    }

    if (value instanceof String) {
      return stringify(value.toString());
    }

    if (Array.isArray(value)) {
      cache.add(value);

      const arr = value.map((elem) => {
        return stringify(elem) ?? String(null);
      });
      return "".concat("[").concat(arr.join(",")).concat("]");
    }

    if (typeof value === "object" && value !== null) {
      cache.add(value);

      const arr = Object.entries(value).reduce((acc, [key, _value]) => {
        const serializedValue = stringify(_value);

        if (serializedValue === undefined) {
          return acc;
        }

        acc.push(`"${key}":${stringify(_value)}`);
        return acc;
      }, [] as string[]);

      return "".concat("{").concat(arr.join(",")).concat("}");
    }

    return "";
  }

  return stringify(value) as string;
}
```
