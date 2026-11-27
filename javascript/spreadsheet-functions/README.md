# Spreadsheet

## Examples

```ts
const sheet = new Spreadsheet();

sheet.setCell('A1', '=A1 + 1');

sheet.getCell('A1'); // '#CYCLE!'
```

```ts
const sheet = new Spreadsheet();

sheet.setCell('A1', '=B1 + 1');
sheet.setCell('B1', '=A1 + 1');
sheet.setCell('C1', '=B1 + 2');

sheet.getCell('C1'); // '#CYCLE!'
```

Overwriting a cell should update the dependency graph too.

```ts
const sheet = new Spreadsheet();

sheet.setCell('A1', '=B1 + 1');
sheet.setCell('B1', '=A1 + 1');

sheet.getCell('A1'); // '#CYCLE!'

sheet.setCell('B1', 3);

sheet.getCell('A1'); // 4
```
