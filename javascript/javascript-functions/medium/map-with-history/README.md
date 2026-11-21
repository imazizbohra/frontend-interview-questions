# Map With History

Implement a `MapWithHistory` class that can store multiple string values for the same key at different timestamps and read them back later.

When reading a key, return the value stored for the greatest timestamp that is less than or equal to the query timestamp.

A key may accumulate many historical values over time, but timestamps for the same key are always added in strictly increasing order.

## MapWithHistory API

`new MapWithHistory()`
Creates a new empty `MapWithHistory` instance.

Stored values are isolated within each instance.

`mapWithHistory.set(key, value, timestamp)`
Stores `value` for `key` at the given `timestamp`.

| Parameter | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to store a value for. |
| `value` | `string` | The string value to store. |
| `timestamp` | `number` | The timestamp associated with the value. |

`mapWithHistory.get(key, timestamp)`

Returns the value stored for the largest timestamp associated with `key` such that `storedTimestamp <= timestamp`.

If no such value exists, return `''`.

| Parameter | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to read from. |
| `timestamp` | `number` | The timestamp to query. |

## Examples

```ts
const mapWithHistory = new MapWithHistory();

mapWithHistory.set('language', 'JavaScript', 1);
mapWithHistory.set('language', 'TypeScript', 4);

mapWithHistory.get('language', 1);
// 'JavaScript'

mapWithHistory.get('language', 3);
// 'JavaScript'

mapWithHistory.get('language', 4);
// 'TypeScript'

mapWithHistory.get('language', 10);
// 'TypeScript'
```

Queries earlier than the first stored timestamp should return `''`.

```ts
const mapWithHistory = new MapWithHistory();

mapWithHistory.set('theme', 'dark', 5);

mapWithHistory.get('theme', 4);
// ''
```

## Notes

- Timestamps for the same key are always added in strictly increasing order.
- Different keys may reuse the same timestamp.
- Only string values are in scope for this question.
- You do not need deletion, iteration, range queries, or out-of-order inserts.
- Inputs are guaranteed valid.

## Solutions

```ts
// Approach 1: Linear search from the end

type TimeEntry = {
  timestamp: number;
  value: string;
};

interface IMapWithHistory {
  set(key: string, value: string, timestamp: number): void;
  get(key: string, timestamp: number): string;
}

export default class MapWithHistory implements IMapWithHistory {
  _store: Map<string, Array<TimeEntry>>;

  constructor() {
    this._store = new Map();
  }

  set(key: string, value: string, timestamp: number): void {
    let history = this._store.get(key);

    if (history == null) {
      history = [];
      this._store.set(key, history);
    }

    history.push({ timestamp, value });
  }

  get(key: string, timestamp: number): string {
    const history = this._store.get(key);

    if (history == null) {
      return '';
    }

    // History is append-only, so the first match scanning backward is the latest valid value.
    for (let index = history.length - 1; index >= 0; index -= 1) {
      const entry = history[index];

      if (entry.timestamp <= timestamp) {
        return entry.value;
      }
    }

    return '';
  }
}
```

```ts
// Approach 2: Binary search

type TimeEntry = {
  timestamp: number;
  value: string;
};

interface IMapWithHistory {
  set(key: string, value: string, timestamp: number): void;
  get(key: string, timestamp: number): string;
}

export default class MapWithHistory implements IMapWithHistory {
  _store: Map<string, Array<TimeEntry>>;

  constructor() {
    this._store = new Map();
  }

  set(key: string, value: string, timestamp: number): void {
    let history = this._store.get(key);

    if (history == null) {
      history = [];
      this._store.set(key, history);
    }

    history.push({ timestamp, value });
  }

  get(key: string, timestamp: number): string {
    const history = this._store.get(key);

    if (history == null) {
      return '';
    }

    let left = 0;
    let right = history.length - 1;
    let result = '';

    // Keep searching right after a match to find the latest entry at or before timestamp.
    while (left <= right) {
      const middle = Math.floor((left + right) / 2);
      const entry = history[middle];

      if (entry.timestamp <= timestamp) {
        result = entry.value;
        left = middle + 1;
      } else {
        right = middle - 1;
      }
    }

    return result;
  }
}
```
