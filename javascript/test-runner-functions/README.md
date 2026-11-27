# Test Runner

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

## References

- Jest [jestjs.io](https://jestjs.io/)
- vitest [vitest.dev](https://vitest.dev/)
