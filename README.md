# ⚖️ Zod Compare

[![Build](https://github.com/dramaorg/quo-expedita/actions/workflows/build.yml/badge.svg)](https://github.com/dramaorg/quo-expedita/actions/workflows/build.yml)
[![npm](https://img.shields.io/npm/v/@dramaorg/quo-expedita)](https://www.npmjs.com/package/@dramaorg/quo-expedita)

Compare two [Zod](https://zod.dev/) schemas recursively.

`@dramaorg/quo-expedita` provides functions to compare Zod schemas, allowing you to determine whether two schemas are the same or compatible.

## Installation

```bash
# npm
npm install zod @dramaorg/quo-expedita

# yarn
yarn add zod @dramaorg/quo-expedita

# pnpm
pnpm add zod @dramaorg/quo-expedita
```

## Usage

```ts
import { z } from "zod";
import { isSameType, isCompatibleType } from "@dramaorg/quo-expedita";

isSameType(z.string(), z.string()); // true
isSameType(z.string(), z.number()); // false
isSameType(
  z.object({ name: z.string(), other: z.number() }),
  z.object({ name: z.string() }),
);
// false

isCompatibleType(
  z.object({ name: z.string(), other: z.number() }),
  z.object({ name: z.string() }),
);
// true
```

## Advanced Usage

### Custom Rules

You can use `createCompareFn` to create a custom comparison function.

```ts
import {
  createCompareFn,
  isSameTypePresetRules,
  defineCompareRule,
} from "@dramaorg/quo-expedita";

const customRule = defineCompareRule(
  "compare description",
  (a, b, next, recheck, context) => {
    // If the schemas are not having the same description, return false
    if (a.description !== b.description) {
      return false;
    }
    return next();
  },
);

const strictIsSameType = createCompareFn([
  customRule,
  ...isSameTypePresetRules,
]);
```

## Debugging

You can pass a `context` object to the comparison functions to get more information about the comparison process.

```ts
const context = {
  stacks: [],
};
isSameType(
  z.object({ name: z.string(), other: z.number() }),
  z.object({ name: z.string(), other: z.string() }),
  context,
);

// type stacks = { name: string; target: [a: ZodType, b: ZodType]; }[]
console.log(context.stacks);
```

## Caveats

The default rules `isSameTypePresetRules` will disregard any custom validations like `min`, `max`, `length`, among others. Additionally, these default rules cannot be utilized for comparing `ZodLazy`, `ZodEffects`, `ZodDefault`, `ZodCatch`, `ZodPipeline`, `ZodTransformer`, `ZodError` types.

If there is a necessity to compare these types, custom rules can be established using `defineCompareRule`.

## API

### `isSameType`

Compares two Zod schemas and returns `true` if they are the same.

```ts
import { isSameType } from "@dramaorg/quo-expedita";

type isSameType: (a: ZodType, b: ZodType, context?: CompareContext) => boolean;
```

### `createCompareFn`

Creates a custom comparison function.

```ts
import { createCompareFn, defineCompareRule } from "@dramaorg/quo-expedita";

type defineCompareRule = (
  name: string,
  rule: CompareFn,
) => {
  name: string;
  rule: CompareFn;
};

type createCompareFn = (rules: CompareRule[]) => typeof isSameType;

// Example
const isSameType = createCompareFn(isSameTypePresetRules);
const isCompatibleType = createCompareFn(isCompatibleTypePresetRules);
```

### `isCompatibleType` (Experimental API)

Compares two Zod schemas and returns `true` if they are compatible.

```ts
import { isCompatibleType } from "@dramaorg/quo-expedita";
// The `higherType` should be a looser type
// The `lowerType` should be a stricter type
type isCompatibleType: (higherType: ZodType, lowerType: ZodType) => boolean;
```

### Preset Rules

You can use the preset rules `isSameTypePresetRules` and `isCompatibleTypePresetRules` to create custom comparison functions.

```ts
import { isSameTypePresetRules, isCompatibleTypePresetRules } from "@dramaorg/quo-expedita";

type isSameTypePresetRules: CompareRule[];
type isCompatibleTypePresetRules: CompareRule[];

// Example
const yourIsSameType = createCompareFn([customRule, ...isSameTypePresetRules]);
```

### Types

```ts
type CompareContext = {
  stacks?: {
    name: string;
    target: [a: ZodType, b: ZodType];
  }[];
} & Record<string, unknown>;

type CompareFn = (
  a: ZodType,
  b: ZodType,
  next: () => boolean,
  recheck: (a: ZodType, b: ZodType) => boolean,
  context: CompareContext,
) => boolean;

type CompareRule = {
  name: string;
  compare: CompareFn;
};
```

## License

MIT
