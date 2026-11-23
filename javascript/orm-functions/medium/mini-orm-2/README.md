# Mini Object-relational Mapper II

This is a follow-up to [Mini Object-relational Mapper](../mini-orm/).

In the base question, `findMany()` only supported exact top-level equality. In this question, the write APIs stay the same, but `findMany()` becomes more expressive.

Implement an enhanced `MiniORM` class with support for operator-based `filtering` and `sorting`.

Like the base question, model names are still arbitrary. The instance properties should come from the constructor input keys, not hardcoded names like `products`.

## `MiniORM` API

`new MiniORM(data)`

Creates a `MiniORM` instance from an object whose keys are model names and whose values are arrays of records.

Each `MiniORM` instance should expose one delegate per model name. For example, if the input contains an `article` array, the instance should expose `db.article`.

Records may contain arbitrary top-level properties.

`model.findMany([args])`
Returns the matching records for the model in insertion order.

`args.where`
Each field can now be either:

- an exact-match value, as in the original question, or
- an operator object with one or more of the following keys:
  - `in`
  - `gt`
  - `gte`
  - `lt`
  - `lte`
  - `contains`

Records should still match with AND semantics across different fields. If multiple operators are provided for the same field, that field must satisfy all of them.

Return a new array each time. Returned records only need shallow copies.

`model.create({ data })`
Appends `data` to the model and returns the created record.

Store a shallow copy of data, not the original object reference.

| Argument | Type | Description |
| --- | --- | --- |
| `data` | `Object` | The record to append. |

`model.update({ where, data })`
Finds the single matching record, shallow-merges data into it, stores the result, and returns the updated record.

| Argument | Type | Description |
| --- | --- | --- |
| `where` | `Object` | Exact-match conditions that are guaranteed to match exactly one record. |
| `data` | `Object` | Partial fields to shallow-merge into the stored record. |

`model.delete({ where })`
Finds the single matching record, removes it from the model, and returns the deleted record.

| Argument | Type | Description |
| --- | --- | --- |
| `where` | `Object` | Exact-match conditions that are guaranteed to match exactly one record. |

`args.orderBy`
An optional object containing exactly one field name whose value is either `'asc'` or `'desc'`.

Apply sorting after filtering.

## Examples

```ts
const db = new MiniORM({
  product: [
    { id: 1, name: 'Keyboard', price: 40, category: 'hardware' },
    { id: 2, name: 'Notebook', price: 12, category: 'stationery' },
    { id: 3, name: 'Monitor', price: 220, category: 'hardware' },
    { id: 4, name: 'Mouse Pad', price: 18, category: 'hardware' },
  ],
});

db.product.findMany({
  where: {
    price: { gte: 10, lt: 50 },
    category: { in: ['hardware', 'accessories'] },
    name: { contains: 'o' },
  },
  orderBy: { price: 'asc' },
});
// [
//   { id: 4, name: 'Mouse Pad', price: 18, category: 'hardware' },
//   { id: 1, name: 'Keyboard', price: 40, category: 'hardware' },
// ]
```

## Notes

- `create()`, `update()`, and `delete()` keep the same behavior as in Mini Object-relational Mapper.
- `update()` and `delete()` still use exact top-level equality in their where clauses.
- `contains` only needs to work for string fields.
- Sorting should preserve insertion order when records have equal values for the `orderBy` field.
- You do not need `skip`, `take`, nested filtering, `OR`, or `NOT`.
- Inputs are guaranteed to be valid.
- `db.user` is only one possible delegate. `db.product`, `db.article`, and other model names should work too.

## Techniques

- Object-oriented programming
- Dynamic object properties
- Shallow cloning
- Filtering arrays of records

## References

- [prisma.io](https://www.prisma.io/docs/orm/fundamentals/reading-data)

## Solutions

```ts
export type ORMRecord = Record<string, unknown>;
export type ORMData = Record<string, Array<ORMRecord>>;
export type ExactWhere = Record<string, unknown>;
type SortDirection = "asc" | "desc";

function isOperatorObject(value: unknown): value is FieldOperators {
  return value != null && !Array.isArray(value) && typeof value === "object";
}

function matchesFieldCondition(
  recordValue: unknown,
  condition: WhereValue,
): boolean {
  // Objects without recognized operators still use the exact-match behavior from part I.
  if (!isOperatorObject(condition)) {
    return recordValue === condition;
  }

  if ("in" in condition && !condition.in!.includes(recordValue)) {
    return false;
  }

  if ("gt" in condition && !((recordValue as any) > (condition.gt as any))) {
    return false;
  }

  if ("gte" in condition && !((recordValue as any) >= (condition.gte as any))) {
    return false;
  }

  if ("lt" in condition && !((recordValue as any) < (condition.lt as any))) {
    return false;
  }

  if ("lte" in condition && !((recordValue as any) <= (condition.lte as any))) {
    return false;
  }

  if (
    "contains" in condition &&
    (typeof recordValue !== "string" ||
      !recordValue.includes(condition.contains!))
  ) {
    return false;
  }

  return true;
}

function matchesWhere(record: ORMRecord, where: WhereClause = {}): boolean {
  return Object.entries(where).every(([field, condition]) =>
    matchesFieldCondition(record[field], condition),
  );
}

function sortRecords(
  records: Array<ORMRecord>,
  orderBy?: Record<string, SortDirection>,
): Array<ORMRecord> {
  if (orderBy == null) {
    return records;
  }

  const [field, direction] = Object.entries(orderBy)[0];
  const multiplier = direction === "desc" ? -1 : 1;

  return records.sort((recordA, recordB) => {
    if (recordA[field] === recordB[field]) {
      return 0;
    }

    return (recordA[field] as any) > (recordB[field] as any)
      ? multiplier
      : -multiplier;
  });
}

export interface FieldOperators {
  in?: Array<unknown>;
  gt?: unknown;
  gte?: unknown;
  lt?: unknown;
  lte?: unknown;
  contains?: string;
}

export type WhereValue = unknown | FieldOperators;
export type WhereClause = Record<string, WhereValue>;

export default class MiniORM {
  [modelName: string]: any;

  #data: ORMData;

  constructor(data: ORMData) {
    this.#data = Object.fromEntries(
      Object.entries(data).map(([key, value]) => {
        const records = value.map((record): ORMRecord => {
          return { ...record };
        });
        return [key, records];
      }),
    );

    for (const key in data) {
      this[key] = this.#createDelegate(key);
    }
  }

  #createDelegate(name: string) {
    return {
      findMany: (args: any) => {
        const records = this.#data[name]
          .filter((record) => matchesWhere(record, args?.where ?? {}))
          .map((record): ORMRecord => {
            return { ...record };
          });

        return sortRecords(records, args?.orderBy);
      },
      create: ({ data }: { data: ORMRecord }) => {
        const record = { ...data };
        this.#data[name].push(record);
        return record;
      },
      update: (args: any) => {
        const data = this.#data[name];

        const record = data.find((elem) => {
          return Object.entries(args.where).every(([key, value]) => {
            return elem[key] === value;
          });
        });

        if (!record) {
          return {};
        }

        for (const [key, value] of Object.entries(args.data)) {
          record[key] = value;
        }

        return { ...record };
      },
      delete: (args: any) => {
        const data = this.#data[name];

        const index = data.findIndex((elem) => {
          return Object.entries(args.where).every(([key, value]) => {
            return elem[key] === value;
          });
        });

        if (index < 0) {
          return {};
        }

        const record = data[index];
        this.#data[name].splice(index, 1);
        return { ...record };
      },
    };
  }
}
```
