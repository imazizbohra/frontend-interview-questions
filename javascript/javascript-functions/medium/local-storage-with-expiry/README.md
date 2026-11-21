# localStorage With Expiry

`localStorage` persists data until it is explicitly removed. Sometimes that is exactly what you want, but sometimes stored values should become invalid after a period of time.

Implement a `LocalStorageWithExpiry` class that wraps `localStorage` and supports storing string values with an optional expiry.

This question is intentionally scoped to a small wrapper. It is not a full replacement for the native Storage API.

## Examples

```ts
const storage = new LocalStorageWithExpiry();

storage.setItem('theme', 'dark');
storage.getItem('theme'); // 'dark'
```

```ts
const storage = new LocalStorageWithExpiry();

storage.setItem('token', 'abc123', { ttl: 60_000 });
storage.getItem('token'); // 'abc123'

// After 60 seconds:
storage.getItem('token'); // null
```

```ts
const storage = new LocalStorageWithExpiry();

storage.setItem('session', 'active', {
  expiresAt: new Date('2026-12-31T23:59:59.000Z'),
});
```

## Notes

- Only string values are in scope for this question.
- Values not written by this wrapper, including malformed JSON and unrecognized stored shapes, should be treated as missing and return null.
- You do not need to implement `removeItem()`, `clear()`, `clearExpired()`, or any other `Storage` APIs.

## References

- Window [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- JavaScript [JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON)

## Solutions

```ts
const ENTRY_MARKER = "__lswe";

interface StorageOptions {
  ttl?: number;
  expiresAt?: number | Date;
}

interface ILocalStorageWithExpiry {
  getItem(key: string): string | null;
  setItem(key: string, value: string, options?: StorageOptions): void;
}

interface StorageEntry {
  [ENTRY_MARKER]: true;
  value: string;
  expiry: number | null;
}

function isStorageEntry(value: unknown): value is StorageEntry {
  return (
    typeof value === "object" &&
    value !== null &&
    (value as Record<string, unknown>)[ENTRY_MARKER] === true &&
    typeof (value as Record<string, unknown>).value === "string" &&
    "expiry" in value
  );
}

function resolveExpiry(options?: StorageOptions): number | null {
  if (options?.ttl != null && options?.expiresAt != null) {
    throw new TypeError(
      'Cannot specify both "ttl" and "expiresAt". Use one or the other.',
    );
  }

  if (options?.ttl != null) {
    return Date.now() + options.ttl;
  }

  if (options?.expiresAt != null) {
    return options.expiresAt instanceof Date
      ? options.expiresAt.getTime()
      : options.expiresAt;
  }

  return null;
}

export default class LocalStorageWithExpiry implements ILocalStorageWithExpiry {
  getItem(key: string): string | null {
    const raw = localStorage.getItem(key);

    if (raw === null) {
      return null;
    }

    let entry: unknown;

    try {
      entry = JSON.parse(raw);
    } catch {
      return null;
    }

    if (!isStorageEntry(entry)) {
      return null;
    }

    if (entry.expiry != null && Date.now() > entry.expiry) {
      // Expired entries are cleaned up lazily the next time they are read.
      localStorage.removeItem(key);
      return null;
    }

    return entry.value;
  }

  setItem(key: string, value: string, options?: StorageOptions): void {
    const entry: StorageEntry = {
      [ENTRY_MARKER]: true,
      value,
      expiry: resolveExpiry(options),
    };

    localStorage.setItem(key, JSON.stringify(entry));
  }
}
```
