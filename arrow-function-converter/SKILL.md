---
name: arrow-function-converter
description: >
  Expert JavaScript/TypeScript refactoring engineer that converts every function declaration and function expression
  in the codebase to arrow function syntax. Use this skill whenever the user wants to modernize JS/TS code,
  convert functions to arrow syntax, standardize function style, or refactor function declarations.
  Trigger on phrases like: "convert to arrow functions", "use arrow functions everywhere", "modernize JS syntax",
  "refactor function declarations", "convert function expressions", "arrow function refactor", or any request
  to change function syntax in a JavaScript or TypeScript codebase. Always use this skill before making
  any function-syntax changes across multiple files.
---

# Arrow Function Converter Skill

You are an expert JavaScript/TypeScript refactoring engineer. Your task is to convert **every function declaration and function expression** in the entire codebase to arrow function syntax — with zero exceptions (where safely possible).

---

## Phase 1 — Discovery

Recursively scan all source files in the project. List every file that contains at least one convertible function. Skip non-JS/TS files (e.g. `.json`, `.md`, `.css`).

---

## Phase 2 — Conversion Rules

Apply the following transformations consistently across every file:

| Pattern | Convert to |
|---|---|
| `function foo() {}` | `const foo = () => {}` |
| `function foo(a, b) { return a + b; }` | `const foo = (a, b) => a + b;` (implicit return for single-expression bodies) |
| `const foo = function() {}` | `const foo = () => {}` |
| Anonymous callbacks: `arr.map(function(x) { return x * 2; })` | `arr.map(x => x * 2)` |
| Named function expressions: `const foo = function bar() {}` | `const foo = () => {}` (drop the internal name) |
| Default-exported functions: `export default function() {}` | `export default () => {}` |
| Named exports: `export function foo() {}` | `export const foo = () => {}` |

---

## Phase 3 — Special Cases (still convert, handle carefully)

- **`this` binding**: Arrow functions do not bind their own `this`. If the original function uses `this`, add a comment `// NOTE: converted from function — verify 'this' context` directly above the arrow function.
- **`arguments` object**: Arrow functions do not have `arguments`. Replace any usage with a rest parameter (`...args`) before converting.
- **Generator functions** (`function*`): Flag with `// NOTE: generator — manual review required` and leave unconverted.
- **Constructors** (functions used with `new`): Flag with `// NOTE: constructor — cannot convert to arrow function` and leave unconverted.
- **Object method shorthand**: `{ foo() {} }` → `{ foo: () => {} }`. Verify `this` context as above.
- **Class methods**: Leave class method syntax (`class Foo { bar() {} }`) **unchanged** — converting them to arrow properties changes prototype behaviour.

---

## Phase 4 — Apply & Validate

- Rewrite each file in place with all conversions applied.
- Preserve all existing logic, spacing, imports, exports, and comments exactly — only function syntax changes.
- Do not reformat unrelated code.
- After all files are processed, output a summary table:

```
File | Functions converted | Flagged for review
```