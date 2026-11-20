# Test Runner II

This is a follow-up to [Test Runner](../test-runner/).

Keep the same `createTestRunner()` factory, `spec()`, `check()` matcher behavior, and synchronous `run()` summary shape, but now add nested suites.

To avoid clashing with common testing globals, this question continues to use `suite()` and `spec()` instead of `describe()` and `test()`.

This question should support:

- Flat specs, like before.
- Nested `suite(name, fn)` blocks.
- Full spec names joined with `' > '`.

## Example(s)

```ts
const { suite, spec, check, run } = createTestRunner();

suite('math', () => {
  suite('add', () => {
    spec('handles zero', () => {
      check(2 + 0).toBe(2);
    });
  });
});

run();
// {
//   total: 1,
//   passed: 1,
//   failed: 0,
//   results: [
//     { name: 'math > add > handles zero', status: 'passed' },
//   ],
// }
```

Flat specs should still work.

```ts
const { suite, spec, check, run } = createTestRunner();

spec('root spec', () => {
  check('runner').not.toBe('jest');
});

suite('math', () => {
  spec('adds', () => {});
});

run().results;
// [
//   { name: 'root spec', status: 'passed' },
//   { name: 'math > adds', status: 'passed' },
// ]
```

## Notes

- Use the same failure normalization as the previous question: `error.message` for `Error` objects, otherwise `String(thrownValue)`.
- Keep full result names deterministic.
- You do not need hooks, async specs, skipped tests, or module-level globals.

## Reference(s)

- Jest [jestjs.io](https://jestjs.io/)
- vitest [vitest.dev](https://vitest.dev/)

## Solution(s)

```ts
// Approach 1

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
  let names: string[] = [];

  function suite(name: string, fn: Runner) {
    names.push(name);
    fn();
    names.pop();
  }

  function spec(name: string, fn: Runner) {
    registry[[...names, name].join(" > ")] = fn;
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

  return { suite, spec, check, run };
}
```

```ts
// Approach 2

export type SpecFn = () => void;
type Matcher<T> = {
  toBe(expected: T): void;
  readonly not: Matcher<T>;
};

export type RunResult = {
  total: number;
  passed: number;
  failed: number;
  results: Array<
    | { name: string; status: 'passed' }
    | { name: string; status: 'failed'; error: string }
  >;
};

type SpecNode = {
  type: 'spec';
  name: string;
  fn: SpecFn;
};

type SuiteNode = {
  type: 'suite';
  name: string;
  children: Array<SuiteNode | SpecNode>;
};

function normalizeError(error: unknown): string {
  return error instanceof Error ? error.message : String(error);
}

export default function createTestRunner() {
  const rootSuite: SuiteNode = {
    type: 'suite',
    name: '',
    children: [],
  };
  const suiteStack: SuiteNode[] = [rootSuite];

  function getCurrentSuite(): SuiteNode {
    return suiteStack[suiteStack.length - 1];
  }

  function runSuite(
    suite: SuiteNode,
    suitePath: string[],
    results: RunResult['results'],
  ): void {
    // Depth-first traversal preserves declaration order for nested suites.
    for (const child of suite.children) {
      if (child.type === 'suite') {
        runSuite(child, [...suitePath, child.name], results);
        continue;
      }

      const name = [...suitePath, child.name].join(' > ');

      try {
        child.fn();
        results.push({ name, status: 'passed' });
      } catch (error) {
        results.push({
          name,
          status: 'failed',
          error: normalizeError(error),
        });
      }
    }
  }

  return {
    suite(name: string, fn: () => void): void {
      const suite: SuiteNode = {
        type: 'suite',
        name,
        children: [],
      };

      getCurrentSuite().children.push(suite);
      suiteStack.push(suite);

      try {
        fn();
      } finally {
        suiteStack.pop();
      }
    },
    spec(name: string, fn: SpecFn): void {
      getCurrentSuite().children.push({
        type: 'spec',
        name,
        fn,
      });
    },
    check<T>(actual: T) {
      function createMatcher(isNot: boolean): Matcher<T> {
        return {
          toBe(expected: T): void {
            const matches = Object.is(actual, expected);

            if (isNot ? matches : !matches) {
              throw new Error(
                `Expected ${String(actual)} ${isNot ? 'not ' : ''}to be ${String(expected)}`,
              );
            }
          },
          get not(): Matcher<T> {
            return createMatcher(!isNot);
          },
        };
      }

      return createMatcher(false);
    },
    run(): RunResult {
      const results: RunResult['results'] = [];

      runSuite(rootSuite, [], results);

      const failed = results.filter(
        (result) => result.status === 'failed',
      ).length;

      return {
        total: results.length,
        passed: results.length - failed,
        failed,
        results,
      };
    },
  };
}
```
