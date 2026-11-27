# Spreadsheet II

This is a follow-up to [Spreadsheet](../spreadsheet/).

Keep the same `Spreadsheet` API, but now formulas can use more arithmetic operators: `+`, `-`, `*`, `/`.

This question still keeps the formula language intentionally small:

- Evaluate strictly left-to-right.
- Do not apply normal math precedence.
- Do not support parentheses.
- Inputs are still guaranteed acyclic.

## Spreadsheet API

`new Spreadsheet()`
Creates a Spreadsheet instance with no cells.

`spreadsheet.setCell(cellId, input)`
Stores a value for `cellId`.
markdown

| Parameter | Type | Description |
| --- | --- | --- |
| `cellId` | `string` | A cell reference such as A1 or B12. |
| `input` | `number \| string` | Either a number or a formula string beginning with `=`. |

`spreadsheet.getCell(cellId)`
Returns the evaluated numeric value of `cellId`.

If `cellId` has not been set, return `0`.

## Examples

```ts
const sheet = new Spreadsheet();

sheet.setCell('A1', 10);
sheet.setCell('B1', '=A1 - 2 * 3');

sheet.getCell('B1'); // 24
```

The expression is evaluated left-to-right, so it behaves like `(10 - 2) * 3`.

```ts
const sheet = new Spreadsheet();

sheet.setCell('A1', 8);
sheet.setCell('B1', '=A1 / 2 + 3');

sheet.getCell('B1'); // 7
```

Referenced cells can still contain formulas.

```ts
const sheet = new Spreadsheet();

sheet.setCell('A1', 4);
sheet.setCell('B1', '=A1 + 6');
sheet.setCell('C1', '=B1 * 2 - 3');

sheet.getCell('C1'); // 17
```

## Notes

- Cell references use uppercase A1-style labels such as `A1` and `B12`.
- Spaces inside formulas should be ignored.
- Formula operands are either cell references or unsigned numeric literals.
- You do not need subtraction, multiplication, division, cycles, ranges, or invalid-formula handling in this question.

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
