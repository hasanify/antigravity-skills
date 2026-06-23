---
name: typescript-any-eliminator
description: >
  TypeScript specialist that audits an entire codebase for all uses of the `any` type and aggressively
  replaces every occurrence with precise, correct TypeScript types. Use this skill whenever the user wants
  to eliminate `any` types, improve type safety, fix TypeScript strict mode errors, or tighten type
  coverage across a project. Trigger on phrases like: "remove any types", "fix any types", "replace any
  with proper types", "improve type safety", "no-explicit-any", "strict TypeScript", "type everything
  properly", "get rid of any", or any request to improve TypeScript typing quality — even casual ones
  like "our types are a mess". Always use this skill before making type changes across multiple files.
---

# TypeScript `any` Eliminator Skill

You are a TypeScript expert. Your job is to hunt down every `any` type in the codebase and replace it with the most precise, correct type possible — no exceptions, no shortcuts, no `unknown` as a lazy escape hatch.

---

## Phase 1 — Full Audit

Recursively scan every `.ts` and `.tsx` file. Find and catalogue **every** occurrence of `any`, including:

- Explicit annotations: `foo: any`, `bar: any[]`, `(x: any) => void`
- Return types: `(): any`
- Generics: `Array<any>`, `Promise<any>`, `Record<string, any>`
- Cast expressions: `as any`, `<any>value`
- Implicit `any` from untyped function parameters or missing return types
- Type aliases: `type Foo = any`
- Re-exported types that wrap `any`
- `@ts-ignore` and `@ts-expect-error` comments suppressing type errors — flag these for review

Group findings by file. For each occurrence, record:
- File path and line number
- The current `any` usage in context (show the surrounding 1–2 lines)
- Inferred replacement strategy (see Phase 2)

Output this as a structured audit report before making any changes.

---

## Phase 2 — Replacement Strategy

For each `any`, determine the most precise replacement using this decision hierarchy:

### 1. Infer from usage (preferred)
Trace how the variable, parameter, or return value is actually used throughout the codebase. Let usage patterns drive the type:
- If it's always indexed with string keys → `Record<string, T>`
- If it's always called as a function → `(...args: T[]) => R`
- If it's used in array methods → `T[]`
- If it's narrowed with `instanceof` or `typeof` checks → use a union type

### 2. Use existing domain types
Check for interfaces, types, or classes already defined in the project that match the shape. Always prefer reusing established types over inventing new ones.

### 3. Introduce a precise new type or interface
If no existing type fits, define one. Place it in the appropriate `/types` file. Name it clearly after what it represents — not what it stores (e.g. `UserPayload`, not `DataObject`).

### 4. Use generics to preserve flexibility
If the `any` exists to make a utility function flexible, replace it with a generic type parameter:
- `(x: any) => any` → `<T>(x: T) => T`
- `Array<any>` in a utility → `<T>(arr: T[]) => T[]`

### 5. Use `unknown` only when truly unknown
Use `unknown` only for values that are genuinely unknown at compile time (e.g. `JSON.parse`, `catch` clause errors, external API responses before validation). Always pair with a type guard or `zod`/`io-ts` schema for narrowing.

### 6. Use union types for multi-shape values
If a value can legitimately be one of several types, use a union: `string | number | boolean` rather than `any`.

### Never do these:
- Do not replace `any` with `object` — it's almost as unsafe
- Do not replace `any` with `{}` — it accepts everything except `null`/`undefined`
- Do not use `unknown` where the type is actually knowable from context
- Do not suppress errors with `@ts-ignore` or `as unknown as T` double-cast hacks

---

## Phase 3 — Apply Fixes

Fix one file at a time, in dependency order (types and utilities before features, features before UI).

For each file:
1. Apply all replacements
2. If new shared types are introduced, create or update the appropriate file in `/types`
3. Update any downstream files whose types changed as a result
4. Remove any `@ts-ignore` or `@ts-expect-error` that is no longer needed after the fix
5. Show a diff summary: lines changed, types introduced, `any` occurrences eliminated

---

## Phase 4 — Strict Mode Verification

After all fixes are applied:

- Check `tsconfig.json` for `"strict": true` and `"noImplicitAny": true` — if absent, recommend adding them
- Verify the project compiles without type errors: run `tsc --noEmit`
- If compilation fails, fix the remaining errors before finishing
- List any `any` occurrences that genuinely could not be eliminated and explain exactly why, with a suggested future path to resolution

---

## Phase 5 — Final Report

Output a summary table:

```
File | any occurrences found | Eliminated | Remaining | Notes
```

Then list all new types and interfaces introduced, with the file they were added to.

---

## Rules

- Never change runtime behaviour — only type annotations change
- Never delete code to avoid a type error — fix the type
- If fixing a type in one file causes type errors in dependents, fix the dependents too (cascade fully)
- Prefer narrowing over widening: always choose the most specific type that is still correct
- If a third-party library has poor types (e.g. returns `any`), wrap the call site with a typed assertion and add a comment explaining the source