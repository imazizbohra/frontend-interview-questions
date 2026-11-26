# Mini Object-relational Mapper III

This is a follow-up to [Mini Object-relational Mapper II](../../medium/mini-orm-2/).

In this question, filtering and sorting still work the same way, but findMany() also needs to shape the returned data:

- `select` picks which top-level fields to return.
- `include` loads single-level relations using explicit relation metadata.

Both model names and relation names are arbitrary. They should be derived from the constructor inputs and relation metadata, not hardcoded.

## Additional constructor argument

`new MiniORM(data, relations)
`
`relations` is an optional object describing how models are connected.

Each relation definition has:

| Field | Type | Description |
| --- | --- | --- |
| `model` | `string` | The related model name. |
| `type` | `'one' \| 'many'` | Whether the relation returns one record or many. |
| `sourceKey` | `string` | The field on the current record to read from. |
| `targetKey` | `string` | The field on the related model to compare against. |

## `findMany()` additions

### `args.select`

An optional object whose keys are field names and whose values are booleans.

Return only the fields whose values are `true`.

### `args.include`

An optional object whose keys are relation names and whose values are booleans.

For each `true` relation:

- If the relation type is `'one'`, return the related record or `null`.
- If the relation type is `'many'`, return an array of related records in insertion order.

## Examples

```ts
const db = new MiniORM(
  {
    playlist: [
      { id: 1, name: 'Road Trip' },
      { id: 2, name: 'Focus' },
    ],
    track: [
      { id: 10, title: 'Intro', playlistId: 1 },
      { id: 11, title: 'Highway', playlistId: 1 },
      { id: 12, title: 'Deep Work', playlistId: 2 },
    ],
  },
  {
    playlist: {
      tracks: {
        model: 'track',
        type: 'many',
        sourceKey: 'id',
        targetKey: 'playlistId',
      },
    },
    track: {
      playlist: {
        model: 'playlist',
        type: 'one',
        sourceKey: 'playlistId',
        targetKey: 'id',
      },
    },
  },
);

db.playlist.findMany({
  select: { id: true, name: true },
  include: { tracks: true },
});
// [
//   {
//     id: 1,
//     name: 'Road Trip',
//     tracks: [
//       { id: 10, title: 'Intro', playlistId: 1 },
//       { id: 11, title: 'Highway', playlistId: 1 },
//     ],
//   },
//   {
//     id: 2,
//     name: 'Focus',
//     tracks: [{ id: 12, title: 'Deep Work', playlistId: 2 }],
//   },
// ]
```

## Notes

- `select` and `include` apply only at the top level of `findMany()`.
- Included related records should be shallow clones, not live references into storage.
- Ignore relation names that are absent from the relation metadata, as well as include entries set to `false`.
- Nested `select`, nested `include`, and nested writes are out of scope.
- `create()`, `update()`, and `delete()` keep the same behavior as in the previous questions.
- Neither model names nor relation names are special-cased. Build them from the provided data and relation config.

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

type SortDirection = 'asc' | 'desc';

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

export interface RelationDefinition {
  model: string;
  type: 'one' | 'many';
  sourceKey: string;
  targetKey: string;
}

export type RelationMap = Record<string, Record<string, RelationDefinition>>;
export type SelectMap = Record<string, boolean>;
export type IncludeMap = Record<string, boolean>;

interface FindManyArgs {
  where?: WhereClause;
  orderBy?: Record<string, SortDirection>;
  select?: SelectMap;
  include?: IncludeMap;
}

interface CreateArgs {
  data: ORMRecord;
}

interface UpdateArgs {
  where: ExactWhere;
  data: Partial<ORMRecord>;
}

interface DeleteArgs {
  where: ExactWhere;
}

interface ModelDelegate {
  findMany(args?: FindManyArgs): Array<ORMRecord>;
  create(args: CreateArgs): ORMRecord;
  update(args: UpdateArgs): ORMRecord;
  delete(args: DeleteArgs): ORMRecord;
}

const OPERATOR_KEYS = new Set(['in', 'gt', 'gte', 'lt', 'lte', 'contains']);

function cloneRecord(record: ORMRecord): ORMRecord {
  return { ...record };
}

function cloneRecords(records: Array<ORMRecord>): Array<ORMRecord> {
  return records.map(cloneRecord);
}

function isOperatorObject(value: unknown): value is FieldOperators {
  return (
    value != null &&
    !Array.isArray(value) &&
    typeof value === 'object' &&
    Object.keys(value).some((key) => OPERATOR_KEYS.has(key))
  );
}

function matchesExactWhere(record: ORMRecord, where: ExactWhere = {}): boolean {
  return Object.entries(where).every(
    ([field, value]) => record[field] === value,
  );
}

function matchesFieldCondition(
  recordValue: unknown,
  condition: WhereValue,
): boolean {
  if (!isOperatorObject(condition)) {
    return recordValue === condition;
  }

  if ('in' in condition && !condition.in!.includes(recordValue)) {
    return false;
  }

  if ('gt' in condition && !((recordValue as any) > (condition.gt as any))) {
    return false;
  }

  if ('gte' in condition && !((recordValue as any) >= (condition.gte as any))) {
    return false;
  }

  if ('lt' in condition && !((recordValue as any) < (condition.lt as any))) {
    return false;
  }

  if ('lte' in condition && !((recordValue as any) <= (condition.lte as any))) {
    return false;
  }

  if (
    'contains' in condition &&
    (typeof recordValue !== 'string' ||
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
  const multiplier = direction === 'desc' ? -1 : 1;

  return records.slice().sort((recordA, recordB) => {
    if (recordA[field] === recordB[field]) {
      return 0;
    }

    return (recordA[field] as any) > (recordB[field] as any)
      ? multiplier
      : -multiplier;
  });
}

export default class MiniORM {
  [modelName: string]: any;

  _data: Map<string, Array<ORMRecord>>;
  _relations: RelationMap;

  constructor(data: ORMData, relations: RelationMap = {}) {
    this._data = new Map();
    this._relations = relations;

    Object.entries(data).forEach(([modelName, records]) => {
      this._data.set(modelName, cloneRecords(records));
      this[modelName] = this._createDelegate(modelName);
    });
  }

  _createDelegate(modelName: string): ModelDelegate {
    return {
      findMany: (args: FindManyArgs = {}) => this._findMany(modelName, args),
      create: (args: CreateArgs) => this._create(modelName, args),
      update: (args: UpdateArgs) => this._update(modelName, args),
      delete: (args: DeleteArgs) => this._delete(modelName, args),
    };
  }

  _findMany(modelName: string, args: FindManyArgs = {}): Array<ORMRecord> {
    const records = this._getModelRecords(modelName);
    const filteredRecords = records.filter((record) =>
      matchesWhere(record, args.where),
    );
    const sortedRecords = sortRecords(filteredRecords, args.orderBy);

    // Shape only after filtering/sorting so those steps still see the full stored record.
    return sortedRecords.map((record) =>
      this._shapeRecord(modelName, record, args),
    );
  }

  _shapeRecord(
    modelName: string,
    record: ORMRecord,
    args: FindManyArgs,
  ): ORMRecord {
    const nextRecord =
      args.select == null
        ? cloneRecord(record)
        : this._applySelect(record, args.select);

    if (args.include == null) {
      return nextRecord;
    }

    Object.entries(args.include).forEach(([relationName, shouldInclude]) => {
      if (!shouldInclude) {
        return;
      }

      const relationDefinition = this._relations[modelName]?.[relationName];

      if (relationDefinition == null) {
        return;
      }

      // Includes resolve from the original stored record, not the select-trimmed shape.
      nextRecord[relationName] = this._resolveRelation(
        record,
        relationDefinition,
      );
    });

    return nextRecord;
  }

  _applySelect(record: ORMRecord, select: SelectMap): ORMRecord {
    const nextRecord: ORMRecord = {};

    Object.entries(select).forEach(([field, selected]) => {
      if (selected) {
        nextRecord[field] = record[field];
      }
    });

    return nextRecord;
  }

  _resolveRelation(
    record: ORMRecord,
    relationDefinition: RelationDefinition,
  ): unknown {
    const relatedRecords = this._getModelRecords(relationDefinition.model);
    const matches = relatedRecords.filter(
      (relatedRecord) =>
        relatedRecord[relationDefinition.targetKey] ===
        record[relationDefinition.sourceKey],
    );

    return relationDefinition.type === 'one'
      ? matches.length === 0
        ? null
        : cloneRecord(matches[0])
      : cloneRecords(matches);
  }

  _create(modelName: string, args: CreateArgs): ORMRecord {
    const records = this._getModelRecords(modelName);
    const nextRecord = cloneRecord(args.data);

    records.push(nextRecord);

    return cloneRecord(nextRecord);
  }

  _update(modelName: string, args: UpdateArgs): ORMRecord {
    const records = this._getModelRecords(modelName);
    const recordIndex = records.findIndex((record) =>
      matchesExactWhere(record, args.where),
    );
    const nextRecord = {
      ...records[recordIndex],
      ...args.data,
    };

    records[recordIndex] = nextRecord;

    return cloneRecord(nextRecord);
  }

  _delete(modelName: string, args: DeleteArgs): ORMRecord {
    const records = this._getModelRecords(modelName);
    const recordIndex = records.findIndex((record) =>
      matchesExactWhere(record, args.where),
    );
    const [deletedRecord] = records.splice(recordIndex, 1);

    return cloneRecord(deletedRecord);
  }

  _getModelRecords(modelName: string): Array<ORMRecord> {
    return this._data.get(modelName) ?? [];
  }
}
```
