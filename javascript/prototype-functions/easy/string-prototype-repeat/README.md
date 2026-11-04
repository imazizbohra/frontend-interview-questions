# String.prototype.repeat

`String.prototype.repeat` creates and returns a new string containing the original string repeated a given number of times.

Implement your own `String.prototype.repeat` without calling the native repeat method. To avoid overwriting the actual `String.prototype.repeat`, implement it as `String.prototype.myRepeat`.

This question uses an interview-sized contract instead of full spec parity:

- Omitted count behaves like 0.
- NaN behaves like 0.
- Negative counts and Infinity throw RangeError.
- null and undefined receivers should throw TypeError.
- Non-null and non-undefined receivers should be coerced with String(...).
Positive fractional counts are truncated toward zero.

## Example(s)

```ts
'ab'.myRepeat(3); // 'ababab'
'x'.myRepeat(1); // 'x'
'x'.myRepeat(0); // ''
```

Counts should be normalized before repeating.

```ts
'abc'.myRepeat(); // ''
'abc'.myRepeat(NaN); // ''
'abc'.myRepeat(2.8); // 'abcabc'
```

The method should also work with non-string receivers after coercion.

```ts
String.prototype.myRepeat.call(123, 2); // '123123'
String.prototype.myRepeat.call(true, 3); // 'truetruetrue'
String.prototype.myRepeat.call({ toString: () => 'x' }, 2); // 'xx'
```

Invalid receivers and counts should throw.

```ts
String.prototype.myRepeat.call(null, 2); // TypeError
'abc'.myRepeat(-1); // RangeError
'abc'.myRepeat(Infinity); // RangeError
```

## Notes

- The native String.prototype.repeat is off-limits for this question.
- Engine max string length limits are out of scope.
- Symbol receivers are out of scope.
- Full coercion of arbitrary non-number count inputs is out of scope.

## Reference(s)

- String [repeat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/repeat)

## Solution(s)

```ts
String.prototype.myRepeat = function (this: unknown, count?: number) {
  if (count == null || Number.isNaN(count)) {
    count = 0;
  }

  if (count < 0 || count === Infinity) {
    throw new RangeError(`Invalid count value: ${count}`);
  }

  if (this == null) {
    throw new TypeError(
      "String.prototype.myRepeat called on null or undefined",
    );
  }

  const str = String(this);
  count = Math.trunc(count);

  return new Array(count).fill(str).join("");
};
```
