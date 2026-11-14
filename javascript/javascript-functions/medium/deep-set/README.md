# Deep Set

[dset](https://github.com/lukeed/dset) is a tiny utility for writing values into deeply nested objects and arrays.

Implement a simplified `deepSet(obj, path, value)` function.

The function should mutate `obj` in place by writing `value` at `path`. If part of the path does not exist yet, create the missing objects or arrays along the way.

**Note**: Similar to [unsquash object](../unsquash-object/)

## Arguments

`deepSet(obj, path, value)`

- `obj` (`object | Array<unknown>`): The object to mutate.
- `path` (`string | Array<string | number>`): The path where the value should be written.
  - A string path uses . as the separator, such as 'user.profile.name'.
  - An array path can contain strings and numbers, such as `['user', 'profile', 'name']` or `['items', 0, 'label']`.
- `value` (`unknown`): The value to write at the target path.

## Example(s)

Write into nested objects.

```ts
const object = { user: { profile: { name: 'John' } } };

deepSet(object, 'user.profile.age', 30);

// {
//   user: {
//     profile: {
//       name: 'John',
//       age: 30,
//     },
//   },
// }
```

Write into arrays using numeric path segments.

```ts
const object = { items: ['a', 'b', 'c'] };

deepSet(object, 'items.1', 'updated');

// { items: ['a', 'updated', 'c'] }
```

Create mixed object/array structures when needed.

```ts
const object = {};

deepSet(object, 'a.0.b.1', 2);

// {
//   a: [
//     {
//       b: [undefined, 2],
//     },
//   ],
// }
```

If traversal needs to continue through a primitive value, replace that branch with a new container.

```ts
const object = { config: true };

deepSet(object, 'config.theme.name', 'dark');

// {
//   config: {
//     theme: {
//       name: 'dark',
//     },
//   },
// }
```

## Notes

- Missing containers should be created automatically.
- Use an array when the next path segment is a numeric index. Otherwise, use an object.
- This question is intentionally limited:
  - Bracket syntax like `a[0].b` is out of scope.
  - The function should mutate the input object instead of returning a new one.
  - Production-hardening details like prototype-pollution protection are out of scope.

## Reference(s)

- Github [dset](https://github.com/lukeed/dset)

## Solution(s)

```ts
type PathSegment = string | number;
type Path = string | Array<PathSegment>;
type Container = Record<PathSegment, unknown> | Array<unknown>;

export default function deepSet(
  obj: Container,
  path: Path,
  value: unknown,
): void {
  const pathArr = typeof path === "string" ? path.split(".") : path;
  let current = obj;

  for (let i = 0; i < pathArr.length - 1; i++) {
    const firstPath = pathArr[i] as number;
    const nextPath = pathArr[i + 1];

    if (typeof current[firstPath] === "object" && current[firstPath] !== null) {
      // reuse existing container
      current = current[firstPath] as Container;
    } else {
      // create new container
      current[firstPath] = Number.isNaN(Number(nextPath)) ? {} : [];
      current = current[firstPath] as Container;
    }
  }

  const lastPath = pathArr[pathArr.length - 1] as number;
  current[lastPath] = value;
}
```
