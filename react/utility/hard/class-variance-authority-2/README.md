# Class Variance Authority II

This is a follow-up to [Class Variance Authority](../../medium/class-variance-authority/).

Implement an enhanced simplified `cva` function. This version needs to support `base`, `variants`, `defaultVariants`, and `compoundVariants`.

You do not need to support `compoundVariants[].className` or top-level `class` / `className`.

## Arguments

`cva(base, config)`

- `base` (`string | null | undefined`): Base classes that are always included.
- `config` (`object`, optional): Configuration for the generated function.
  - `variants`: An object whose keys are variant names and whose values map variant values to class names.
  - `defaultVariants`: Default variant values to use when the returned function is called without that variant.
  - `compoundVariants`: An array of entries that each contain:
    - Variant conditions keyed by variant name.
    - Condition values that are either a single accepted value or an array of accepted values.
    - A `class` field to append when all conditions match.

## Returns

Returns a function that accepts an object of selected variant values and returns the final class string.

## Notes

- Ignore unknown variant names in the returned function's argument.
- Ignore resolved variant values that do not exist in the corresponding variant map when adding normal variant classes.
- Match `compoundVariants` against the resolved variant values after applying `defaultVariants`.
- Append all matching compound variant `class` values after the normal variant classes, in `compoundVariants` array order.
- Join all truthy class names with a single space and no leading or trailing whitespace.

## Examples

```tsx
const button = cva('btn', {
  variants: {
    intent: {
      primary: 'btn-primary',
      secondary: 'btn-secondary',
    },
    size: {
      small: 'btn-small',
      medium: 'btn-medium',
    },
    disabled: {
      true: 'btn-disabled',
      false: null,
    },
  },
  defaultVariants: {
    intent: 'primary',
    size: 'medium',
    disabled: false,
  },
  compoundVariants: [
    {
      intent: 'primary',
      size: 'medium',
      class: 'btn-primary-medium',
    },
    {
      intent: ['primary', 'secondary'],
      disabled: true,
      class: 'btn-disabled-interactive',
    },
  ],
});

button();
// 'btn btn-primary btn-medium btn-primary-medium'

button({ intent: 'secondary', disabled: true });
// 'btn btn-secondary btn-medium btn-disabled btn-disabled-interactive'
```

## References

Github [cva](https://github.com/joe-bell/cva)

## Solutions

```ts
export type ClassValue = string | false | null | undefined;
export type VariantValue = string | boolean | null | undefined;
export type VariantClasses = Record<string, ClassValue>;
export type VariantSchema = Record<string, VariantClasses>;
export type VariantSelection = Record<string, VariantValue>;
export type CompoundVariantValue = VariantValue | Array<VariantValue>;
export type CompoundVariant = Record<string, CompoundVariantValue> & {
  class: ClassValue;
};

export interface CVAConfig {
  variants?: VariantSchema;
  defaultVariants?: VariantSelection;
  compoundVariants?: Array<CompoundVariant>;
}

function matchesCompoundVariant(
  compoundVariant: CompoundVariant,
  resolvedVariants: VariantSelection,
): boolean {
  for (const variantName in compoundVariant) {
    if (
      !Object.hasOwn(compoundVariant, variantName) ||
      variantName === 'class'
    ) {
      continue;
    }

    const expectedValue = compoundVariant[variantName];
    const resolvedValue = resolvedVariants[variantName];

    // Arrays mean "any of these values" for one compound condition.
    if (Array.isArray(expectedValue)) {
      if (!expectedValue.includes(resolvedValue)) {
        return false;
      }

      continue;
    }

    if (resolvedValue !== expectedValue) {
      return false;
    }
  }

  return true;
}

export default function cva(
  base?: ClassValue,
  config: CVAConfig = {},
): (props?: VariantSelection) => string {
  const { variants = {}, defaultVariants = {}, compoundVariants = [] } = config;

  return function getClassName(props: VariantSelection = {}): string {
    const classes: Array<string> = [];
    // Resolve every variant once so the compound pass can reuse the same
    // props-vs-default precedence that normal variants use.
    const resolvedVariants: VariantSelection = {};

    if (base) {
      classes.push(base);
    }

    for (const variantName in variants) {
      if (!Object.hasOwn(variants, variantName)) {
        continue;
      }

      const variantMap = variants[variantName];
      const variantValue = props[variantName] ?? defaultVariants[variantName];

      resolvedVariants[variantName] = variantValue;

      if (variantValue == null) {
        continue;
      }

      const variantKey = String(variantValue);

      if (!Object.hasOwn(variantMap, variantKey)) {
        continue;
      }

      const variantClass = variantMap[variantKey];

      if (variantClass) {
        classes.push(variantClass);
      }
    }

    // Compound checks happen after the first pass so they can read the fully
    // resolved variant state without recomputing it.
    for (const compoundVariant of compoundVariants) {
      if (!matchesCompoundVariant(compoundVariant, resolvedVariants)) {
        continue;
      }

      if (compoundVariant.class) {
        classes.push(compoundVariant.class);
      }
    }

    return classes.join(' ');
  };
}
```
