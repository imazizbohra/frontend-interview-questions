# URLSearchParams II

This is a follow-up to [URLSearchParams](../url-search-params/).

`URLSearchParams` is a browser API for parsing and working with URL query strings while preserving duplicate keys and insertion order.

In this question, implement an interview-sized subset of `URLSearchParams` as a class named `URLSearchParamsPolyfill`.

Do not call or wrap the native `URLSearchParams` in your implementation. Build the behavior yourself.

This question should support:

- A leading `?` being ignored.
- Repeated keys being preserved in insertion order.
- Bare keys like debug being treated as debug=.
- Percent-decoding.
- `+` being treated as a space while parsing.

`URLSearchParamsPolyfill` API
In addition to the constructor and read methods from the previous question, implement the following:

`params.append(name, value)`
Appends a new name-value pair to the end of the stored list.

`params.set(name, value)`
Replaces all existing pairs for name with a single pair. If the name already exists, keep the position of its first occurrence. If the name does not exist, append the new pair to the end.

`params.delete(name)`
Removes all pairs whose name matches name.

This question should still preserve:

- Duplicate keys in insertion order.
- `get`, `getAll`, and `has` from the previous question.
- Query-string-only construction with decoding behavior identical to the previous question.

## Arguments

`query:(string)` A query string to parse. Defaults to `''`.

## Returns

- Returns a new `URLSearchParamsPolyfill` instance.
- Mutation methods do not return anything.

## Examples

```ts
const params = new URLSearchParamsPolyfill('a=1&a=2&b=3');

params.append('a', '4');
params.getAll('a'); // ['1', '2', '4']
```

set() should collapse duplicates into one value.

```ts
const params = new URLSearchParamsPolyfill('a=1&a=2&b=3');

params.set('a', '5');
params.getAll('a'); // ['5']
params.get('b'); // '3'
```

delete() should remove all matching pairs.

```ts
const params = new URLSearchParamsPolyfill('tag=js&tag=ts&mode=fast');

params.delete('tag');
params.has('tag'); // false
params.get('mode'); // 'fast'
```

## References

- JavaScript [decodeURIComponent](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/decodeURIComponent)
- Web API [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)

## Solutions

```ts
const decodeQuery = (query = "") => {
  return decodeURIComponent(query.replace(/\+/g, " "));
};

export default class URLSearchParamsPolyfill {
  #query: string;
  #pairs: Array<Array<string>>;

  constructor(query = "") {
    this.#query = query.startsWith("?") ? query.slice(1) : query;

    this.#pairs = this.#query.split("&").map((query) => {
      const [key, ...rest] = query.split("=");
      const value = [...rest].join("");
      return [decodeQuery(key), decodeQuery(value)] as const;
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

  append(name: string, value: string): void {
    this.#pairs.push([decodeQuery(name), decodeQuery(value)]);
  }

  set(name: string, value: string): void {
    const pairs = this.#pairs.filter(([key, value]) => {
      return key === name;
    });

    if (pairs.length === 0) {
      this.append(name, value);
      return;
    } else if (pairs.length === 1) {
      pairs[0][1] = decodeQuery(value);
      return;
    }

    this.delete(name);
    this.append(name, value);
  }

  delete(name: string): void {
    this.#pairs = this.#pairs.filter(([key, value]) => {
      return key !== name;
    });
  }
}
```
