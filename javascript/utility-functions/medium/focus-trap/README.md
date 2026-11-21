# Focus Trap

Modal dialogs and popovers often need to keep keyboard focus inside themselves while they are open.

In this question, implement `createFocusTrap(container)`, a simplified version of the core behavior from [focus-trap](https://github.com/focus-trap/focus-trap).

The returned trap should:

- Move focus to the first tabbable descendant when `activate()` is called.
- Wrap `Tab` from the last tabbable element back to the first.
- Wrap `Shift+Tab` from the first tabbable element back to the last.
- If focus is outside while the trap is active, move it to the first tabbable element on Tab and the last on Shift+Tab.
- Stop trapping once `deactivate()` is called.

## Arguments

`createFocusTrap(container)`

`container (HTMLElement)`: The container whose descendants should keep focus within the trap.

## Returns

Returns an object with the following methods:

`trap.activate()`
Activates the trap and moves focus to the first tabbable descendant inside container.

`trap.deactivate()`
Deactivates the trap and removes its keyboard listeners.

## Examples

```ts
const modal = document.querySelector('#modal');
const trap = createFocusTrap(modal);

trap.activate();
document.activeElement;
// First tabbable element inside #modal.
```

```ts
trap.deactivate();
// Later Tab presses should no longer be intercepted by the trap.
```

## Notes

- In this question, `container` is guaranteed to have at least one tabbable descendant.
- Treat the following as tabbable: enabled `button`, `input`, `select`, `textarea`, `a[href]`, and `elements` with non-negative `tabIndex`.
- Repeated `activate()` or `deactivate()` calls should be safe and must not install duplicate listeners.
- You do not need to support `Escape`, outside clicks, outside focus changes, or empty traps in this question.

## References

- Github [focus-trap](https://github.com/focus-trap/focus-trap)

## Solutions

```ts
export type FocusTrap = {
  activate: () => void;
  deactivate: () => void;
};

export default function createFocusTrap(container: HTMLElement): FocusTrap {
  let isActive = false;
  const tabbableElems = [
    ...container.querySelectorAll<HTMLElement>(
      "button, input, select, textarea, a[href], [tabindex]",
    ),
  ].filter((el) => el.tabIndex >= 0 && !el.matches(":disabled"));
  const length = tabbableElems.length;

  function listener(event: KeyboardEvent) {
    if (event.key !== "Tab") {
      return;
    }

    event.preventDefault();

    const firstElement = tabbableElems[0];
    const lastElement = tabbableElems[tabbableElems.length - 1];
    const activeElement = event.target as HTMLElement;

    // If focus escaped the trap, pull it back to the edge the user is tabbing toward.
    if (!container.contains(activeElement)) {
      (event.shiftKey ? lastElement : firstElement).focus();
      return;
    }

    if (event.shiftKey && activeElement === firstElement) {
      lastElement.focus();
      return;
    }

    if (!event.shiftKey && activeElement === lastElement) {
      firstElement.focus();
    }
  }

  function activate() {
    if (isActive) {
      return;
    }

    isActive = true;
    tabbableElems[0].focus();

    document.addEventListener("keydown", listener);
  }

  function deactivate() {
    if (isActive) {
      isActive = false;
      document.removeEventListener("keydown", listener);
    }
  }

  return {
    activate,
    deactivate,
  };
}
```
