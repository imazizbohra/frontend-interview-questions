# URLSearchParams

`URLSearchParams` is a browser API for parsing and working with URL query strings while preserving duplicate keys and insertion order.

In this question, implement an interview-sized subset of `URLSearchParams` as a class named `URLSearchParamsPolyfill`.

Do not call or wrap the native `URLSearchParams` in your implementation. Build the behavior yourself.

This question should support:

- A leading `?` being ignored.
- Repeated keys being preserved in insertion order.
- Bare keys like debug being treated as debug=.
- Percent-decoding.
- `+` being treated as a space while parsing.

## Arguments

`query:(string)` A query string to parse. Defaults to `''`.

## Returns

Returns a new `URLSearchParamsPolyfill` instance.

## Examples

```ts
const params = new URLSearchParamsPolyfill('?a=1&a=2&debug');

params.get('a'); // '1'
params.getAll('a'); // ['1', '2']
params.get('debug'); // ''
params.get('missing'); // null
```

`+` and percent-encoded values should be decoded while parsing.

```ts
const params = new URLSearchParamsPolyfill('city=New+York&note=a%26b');

params.get('city'); // 'New York'
params.get('note'); // 'a&b'
```

Duplicate keys should be preserved.

```ts
const params = new URLSearchParamsPolyfill('tag=js&tag=ts');

params.get('tag'); // 'js'
params.getAll('tag'); // ['js', 'ts']
```

## References

- JavaScript [decodeURIComponent](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/decodeURIComponent)
- Web API [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)

## Solutions

```ts
export default class URLSearchParamsPolyfill {
  #query: string;
  #pairs: Array<Array<string>>;

  constructor(query = "") {
    this.#query = query.startsWith("?") ? query.slice(1) : query;

    this.#pairs = this.#query.split("&").map((query) => {
      const [key, ...rest] = query.split("=");

      const sKey = key.replace(/\+/g, " ");
      const sValue = [...rest].join("").replace(/\+/g, " ");

      return [decodeURIComponent(sKey), decodeURIComponent(sValue)] as const;
    });
  }

  get(name: string): string | null {
    const pair = this.#pairs.find(([key, value]) => {
      return key === name;
    });

    return pair ? pair[1] : null;
  }

  getAll(name: string): Array<string> {
    return this.#pairs
      .filter(([key, value]) => {
        return key === name;
      })
      .map(([_, value]) => {
        return value;
      });
  }

  has(name: string): boolean {
    return this.#pairs.some(([key, value]) => {
      return key === name;
    });
  }
}
```
