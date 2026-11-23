# Schema Validator II

This is a follow-up to [Schema Validator](../schema-validator/).

Keep the same `v` API and `safeParse(value)` result shape, but now add chainable primitive rules:

- `v.string().min(n).max(n)`
- `v.number().min(n).max(n)`

Object validation stays the same as in the previous question:

- `v.object(shape)` only needs to support flat required fields.
- Unknown object keys are still allowed and preserved.

## API

`v.string()`
Creates a schema that accepts string values. Non-string values should return `Expected string`.

`v.number()`
Creates a schema that accepts number values. Non-number values should return `Expected number`.

`v.boolean()`
Creates a schema that accepts boolean values. Non-boolean values should return `Expected boolean`.

`v.object(shape)`
Creates a schema that validates an object against `shape`.

`stringSchema.min(length)`
Requires the string length to be at least `length`. If the value is too short, return `Expected at least ${length} characters`.

`stringSchema.max(length)`
Requires the string length to be at most `length`. If the value is too long, return `Expected at most ${length} characters`.

`numberSchema.min(value)`
Requires the number to be greater than or equal to `value`. If the value is too small, return `Expected number >= ${value}`.

`numberSchema.max(value)`
Requires the number to be less than or equal to `value`. If the value is too large, return `Expected number <= ${value}`.

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
const Name = v.string().min(2).max(5);

Name.safeParse('Ada');
// { success: true, data: 'Ada' }

Name.safeParse('');
// {
//   success: false,
//   errors: [{ path: [], message: 'Expected at least 2 characters' }],
// }
```

Schema builder methods must be immutable.

```ts
const BaseName = v.string();
const ShortName = BaseName.min(2);

BaseName.safeParse('');
// { success: true, data: '' }

ShortName.safeParse('');
// {
//   success: false,
//   errors: [{ path: [], message: 'Expected at least 2 characters' }],
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

export interface StringSchema extends Schema {
  min(length: number): StringSchema;
  max(length: number): StringSchema;
}

export interface NumberSchema extends Schema {
  min(value: number): NumberSchema;
  max(value: number): NumberSchema;
}

export interface SchemaFactory {
  string(): StringSchema;
  number(): NumberSchema;
  boolean(): Schema;
  object(shape: Record<string, Schema>): Schema;
}

const v: SchemaFactory = {
  string() {
    function createSchema(
      rules: Record<string, number>,
      min?: number,
      max?: number,
    ): StringSchema {
      return {
        min(min: number) {
          return createSchema({ ...rules, min }, min, max);
        },
        max(max: number) {
          return createSchema({ ...rules, max }, min, max);
        },
        safeParse(value: unknown) {
          if (!(typeof value === "string")) {
            return {
              success: false,
              errors: [{ path: [], message: "Expected string" }],
            };
          }

          const errors: ValidationError[] = [];

          for (const key in rules) {
            if (key === "min" && value.length < rules[key]) {
              errors.push({
                path: [],
                message: `Expected at least ${min} characters`,
              });
            }

            if (key === "max" && value.length > rules[key]) {
              errors.push({
                path: [],
                message: `Expected at most ${max} characters`,
              });
            }
          }

          if (errors.length > 0) {
            return {
              success: false,
              errors,
            };
          }

          return {
            success: true,
            data: value,
          };
        },
      };
    }

    return createSchema({});
  },
  number() {
    function createSchema(
      rules: Record<string, number>,
      min?: number,
      max?: number,
    ): StringSchema {
      return {
        min(min: number) {
          return createSchema({ ...rules, min }, min, max);
        },
        max(max: number) {
          return createSchema({ ...rules, max }, min, max);
        },
        safeParse(value: unknown) {
          if (!(typeof value === "number")) {
            return {
              success: false,
              errors: [{ path: [], message: "Expected number" }],
            };
          }

          const errors: ValidationError[] = [];

          for (const key in rules) {
            if (key === "min" && value < rules[key]) {
              errors.push({ path: [], message: `Expected number >= ${min}` });
            }

            if (key === "max" && value > rules[key]) {
              errors.push({ path: [], message: `Expected number <= ${max}` });
            }
          }

          if (errors.length > 0) {
            return {
              success: false,
              errors,
            };
          }

          return {
            success: true,
            data: value,
          };
        },
      };
    }

    return createSchema({});
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
        }

        return {
          success: true,
          data,
        };
      },
    };
  },
};

export default v;
```
