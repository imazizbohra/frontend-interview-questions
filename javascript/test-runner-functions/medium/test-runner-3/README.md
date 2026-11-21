# Test Runner III

This is a follow-up to [Test Runner II](../test-runner-2/).

Keep the same `createTestRunner()` API, nested suites, matcher behavior, synchronous specs, and `run()` result shape, but now add per-spec hooks:

- `setupEach(fn)`
- `cleanupEach(fn)`

Hooks are scoped to the current suite and inherited by descendant specs.

## Examples

```ts
const log = [];
const { suite, spec, check, run, setupEach, cleanupEach } = createTestRunner();

suite('math', () => {
  setupEach(() => {
    log.push('setup math');
  });
  cleanupEach(() => {
    log.push('cleanup math');
  });

  suite('add', () => {
    setupEach(() => {
      log.push('setup add');
    });
    cleanupEach(() => {
      log.push('cleanup add');
    });

    spec('works', () => {
      check('spec').not.toBe('other');
      log.push('spec');
    });
  });
});

run();
log;
// [
//   'setup math',
//   'setup add',
//   'spec',
//   'cleanup add',
//   'cleanup math',
// ]
```

Hooks should rerun for every matching spec.

```ts
const log = [];
const { suite, spec, run, setupEach } = createTestRunner();

suite('math', () => {
  setupEach(() => {
    log.push('setup');
  });

  spec('one', () => {
    log.push('one');
  });
  spec('two', () => {
    log.push('two');
  });
});

run();
log;
// ['setup', 'one', 'setup', 'two']
```

## Notes

- Keep hooks synchronous in this question.
- Hooks should apply to descendant specs only.
- You do not need beforeAll, afterAll, skipped tests, async hooks, or detailed hook failure semantics.

## References

- Jest [jestjs.io](https://jestjs.io/)
- vitest [vitest.dev](https://vitest.dev/)

## Solutions

```ts
export type HookFn = () => void;
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
    | { name: string; status: "passed" }
    | { name: string; status: "failed"; error: string }
  >;
};

type SpecNode = {
  type: "spec";
  name: string;
  fn: SpecFn;
};

type SuiteNode = {
  type: "suite";
  name: string;
  setupHooks: HookFn[];
  children: Array<SuiteNode | SpecNode>;
  cleanupHooks: HookFn[];
};

function normalizeError(error: unknown): string {
  return error instanceof Error ? error.message : String(error);
}

export default function createTestRunner() {
  const rootSuite: SuiteNode = {
    type: "suite",
    name: "",
    setupHooks: [],
    children: [],
    cleanupHooks: [],
  };
  const suiteStack: SuiteNode[] = [rootSuite];

  function getCurrentSuite(): SuiteNode {
    return suiteStack[suiteStack.length - 1];
  }

  function runSuite(
    suite: SuiteNode,
    suitePath: string[],
    inheritedSetups: HookFn[],
    inheritedCleanups: HookFn[],
    results: RunResult["results"],
  ): void {
    // Setups run outer-to-inner, while cleanups need the reverse scope order.
    const setupHooks = [...inheritedSetups, ...suite.setupHooks];
    const cleanupHooks = [...suite.cleanupHooks, ...inheritedCleanups];

    // Depth-first traversal preserves declaration order for nested suites.
    for (const child of suite.children) {
      if (child.type === "suite") {
        runSuite(
          child,
          [...suitePath, child.name],
          setupHooks,
          cleanupHooks,
          results,
        );
        continue;
      }

      const name = [...suitePath, child.name].join(" > ");

      try {
        for (const hook of setupHooks) {
          hook();
        }

        child.fn();

        for (const hook of cleanupHooks) {
          hook();
        }

        results.push({ name, status: "passed" });
      } catch (error) {
        results.push({
          name,
          status: "failed",
          error: normalizeError(error),
        });
      }
    }
  }

  return {
    suite(name: string, fn: () => void): void {
      const suite: SuiteNode = {
        type: "suite",
        name,
        setupHooks: [],
        children: [],
        cleanupHooks: [],
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
        type: "spec",
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
                `Expected ${String(actual)} ${isNot ? "not " : ""}to be ${String(expected)}`,
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
    setupEach(fn: HookFn) {
      getCurrentSuite().setupHooks.push(fn);
    },
    cleanupEach(fn: HookFn) {
      getCurrentSuite().cleanupHooks.push(fn);
    },
    run(): RunResult {
      const results: RunResult["results"] = [];

      runSuite(rootSuite, [], [], [], results);

      const failed = results.filter(
        (result) => result.status === "failed",
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
