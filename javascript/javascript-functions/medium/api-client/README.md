# API Client

Many apps wrap `fetch()` in small helper APIs so request configuration can be built up fluently before the request is sent.

In this question, implement `createAPI(baseURL)`. It should return an object with a `url(path)` method, and `url(path)` should return a request builder that supports:

- `setHeaders(headers)`
- `get()`
- `post(body?)`

Each call to api.url(path) should create a fresh request builder for that path.

## Example(s)

```ts
const api = createAPI('https://example.com/');

api.url('/posts').setHeaders({ Authorization: 'Bearer xyz' }).get();

// fetch('https://example.com/posts', {
//   method: 'GET',
//   headers: { Authorization: 'Bearer xyz' },
// });


api
  .url('/users')
  .setHeaders({ 'Content-Type': 'application/json' })
  .post(JSON.stringify({ name: 'Alice' }));

// fetch('https://example.com/users', {
//   method: 'POST',
//   headers: { 'Content-Type': 'application/json' },
//   body: '{"name":"Alice"}',
// });

api
  .url('/posts')
  .setHeaders({ Authorization: 'Bearer old-token' })
  .setHeaders({ Authorization: 'Bearer new-token', Accept: 'application/json' })
  .get();

// fetch('https://example.com/posts', {
//   method: 'GET',
//   headers: {
//     Authorization: 'Bearer new-token',
//     Accept: 'application/json',
//   },
// });
```

## Notes

- Resolve request URLs with `new URL(path, baseURL)`.
- `get()` and `post()` should return the raw `fetch()` promise.
- url(path) is always called before `get()` or `post()`.
- Inputs are guaranteed valid.
- You do not need query params, retries, interceptors, response parsing, or automatic JSON serialization in this question.

## Reference(s)

- Web API [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)

## Solution(s)

```ts
export type HeadersMap = Record<string, string>;

export type APIRequestBuilder = {
  setHeaders(headers: HeadersMap): APIRequestBuilder;
  get(): Promise<Response>;
  post(body?: BodyInit): Promise<Response>;
};

export default function createAPI(baseURL: string): {
  url(path: string): APIRequestBuilder;
} {
  class RequestBuilder implements APIRequestBuilder {
    #reqPath: string;
    #reqHeaders: HeadersMap;

    constructor(path: string) {
      this.#reqPath = path;
      this.#reqHeaders = {};
    }

    setHeaders(headers: HeadersMap): APIRequestBuilder {
      Object.assign(this.#reqHeaders, headers);
      return this;
    }

    get(): Promise<Response> {
      const url = new URL(this.#reqPath, baseURL).toString();
      return fetch(url, {
        method: "GET",
        headers: this.#reqHeaders,
      });
    }

    post(body?: BodyInit): Promise<Response> {
      const url = new URL(this.#reqPath, baseURL).toString();
      return fetch(url, {
        method: "POST",
        headers: this.#reqHeaders,
        body,
      });
    }
  }

  return {
    url: function (path: string): APIRequestBuilder {
      return new RequestBuilder(path);
    },
  };
}
```
