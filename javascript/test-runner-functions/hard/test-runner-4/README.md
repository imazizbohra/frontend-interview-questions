# Test Runner IV

This is a follow-up to [Test Runner III](../../medium/test-runner-3/).

Keep the same `createTestRunner()` API, nested suites, per-spec hooks, and matcher behavior, but now allow specs and hooks to be async.

The main change is that `run()` should now return a `Promise` and await each setup hook, spec, and cleanup hook in sequence.

## API

`spec(name, fn)`

`fn` may now be synchronous or async.

`setupEach(fn)` and `cleanupEach(fn)`
Hooks may now be synchronous or async.

`run()`
Now returns `Promise<RunResult>`.

Execution should still be sequential in declaration order. For each spec:

- Await inherited setup hooks from outer to inner.
- Await the spec.
- Await inherited cleanup hooks from inner to outer.

If a spec or hook rejects:

- Use `error.message` when it rejects with an Error.
- Otherwise use `String(rejectedValue)`.

`suite(name, fn)`, `check(actual).toBe(expected)`, and `check(actual).not.toBe(expected)`
Keep the same behavior as part III. That includes chainable `.not`, where each extra `.not` flips the matcher again.

## Examples

```ts
const { spec, check, run } = createTestRunner();

spec('loads data', async () => {
  const value = await Promise.resolve(42);
  check(value).not.toBe(0);
});

await run();
// {
//   total: 1,
//   passed: 1,
//   failed: 0,
//   results: [{ name: 'loads data', status: 'passed' }],
// }
```

Async hooks should keep the same ordering as the synchronous version.

```ts
const log = [];
const { suite, spec, run, setupEach, cleanupEach } = createTestRunner();

suite('math', () => {
  setupEach(async () => {
    await Promise.resolve();
    log.push('setup math');
  });
  cleanupEach(async () => {
    await Promise.resolve();
    log.push('cleanup math');
  });

  suite('add', () => {
    setupEach(async () => {
      await Promise.resolve();
      log.push('setup add');
    });
    cleanupEach(async () => {
      await Promise.resolve();
      log.push('cleanup add');
    });

    spec('works', async () => {
      await Promise.resolve();
      log.push('spec');
    });
  });
});

await run();
log;
// [
//   'setup math',
//   'setup add',
//   'spec',
//   'cleanup add',
//   'cleanup math',
// ]
```

## Notes

- Do not run specs in parallel.
- Keep the same summary result shape as earlier questions; only `run()` becomes async.
- You do not need timeouts, retries, fake timers, skipped tests, or reporters.

## References

- Jest [jestjs.io](https://jestjs.io/)
- vitest [vitest.dev](https://vitest.dev/)

## Solutions

```ts
export type HookFn = () => void | Promise<void>;
export type SpecFn = () => void | Promise<void>;
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

  async function runSuite(
    suite: SuiteNode,
    suitePath: string[],
    inheritedSetups: HookFn[],
    inheritedCleanups: HookFn[],
    results: RunResult["results"],
  ): Promise<void> {
    // Setups run outer-to-inner, while cleanups need the reverse scope order.
    const setupHooks = [...inheritedSetups, ...suite.setupHooks];
    const cleanupHooks = [...suite.cleanupHooks, ...inheritedCleanups];

    // Depth-first traversal preserves declaration order for nested suites.
    for (const child of suite.children) {
      if (child.type === "suite") {
        await runSuite(
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
          await hook();
        }

        await child.fn();

        for (const hook of cleanupHooks) {
          await hook();
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
    async run(): Promise<RunResult> {
      const results: RunResult["results"] = [];

      await runSuite(rootSuite, [], [], [], results);

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
