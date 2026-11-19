# Assert Lite

`Node.js` ships with an [assert](https://nodejs.org/api/assert.html) module for writing test expectations.

In this question, implement a tiny assertion library inspired by `node:assert/strict`.

This first question is intentionally limited:

- Export `AssertionError`, `ok(value, message?)`, and `strictEqual(actual, expected, message?)`.
- Passing assertions should return `undefined`.
- Failing assertions should throw `AssertionError`.
- `strictEqual()` should use `Object.is()` semantics.

## Example(s)

```ts
ok('hello');
strictEqual(3, 3);
// No error.
```

```ts
strictEqual(3, '3');
// Throws AssertionError {
//   name: 'AssertionError',
//   message: 'Expected values to be strictly equal',
//   actual: 3,
//   expected: '3',
//   operator: 'strictEqual',
// }
```

Custom messages should override the default failure message.

```ts
ok('', 'Name is required');
// Throws AssertionError {
//   message: 'Name is required',
//   operator: 'ok',
// }
```

## Notes

- Keep the API intentionally small and interview-scoped.
- You do not need diff formatting, loose equality assertions, negated assertions, or stack trace customization.
- Tests in this question only use ordinary interview values; you do not need to optimize for every edge case of the real Node.js module.

## Reference(s)

- NodeJs [assert](https://nodejs.org/api/assert.html)

## Solution(s)

```ts
type AssertionErrorOptions = {
  message?: string;
  actual?: unknown;
  expected?: unknown;
  operator?: string;
};

export class AssertionError extends Error {
  name: string;
  actual?: unknown;
  expected?: unknown;
  operator?: string;

  constructor(options: AssertionErrorOptions = {}) {
    const {
      actual,
      expected,
      message = "Assertion failed",
      operator,
    } = options;
    super(message);
    // before: error -> Error.prototype -> Object.prototype
    Object.setPrototypeOf(this, new.target.prototype); // fix for older version
    // after: error -> AssertionError.prototype -> Error.prototype -> Object.prototype

    this.name = "AssertionError";
    this.actual = actual;
    this.expected = expected;
    this.operator = operator;
  }
}

export function ok(value: unknown, message?: string): void {
  if (value) {
    return;
  }

  throw new AssertionError({
    actual: value,
    expected: true,
    message: message ?? "Expected value to be truthy",
    operator: "ok",
  });
}

export function strictEqual(
  actual: unknown,
  expected: unknown,
  message?: string,
): void {
  if (Object.is(actual, expected)) {
    return;
  }

  throw new AssertionError({
    actual: actual,
    expected: expected,
    message: message ?? "Expected values to be strictly equal",
    operator: "strictEqual",
  });
}
```
