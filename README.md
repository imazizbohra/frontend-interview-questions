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

### Check negative array index

```ts
if (index < -length || index >= length) {
  // out-of-bound index
}
```

### Normalizing array index

```ts
const nStart: number =
    start < 0 ? Math.max(start + length, 0) : Math.min(start, length);
```


## Credits

- [greatfrontend.com](https://www.greatfrontend.com/)
- [patterns.dev](https://www.patterns.dev/)
