# Test Runner

Test runners like `Jest` and `Vitest` let code register tests, execute them, and summarize the results.

In this question, implement a small runner factory called `createTestRunner()`.

To avoid clashing with common testing globals such as `test()` and `expect()`, this question uses `spec()` and `check()` instead.

This first question is intentionally limited:

- Only flat specs are supported.
- Specs are synchronous.
- The only matcher is `toBe()` with chainable `.not`.

## Example(s)

```ts
const { spec, check, run } = createTestRunner();

spec('adds numbers', () => {
  check(1 + 2).toBe(3);
});

spec('compares strings', () => {
  check('runner').not.toBe('jest');
});

run();
// {
//   total: 2,
//   passed: 2,
//   failed: 0,
//   results: [
//     { name: 'adds numbers', status: 'passed' },
//     { name: 'compares strings', status: 'passed' },
//   ],
// }
```

Failing specs should be recorded instead of stopping the whole run.

```ts
const { spec, check, run } = createTestRunner();

spec('passes', () => {
  check(2 + 2).toBe(4);
});

spec('fails', () => {
  check('Jest').toBe('Vitest');
});

run();
// {
//   total: 2,
//   passed: 1,
//   failed: 1,
//   results: [
//     { name: 'passes', status: 'passed' },
//     {
//       name: 'fails',
//       status: 'failed',
//       error: 'Expected Jest to be Vitest',
//     },
//   ],
// }
```

## Notes

- Keep runner state local to createTestRunner(). Do not use shared module-level state.
- Keep the only matcher as toBe() with chainable .not.
- You do not need nested suites, hooks, async specs, skipped tests, snapshots, mocks, spies, or console output.

## Reference(s)

- Jest [jestjs.io](https://jestjs.io/)
- vitest [vitest.dev](https://vitest.dev/)

## Solution(s)

```ts
export type RunResult = {
  total: number;
  passed: number;
  failed: number;
  results: Array<
    | { name: string; status: "passed" }
    | { name: string; status: "failed"; error: string }
  >;
};

type Runner = () => void;

export default function createTestRunner() {
  const registry: Record<string, Runner> = {};

  function spec(name: string, fn: Runner) {
    registry[name] = fn;
  }

  function check(actual: unknown) {
    let flag = true;

    const obj = {
      toBe(expected: unknown) {
        if (flag) {
          if (!Object.is(actual, expected)) {
            throw new Error(
              `Expected ${String(actual)} to be ${String(expected)}`,
            );
          }
        } else {
          if (Object.is(actual, expected)) {
            throw new Error(
              `Expected ${String(actual)} not to be ${String(expected)}`,
            );
          }
        }
      },
      get not() {
        flag = !flag;
        return this;
      },
    };

    return obj;
  }

  function run() {
    const results: any[] = [];
    const total: number = Object.entries(registry).length;
    let passed: number = 0;
    let failed: number = 0;

    Object.entries(registry).forEach(([name, fn]) => {
      try {
        fn();
        passed++;
        results.push({ name: name, status: "passed" });
      } catch (error: unknown) {
        failed++;
        results.push({
          name: name,
          status: "failed",
          error: error instanceof Error ? error.message : error,
        });
      }
    });

    return {
      total,
      passed,
      failed,
      results,
    };
  }

  return { spec, check, run };
}
```
