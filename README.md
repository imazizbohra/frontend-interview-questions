# frontend-interview-questions

Collection of frontend interview questions

## Javascript

- [Design Pattern - Easy](./javascript/design-pattern/easy/)
- [Functions - Easy](./javascript/functions/easy/)
- [Jquery Functions - Easy](./javascript/jquery-functions/easy/)
- [Lodash Functions - Easy](./javascript/lodash-functions/easy/)
- [Promise Functions - Easy](./javascript/promise-functions/easy/)
- [Prototype Functions - Easy](./javascript/prototype-functions/easy/)

## React

[Hooks - Easy](./react/hooks/easy/)

## Techniques

### Check out-of-bound index

```ts
if (index < -length || index >= length) {
  // out-of-bound index
}
```

### Normalizing array index

```ts
const nIndex =
    index < 0 ? Math.max(index + length, 0) : Math.min(index, length);
```

### Find type of a value

```ts
// output format: "array"

const getType = (value) => {
  if (value === null) return "null";

  if (value === undefined) return "undefined";

  return Object.getPrototypeOf(value)?.constructor?.name?.toLowerCase() ?? "object";
};
```

```ts
// output format: "[object Array]"

const getType = (value) => {
  return Object.prototype.toString.call(value);
};
```

### Check if a value is primitive

```ts
const isPrimitive = (
  value: unknown,
): value is null | undefined | string | number | boolean | symbol | bigint => {
  return Object(value) !== value;
};
```

### Cound no of items in a range

```ts
// if step = 0, handle divide by zero
// if range is negative then step also becomes negative; 
// division by negative produces positive value

const count = (end - start) / (step || 1);
```

## Credits

- [greatfrontend.com](https://www.greatfrontend.com/)
- [patterns.dev](https://www.patterns.dev/)
