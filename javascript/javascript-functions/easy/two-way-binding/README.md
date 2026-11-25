# Two-way Binding

Two-way data binding keeps program state and a form control synchronized. Implement `model(state, input)` for one state object and one `HTMLInputElement`.

```ts
interface ModelState {
  value: string;
}

function model(state: ModelState, input: HTMLInputElement): void;
```

The binding follows three rules:

- Initialize `input.value` from `state.value`. State is the initial source of truth.
- After binding, assigning to `state.value` immediately updates `input.value`.
- When the input dispatches a `change` event, read its current value into `state.value`.

## Examples

```ts
const state = { value: 'hello' };
const input = document.createElement('input');

model(state, input);
console.log(input.value); // 'hello'

state.value = 'from state';
console.log(input.value); // 'from state'

input.value = 'from input';
input.dispatchEvent(new Event('change'));
console.log(state.value); // 'from input'
```

Limit the implementation to the `value` property and the `change` event. Multiple controls, nested paths, validation, cleanup, and repeated calls to `model` for the same state object are outside the scope.

After binding, `value` should remain an enumerable and configurable own property of state.

## Solutions

```ts
export type ModelState = {
  value: string;
};

export default function model(
  state: ModelState,
  input: HTMLInputElement,
): void {
  let { value } = state;
  input.value = value;

  Object.defineProperty(state, "value", {
    get() {
      return value;
    },
    set(nextValue) {
      value = nextValue;
      input.value = value;
    },
  });

  const listener = (event: Event) => {
    const target = event.target as HTMLInputElement;
    state.value = target.value;
  };

  input.addEventListener("change", listener);
}
```
