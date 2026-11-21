# Middlewares

Implement a `middlewares` function that takes any number of middleware functions and composes them into a single callable function. This composed function accepts a `context`, returns a `Promise`, and calls each middleware in order.

Each middleware is a function that receives two arguments:

- context: An object shared across all middlewares.
- next: A function that invokes the next middleware in the chain.

When `next()` is called, the next middleware should run. If a middleware does not call `next()`, the chain stops.

The execution should be **asynchronous** and **sequential**, similar to how middleware works in frameworks like `Koa`.

The composed function can be called multiple times, including concurrently, and each invocation should traverse the middleware chain independently. If `context` is omitted, use an empty object.

## Examples

```ts
async function fn1(ctx, next) {
  ctx.stack.push('fn1-start');
  await next();
  ctx.stack.push('fn1-end');
}

async function fn2(ctx, next) {
  ctx.stack.push('fn2-start');
  await new Promise((resolve) => setTimeout(resolve, 1000));
  await next();
  ctx.stack.push('fn2-end');
}

function fn3(ctx, next) {
  ctx.stack.push('fn3-start');
  next();
  ctx.stack.push('fn3-end');
}

const composedFn = middlewares(fn1, fn2, fn3);

const context = { stack: [] };
await composedFn(context);

console.log(context.stack);
// ['fn1-start', 'fn2-start', 'fn3-start', 'fn3-end', 'fn2-end', 'fn1-end']
```

## References

- Concept [Middleware](https://developer.mozilla.org/en-US/docs/Glossary/Middleware)
- Design Pattern [middleware](https://www.patterns.dev/vanilla/mediator-pattern/)

## Solutions

```ts
type MiddlewareFn =
  | ((context: any, next: () => Promise<void>) => Promise<void>)
  | ((context: any, next: () => Promise<void>) => void);

export default function middlewares(
  ...fns: Array<MiddlewareFn>
): (context?: any) => Promise<void> {
  return async function (context: any = {}) {
    const next = async (index: number) => {
      if (index === fns.length) {
        return;
      }

      const middleware = fns[index];
      await middleware(context, () => next(index + 1));
    };

    await next(0);
  };
}
```
