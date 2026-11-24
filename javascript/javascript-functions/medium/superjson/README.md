# Superjson

`JSON.stringify()` works well for plain JSON data, but it loses information for values like `undefined`, `Date`, and `RegExp`.

Popular libraries like [SuperJSON](https://github.com/ravionhq/superjson) and [devalue](https://github.com/sveltejs/devalue) were built to solve exactly this kind of problem.

Implement two functions:

- `serialize(value)`: returns a string.
- `deserialize(serialized)`: returns the original value represented by that string.

The exact serialized format is up to you.

For this question, support:

- All normal JSON values
- `undefined`
- `NaN`, `Infinity`, `-Infinity`
- `BigInt`
- `Date`
- `RegExp`

Supported values can appear anywhere inside dense arrays and plain objects.

## Examples

```ts
const value = {
  createdAt: new Date('2026-01-01T00:00:00.000Z'),
  retries: Infinity,
  missing: undefined,
  matcher: /user-\d+/gi,
};

const serialized = serialize(value);
// Any string format is fine as long as deserialize(serialized)
// recreates an equivalent value.

const restored = deserialize(serialized);

restored.createdAt instanceof Date; // true
restored.retries; // Infinity
restored.missing; // undefined
restored.matcher.test('USER-42'); // true
```

```ts
const value = [undefined, BigInt(42), { score: NaN }];
const restored = deserialize(serialize(value));

restored[0]; // undefined
restored[1]; // 42n
Number.isNaN(restored[2].score); // true
```

## Notes

- You may assume `deserialize` only receives strings produced by `serialize`.
- `serialize` should throw a `TypeError` for functions, symbols, symbol keys, `Map`, `Set`, cyclic references, sparse arrays, and custom class instances.
- Malformed serialized strings are out of scope.

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

  if (value instanceof Map || value instanceof Set) {
    throw new TypeError("Map and Set are not supported in this version");
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
