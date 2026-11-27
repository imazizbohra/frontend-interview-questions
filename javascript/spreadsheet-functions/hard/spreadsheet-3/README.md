# Spreadsheet III

This is a follow-up to [Spreadsheet II](../../medium/spreadsheet-2/).

Keep the same Spreadsheet API and the same formula syntax from the previous question, but now formulas may contain cycles.

Add support for:

- direct cycles
- indirect cycles
- self-references

If a cell is part of a cycle, `getCell(cellId)` should return `'#CYCLE!'`.

If a cell depends on another cell that is part of a cycle, it should also return `'#CYCLE!'`.

## Spreadsheet API

### `new Spreadsheet()`

Creates a Spreadsheet instance with no cells.

### `spreadsheet.setCell(cellId, input)`

Stores a value for `cellId`.

| Parameter | Type | Description |
| --- | --- | --- |
| `cellId` | `string` | A cell reference such as A1 or B12. |
| `input` | `number \| string` | Either a number or a formula string beginning with `=`. |

### `spreadsheet.getCell(cellId)`

Returns the evaluated numeric value of `cellId`, `0` for unset cells, or `'#CYCLE!'` when `cellId` depends on a cycle.

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

## Notes

- Keep the same left-to-right evaluation rules from [Spreadsheet II](../../medium/spreadsheet-2/).
- Spaces inside formulas should be ignored.
- Formula operands are either cell references or unsigned numeric literals.
- You do not need ranges, functions, parentheses, precedence, or invalid-formula handling in this question.

## References

- Javascript [RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)

## Solutions

```ts
export type CellInput = number | string;
const CELL_REFERENCE_REGEX = /^[A-Z]+[1-9][0-9]*$/;

export default class Spreadsheet {
  readonly #cells = new Map<string, CellInput>();

  setCell(cellId: string, input: CellInput): void {
    this.#cells.set(cellId, input);
  }

  getCell(cellId: string): number {
    return this.#evaluateCell(cellId);
  }

  #evaluateCell(cellId: string) {
    const input = this.#cells.get(cellId);

    if (!input) {
      return 0;
    }

    if (typeof input === "number") {
      return input;
    }

    return this.#evaluateFormula(input);
  }

  #evaluateFormula(formula: string): number {
    const expression = formula.replace(/\s+/g, " ").slice(1).trim();

    return this.#evaluateExpression(expression.split(" "));
  }

  #evaluateExpression(tokens: string[]): number {
    const stack: Array<string | number> = [];

    for (const token of tokens) {
      stack.push(token);

      if (stack.length === 3) {
        const leftOperand = this.#resolveOperand(stack.shift() as string);
        const operator = stack.shift() as string;
        const rightOperand = this.#resolveOperand(stack.shift() as string);

        stack.push(this.#calculate(leftOperand, operator, rightOperand));
      }
    }

    return Number(stack[0]);
  }

  #resolveOperand(operand: string): number {
    if (CELL_REFERENCE_REGEX.test(operand)) {
      return this.#evaluateCell(operand);
    }

    return Number(operand);
  }

  #calculate(
    leftOperand: number,
    operator: string,
    rightOperand: number,
  ): number {
    return new Function("a", "b", "c", `return a ${operator} c`)(
      leftOperand,
      operator,
      rightOperand,
    );
  }
}
```
