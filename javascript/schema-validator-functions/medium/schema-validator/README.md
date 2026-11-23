# Schema Validator

Libraries like [Zod](https://zod.dev/) and [Joi](https://joi.dev/) let application code describe data shapes once and reuse them to validate values.

In this question, implement a small validator builder exposed through `v`. Each schema should expose `safeParse(value)`.

This first question is intentionally limited:

- Support `v.string()`, `v.number()`, `v.boolean()`, and `v.object(shape)`.
- `v.object(shape)` only supports a flat structure, and all fields are required.
- Unknown object keys are allowed and preserved.

## API

`v.string()`
Creates a schema that accepts string values. Non-string values should return `Expected string`.

`v.number()`
Creates a schema that accepts number values. Non-number values should return `Expected number`.

`v.boolean()`
Creates a schema that accepts boolean values. Non-boolean values should return `Expected boolean`.

`v.object(shape)`
Creates a schema that validates an object against `shape`.

`shape` is an object whose values are schemas created by `v`.

Non-object values such as `null` or `arrays` should return `Expected object`.

Fields in `shape` are required in this question. Missing or `undefined` fields should return `Required`.

`schema.safeParse(value)`

Returns one of the following:

```json
{ success: true, data: value }
```

or

```json
{
  success: false,
  errors: [
    { path: Array<string | number>, message: string },
  ],
}
```

## Examples

```ts
const User = v.object({
  name: v.string(),
  age: v.number(),
  admin: v.boolean(),
});

User.safeParse({
  name: 'Alice',
  age: 30,
  admin: false,
  team: 'Core',
});
// {
//   success: true,
//   data: {
//     name: 'Alice',
//     age: 30,
//     admin: false,
//     team: 'Core',
//   },
// }
```

Validation failures should return all top-level field errors in schema declaration order.

```ts
const User = v.object({
  name: v.string(),
  age: v.number(),
  admin: v.boolean(),
});

User.safeParse({
  name: 123,
  admin: 'yes',
});
// {
//   success: false,
//   errors: [
//     { path: ['name'], message: 'Expected string' },
//     { path: ['age'], message: 'Required' },
//     { path: ['admin'], message: 'Expected boolean' },
//   ],
// }
```

## Notes

- Unknown object keys are allowed and should be preserved in `data`.
- Object fields are required in this question. Missing or `undefined` fields should return `Required`.
- `safeParse()` must not mutate the input value.
- You do not need arrays, nested objects, optional fields, coercion, transforms, async validation, or custom error messages.

## References

- [zod.dev](https://zod.dev/)
- [joi.dev](https://joi.dev/)

## Solutions

```ts
export type PathSegment = string | number;

export type ValidationError = {
  path: PathSegment[];
  message: string;
};

export type ParseResult =
  | {
      success: true;
      data: unknown;
    }
  | {
      success: false;
      errors: ValidationError[];
    };

export interface Schema {
  safeParse(value: unknown): ParseResult;
}

export interface SchemaFactory {
  string(): Schema;
  number(): Schema;
  boolean(): Schema;
  object(shape: Record<string, Schema>): Schema;
}

const v: SchemaFactory = {
  string() {
    return {
      safeParse(value: unknown) {
        if (typeof value === "string") {
          return {
            success: true,
            data: value,
          };
        }

        return {
          success: false,
          errors: [{ path: [], message: "Expected string" }],
        };
      },
    };
  },
  number() {
    return {
      safeParse(value: unknown) {
        if (typeof value === "number") {
          return {
            success: true,
            data: value,
          };
        }

        return {
          success: false,
          errors: [{ path: [], message: "Expected number" }],
        };
      },
    };
  },
  boolean() {
    return {
      safeParse(value: unknown) {
        if (typeof value === "boolean") {
          return {
            success: true,
            data: value,
          };
        }

        return {
          success: false,
          errors: [{ path: [], message: "Expected boolean" }],
        };
      },
    };
  },
  object(shape) {
    return {
      safeParse(value: unknown) {
        if (
          typeof value !== "object" ||
          value == null ||
          Array.isArray(value)
        ) {
          return {
            success: false,
            errors: [{ path: [], message: "Expected object" }],
          };
        }

        const data: Record<string, unknown> = { ...value };
        const errors: ValidationError[] = [];

        for (const key in shape) {
          if (!(key in data) || data[key] == null) {
            errors.push({
              path: [key],
              message: "Required",
            });
            continue;
          }

          const schema = shape[key];
          const result = schema.safeParse(data[key]);

          if (!result.success) {
            errors.push({
              path: [key],
              message: result.errors[0].message,
            });
          }
        }

        if (errors.length > 0) {
          return {
            success: false,
            errors,
          };
        } else {
          return {
            success: true,
            data,
          };
        }
      },
    };
  },
};

export default v;
```
