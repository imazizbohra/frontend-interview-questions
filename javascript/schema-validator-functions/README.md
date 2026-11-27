# Schema Validator

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

## References

- [zod.dev](https://zod.dev/)
- [joi.dev](https://joi.dev/)
