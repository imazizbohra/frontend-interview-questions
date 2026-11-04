# Techniques

## Check negative array index

```ts
if (index < -length || index >= length) {
  // out-of-bound index
}
```

## Normalizing array index

```ts
const nStart: number =
    start < 0 ? Math.max(start + length, 0) : Math.min(start, length);
```
