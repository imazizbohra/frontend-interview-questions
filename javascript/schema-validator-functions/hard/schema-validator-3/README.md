# Schema Validator III

This is a follow-up to [Schema Validator II](../../medium/schema-validator-2/).

Keep the same `v` API and `safeParse(value)` result shape, but now add recursive schema composition:

- Nested `v.object(shape)`
- `v.array(itemSchema)`
- `.optional()`

Validation errors should include full paths with string and number segments.

## API

### Primitive Validators

- `v.string()`

- Still accepts string values.
- Non-string values should return `Expected string`.

- `v.number()`

- Still accepts number values.
- Non-number values should return `Expected number`.

- `v.boolean()`

- Still accepts boolean values.
- Non-boolean values should return `Expected boolean`.

### Object Validator

- `v.object(shape)`

- Now supports recursive nesting.
- Non-object values, such as `null` or arrays, should return `Expected object`.
- Missing or `undefined` required object fields should return `Required` at the full nested path.

### String Validators

- `stringSchema.min(length)`

- Requires the string length to be at least `length`.
- If the value is too short, return: `Expected at least ${length} characters`

- `stringSchema.max(length)`

- Requires the string length to be at most `length`.
- If the value is too long, return: `Expected at most ${length} characters`

### Number Validators

- `numberSchema.min(value)`

- Requires the number to be greater than or equal to `value`.
- If the value is too small, return: `Expected number >= ${value}`

- `numberSchema.max(value)`

- Requires the number to be less than or equal to `value`.
- If the value is too large, return: `Expected number <= ${value}`

### Array Validator

- `v.array(itemSchema)`

- Creates a schema that validates arrays whose items must match `itemSchema`.
- Non-array values should return `Expected array`.

### Optional Validator

- `schema.optional()`

- Creates a new schema that also accepts `undefined`.
- When used on an object field, that field may be missing or `undefined`.
- If an optional value is present but invalid, it should return the same error message as the underlying schema.

## Returns

- `schema.safeParse(value)` still returns:

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
const UserList = v.object({
  users: v.array(
    v.object({
      name: v.string(),
      age: v.number().min(18),
    }),
  ),
});

UserList.safeParse({
  users: [
    { name: 'Alice', age: 30 },
    { name: 123, age: 16 },
  ],
});
// {
//   success: false,
//   errors: [
//     { path: ['users', 1, 'name'], message: 'Expected string' },
//     { path: ['users', 1, 'age'], message: 'Expected number >= 18' },
//   ],
// }
```

Optional fields may be omitted or set to undefined.

```ts
const User = v.object({
  name: v.string(),
  nickname: v.string().optional(),
  tags: v.array(v.string()).optional(),
});

User.safeParse({ name: 'Alice' });
// { success: true, data: { name: 'Alice' } }
```

Schema builder methods must still be immutable.

```ts
const Email = v.string();
const User = v.object({
  primaryEmail: Email,
  backupEmail: Email.optional(),
});

Email.safeParse(undefined);
// {
//   success: false,
//   errors: [{ path: [], message: 'Expected string' }],
// }

User.safeParse({ primaryEmail: 'alice@example.com' });
// {
//   success: true,
//   data: { primaryEmail: 'alice@example.com' },
// }
```

## Notes

- Errors should be collected in deterministic order: object keys in schema declaration order, array items in index order, and chained rule checks in call order.
- Unknown object keys are allowed and should be preserved in `data`.
- Schema instances are immutable. Calling methods such as `.min()`, `.max()`, or `.optional()` must return a new schema instance without changing the original schema. `safeParse()` must not mutate the input value.
- You do not need coercion, transforms, async validation, unions, custom refinements, unknown-key stripping, or custom error messages.

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
  optional(): Schema;
  _validate(
    value: unknown,
    path: PathSegment[],
    errors: ValidationError[],
  ): void;
  _isOptional: boolean;
}

export interface StringSchema extends Schema {
  min(length: number): StringSchema;
  max(length: number): StringSchema;
  optional(): StringSchema;
}

export interface NumberSchema extends Schema {
  min(value: number): NumberSchema;
  max(value: number): NumberSchema;
  optional(): NumberSchema;
}

export interface BooleanSchema extends Schema {
  optional(): BooleanSchema;
}

export interface ObjectSchema extends Schema {
  optional(): ObjectSchema;
}

export interface ArraySchema extends Schema {
  optional(): ArraySchema;
}

type StringRule =
  | {
      type: "min";
      value: number;
    }
  | {
      type: "max";
      value: number;
    };

type NumberRule =
  | {
      type: "min";
      value: number;
    }
  | {
      type: "max";
      value: number;
    };

function createBaseSchema(
  validate: (
    value: unknown,
    path: PathSegment[],
    errors: ValidationError[],
  ) => void,
  isOptional: boolean,
): Pick<Schema, "safeParse" | "_validate" | "_isOptional"> {
  return {
    _isOptional: isOptional,

    safeParse(value: unknown): ParseResult {
      // Optional schemas short-circuit before any type-specific validation runs.
      if (value === undefined && isOptional) {
        return {
          success: true,
          data: value,
        };
      }

      const errors: ValidationError[] = [];
      validate(value, [], errors);

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

    _validate(value, path, errors): void {
      // Nested validators use the same optional rule when parents recurse into them.
      if (value === undefined && isOptional) {
        return;
      }

      validate(value, path, errors);
    },
  };
}

function isPlainObject(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

function createStringSchema(
  rules: StringRule[] = [],
  isOptional = false,
): StringSchema {
  const validate = (
    value: unknown,
    path: PathSegment[],
    errors: ValidationError[],
  ): void => {
    if (typeof value !== "string") {
      errors.push({
        path,
        message: "Expected string",
      });
      return;
    }

    for (const rule of rules) {
      if (rule.type === "min" && value.length < rule.value) {
        errors.push({
          path,
          message: `Expected at least ${rule.value} characters`,
        });
      }

      if (rule.type === "max" && value.length > rule.value) {
        errors.push({
          path,
          message: `Expected at most ${rule.value} characters`,
        });
      }
    }
  };

  return Object.freeze({
    ...createBaseSchema(validate, isOptional),
    min(length: number): StringSchema {
      return createStringSchema(
        [...rules, { type: "min", value: length }],
        isOptional,
      );
    },
    max(length: number): StringSchema {
      return createStringSchema(
        [...rules, { type: "max", value: length }],
        isOptional,
      );
    },
    optional(): StringSchema {
      return createStringSchema([...rules], true);
    },
  });
}

function createNumberSchema(
  rules: NumberRule[] = [],
  isOptional = false,
): NumberSchema {
  const validate = (
    value: unknown,
    path: PathSegment[],
    errors: ValidationError[],
  ): void => {
    if (typeof value !== "number") {
      errors.push({
        path,
        message: "Expected number",
      });
      return;
    }

    for (const rule of rules) {
      if (rule.type === "min" && value < rule.value) {
        errors.push({
          path,
          message: `Expected number >= ${rule.value}`,
        });
      }

      if (rule.type === "max" && value > rule.value) {
        errors.push({
          path,
          message: `Expected number <= ${rule.value}`,
        });
      }
    }
  };

  return Object.freeze({
    ...createBaseSchema(validate, isOptional),
    min(minimum: number): NumberSchema {
      return createNumberSchema(
        [...rules, { type: "min", value: minimum }],
        isOptional,
      );
    },
    max(maximum: number): NumberSchema {
      return createNumberSchema(
        [...rules, { type: "max", value: maximum }],
        isOptional,
      );
    },
    optional(): NumberSchema {
      return createNumberSchema([...rules], true);
    },
  });
}

function createBooleanSchema(isOptional = false): BooleanSchema {
  return Object.freeze({
    ...createBaseSchema((value, path, errors) => {
      if (typeof value !== "boolean") {
        errors.push({
          path,
          message: "Expected boolean",
        });
      }
      return;
    }, isOptional),
    optional(): BooleanSchema {
      return createBooleanSchema(true);
    },
  });
}

function createObjectSchema(
  shape: Record<string, Schema>,
  isOptional = false,
): ObjectSchema {
  const shapeCopy = { ...shape };

  return Object.freeze({
    ...createBaseSchema((value, path, errors) => {
      if (!isPlainObject(value)) {
        errors.push({
          path,
          message: "Expected object",
        });
        return;
      }

      for (const key of Object.keys(shapeCopy)) {
        const fieldSchema = shapeCopy[key];

        // Missing keys are allowed only when the child schema was marked optional.
        if (!Object.hasOwn(value, key) || value[key] === undefined) {
          if (!fieldSchema._isOptional) {
            errors.push({
              path: [...path, key],
              message: "Required",
            });
          }

          continue;
        }

        fieldSchema._validate(value[key], [...path, key], errors);
      }
    }, isOptional),
    optional(): ObjectSchema {
      return createObjectSchema(shapeCopy, true);
    },
  });
}

function createArraySchema(
  itemSchema: Schema,
  isOptional = false,
): ArraySchema {
  return Object.freeze({
    ...createBaseSchema((value, path, errors) => {
      if (!Array.isArray(value)) {
        errors.push({
          path,
          message: "Expected array",
        });
        return;
      }

      for (let index = 0; index < value.length; index += 1) {
        itemSchema._validate(value[index], [...path, index], errors);
      }
    }, isOptional),
    optional(): ArraySchema {
      return createArraySchema(itemSchema, true);
    },
  });
}

const v = Object.freeze({
  string(): StringSchema {
    return createStringSchema();
  },

  number(): NumberSchema {
    return createNumberSchema();
  },

  boolean(): BooleanSchema {
    return createBooleanSchema();
  },

  object(shape: Record<string, Schema>): ObjectSchema {
    return createObjectSchema(shape);
  },

  array(itemSchema: Schema): ArraySchema {
    return createArraySchema(itemSchema);
  },
});

export default v;
```
