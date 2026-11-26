# Superjson II

This is a follow-up to [Superjson](../../medium/superjson/).

`JSON.stringify()` works well for plain JSON data, but it loses information for values like `undefined`, `Date`, and `RegExp`.

Popular libraries like [SuperJSON](https://github.com/ravionhq/superjson) and [devalue](https://github.com/sveltejs/devalue) were built to solve exactly this kind of problem.

Implement the same two functions:

- `serialize(value)`: returns a string.
- `deserialize(serialized)`: returns the original value represented by that string.

This question extends the behavior implemented in Superjson and adds support for:

- Map
- Set

Collection values can contain any previously supported value, as well as nested `Map` and `Set`. `Map` keys should also round-trip correctly, including object keys. Insertion order should be preserved for both collections.

As before, the exact serialized format is up to you.

## Examples

```ts
const value = new Map([
  ['createdAt', new Date('2026-01-01T00:00:00.000Z')],
  ['flags', new Set([undefined, /user-\d+/gi])],
]);

const serialized = serialize(value);
const restored = deserialize(serialized);

restored instanceof Map; // true
restored.get('createdAt') instanceof Date; // true
restored.get('flags') instanceof Set; // true
```

```ts
const key = { id: 1 };
const value = new Map([[key, { retries: Infinity }]]);
const restored = deserialize(serialize(value));

const [[restoredKey, restoredValue]] = restored.entries();
restoredKey; // { id: 1 }
restoredValue.retries; // Infinity
```

## Notes

- You may assume `deserialize` only receives strings produced by `serialize`.
- `serialize` should still throw a `TypeError` for functions, symbols, symbol keys, cyclic references, sparse arrays, and custom class instances.
- Malformed serialized strings are still out of scope.

## References

- [SuperJSON](https://github.com/ravionhq/superjson)
- [devalue](https://github.com/sveltejs/devalue)

## Solutions

```ts
type Encoded =
  | null
  | boolean
  | number
  | string
  | Encoded[]
  | { [key: string]: Encoded };

type EncodedObject = { [key: string]: Encoded };

const TAG_KEY = "$superjson";

function isPlainObject(value: unknown): value is Record<string, unknown> {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === Object.prototype || prototype === null;
}

function makeTag(type: string, extra: EncodedObject = {}): EncodedObject {
  return {
    [TAG_KEY]: type,
    ...extra,
  };
}

function encode(value: unknown, stack = new Set<object>()): Encoded {
  if (
    value === null ||
    typeof value === "boolean" ||
    typeof value === "string"
  ) {
    return value;
  }

  if (typeof value === "number") {
    if (Number.isNaN(value)) {
      return makeTag("NaN");
    }

    if (value === Infinity) {
      return makeTag("Infinity");
    }

    if (value === -Infinity) {
      return makeTag("-Infinity");
    }

    return value;
  }

  if (value === undefined) {
    return makeTag("undefined");
  }

  if (typeof value === "bigint") {
    return makeTag("BigInt", { value: String(value) });
  }

  if (typeof value === "function" || typeof value === "symbol") {
    throw new TypeError("Unsupported value type");
  }

  if (value instanceof Date) {
    return makeTag("Date", { value: value.toISOString() });
  }

  if (value instanceof RegExp) {
    return makeTag("RegExp", {
      source: value.source,
      flags: value.flags,
    });
  }

  if (typeof value !== "object") {
    throw new TypeError("Unsupported value type");
  }

  // Track the active recursion path so cycles throw without rejecting separate branches.
  if (stack.has(value)) {
    throw new TypeError("Circular references are not supported");
  }

  stack.add(value);

  try {
    if (value instanceof Map) {
      return makeTag("Map", {
        entries: Array.from(value).map(([_key, _value]) => {
          return [encode(_key, stack), encode(_value, stack)];
        }),
      });
    }

    if (value instanceof Set) {
      return makeTag("Set", {
        values: Array.from(value).map((_value) => {
          return encode(_value, stack);
        }),
      });
    }

    if (Array.isArray(value)) {
      return value.map((item, index) => {
        if (!(index in value)) {
          throw new TypeError("Sparse arrays are not supported");
        }

        return encode(item, stack);
      });
    }

    if (!isPlainObject(value)) {
      throw new TypeError("Only plain objects are supported");
    }

    if (Object.getOwnPropertySymbols(value).length > 0) {
      throw new TypeError("Symbol keys are not supported");
    }

    const encodedObject: EncodedObject = {};

    for (const [key, entry] of Object.entries(value)) {
      encodedObject[key] = encode(entry, stack);
    }

    return encodedObject;
  } finally {
    // Always pop this object off the path, even if a nested encode throws.
    stack.delete(value);
  }
}

function decode(value: Encoded): unknown {
  if (
    value === null ||
    typeof value === "boolean" ||
    typeof value === "number" ||
    typeof value === "string"
  ) {
    return value;
  }

  if (Array.isArray(value)) {
    return value.map((item) => decode(item));
  }

  // Tagged objects are the escape hatch for values JSON cannot represent directly.
  const tag = value[TAG_KEY];

  if (typeof tag === "string") {
    switch (tag) {
      case "undefined":
        return undefined;
      case "NaN":
        return NaN;
      case "Infinity":
        return Infinity;
      case "-Infinity":
        return -Infinity;
      case "BigInt":
        return BigInt(String(value.value));
      case "Date":
        return new Date(String(value.value));
      case "RegExp":
        return new RegExp(String(value.source), String(value.flags));
      case "Map":
        return new Map(
          (value.entries as Encoded[][]).map(([_key, _value]) => {
            return [decode(_key), decode(_value)];
          }),
        );
      case "Set":
        return new Set(
          (value.values as Encoded[]).map((_value) => decode(_value)),
        );
      default:
        throw new TypeError("Invalid serialized value");
    }
  }

  const decodedObject: Record<string, unknown> = {};

  for (const [key, entry] of Object.entries(value)) {
    decodedObject[key] = decode(entry);
  }

  return decodedObject;
}

export function serialize(value: unknown): string {
  return JSON.stringify(encode(value));
}

export function deserialize(serialized: string): unknown {
  return decode(JSON.parse(serialized) as Encoded);
}
```
