# TypeScript Study Guide

A structured reference covering TypeScript fundamentals, tooling, configuration, and common patterns.

## Table of Contents

1. [Introduction to TypeScript](#1-introduction-to-typescript)
2. [Installation & Setup](#2-installation--setup)
3. [Variables and Type Inference](#3-variables-and-type-inference)
4. [Type Annotations](#4-type-annotations)
5. [Migrating JavaScript to TypeScript](#5-migrating-javascript-to-typescript)
6. [Configuring TypeScript (`tsconfig.json`)](#6-configuring-typescript-tsconfigjson)
7. [Interfaces](#7-interfaces)
8. [Generics](#8-generics)
9. [Built-in Utility Types](#9-built-in-utility-types)
10. [Type Assertions](#10-type-assertions)
11. [Runtime Validation with Zod](#11-runtime-validation-with-zod)
12. [Further Resources](#12-further-resources)

---

## 1. Introduction to TypeScript

TypeScript is a **statically and strongly typed** superset of JavaScript. It adds a type system on top of JavaScript and compiles down to plain JavaScript before it runs, since browsers and Node.js cannot execute TypeScript directly.

- **File extension:** `.ts` (or `.tsx` for files containing JSX)
- **Execution model:** TypeScript is compiled to JavaScript; it does not run natively
- **Type system:** Static (checked at compile time) and strong (types are enforced, with minimal implicit coercion)

### Official Resources

- Handbook: <https://www.typescriptlang.org/docs>
- Introduction to JS vs TS: <https://www.typescriptlang.org/docs/handbook/intro-to-js-ts.html>
- TypeScript Playground: <https://www.typescriptlang.org/play>

---

## 2. Installation & Setup

### Installing the TypeScript Compiler

```bash
npm i -g typescript
tsc -v
```

### Checking JavaScript Files with TypeScript

TypeScript can type-check plain `.js` files without converting them, which is useful for gradually adopting TypeScript in an existing codebase.

```bash
tsc --checkJs --noEmit --strict ./1-javascript/exercise/checkMe.js
```

| Flag | Purpose |
|------|---------|
| `--checkJs` | Type-checks `.js` files in addition to `.ts` files |
| `--noEmit` | Performs type checking only; does not output compiled JS |
| `--strict` | Enables the full set of strict type-checking rules |

---

## 3. Variables and Type Inference

When you declare a variable with `let`, you can **reassign its value**, but TypeScript still infers a type from the initial assignment and enforces that type afterward — unless the variable was explicitly typed as a union or `any`.

```typescript
let count = 4;
count = 10;        // OK — same type (number)
count = "ten";      // Error — string is not assignable to number
```

If you need a variable that can legitimately hold different types over time, declare it with a union type up front (see [Union Types](#union-types)) rather than relying on type widening.

`const`, by contrast, prevents reassignment entirely once a value is assigned.

---

## 4. Type Annotations

### Typed Variables

You can explicitly annotate a variable's type at the point of declaration:

```typescript
let n: number = 4;
let s: string = "hello";
```

### Typing Before Assignment

A variable can be typed first and assigned a value afterward:

```typescript
let n: number;
n = 4;
```

### Primitive Types

```typescript
let n: number = 4;
let s: string = "hi";
let b: boolean = true;
let missing: undefined;
let nothing: null;
```

### Literal Types

A literal type restricts a variable to one or more **exact values**, rather than a general type like `string`. This achieves a similar goal to enums, but using a union of specific values.

```typescript
let state: "alive" | "barely";

state = "alive"; // OK
state = "dead";  // Error: Type '"dead"' is not assignable to type '"alive" | "barely"'
```

### Typed Functions

```typescript
// JavaScript
function add(a, b) {
  return a + b;
}

// TypeScript
function add(a: number, b: number): number {
  return a + b;
}
```

### Typed Arrays

```typescript
const digits: number[] = [1, 2, 3];
```

### Typed Objects

```typescript
let user: { name: string; id: number };

user = { name: "John", id: 1 };
```

### Optional Properties

A `?` marks a property as optional, meaning it can be omitted or `undefined`.

```typescript
let user: { name: string; id?: number };

user = { name: "John" }; // OK — id is optional
```

### Union Types

A union type allows a variable to hold one of several specified types.

```typescript
let something: number | string;

something = 4;
something = "hello";
```

### Type Narrowing (Type Guards)

When a variable has a union type, TypeScript narrows it to a more specific type within conditional checks. This lets you safely access type-specific properties or methods.

```typescript
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // id is narrowed to string here
  } else {
    console.log(id); // id is narrowed to number here
  }
}
```

### Type Aliases

A type alias gives a name to a type so it can be reused across your codebase.

```typescript
type StringOrNumber = string | number;
type LogLevel = "log" | "error" | "warn";

let name: StringOrNumber = "John";
let age: StringOrNumber = 19;
```

---

## 5. Migrating JavaScript to TypeScript

### Running TypeScript Directly with `tsx`

`tsx` is a TypeScript-aware runner that executes `.ts` files directly, without a separate compile step — useful during development.

```bash
npm i -g tsx
tsx typefile.ts
tsx --watch typefile.ts
```

### Compiling with `tsc`

`tsc` is the official TypeScript compiler. It type-checks code and emits plain JavaScript.

```bash
tsc --strict typefile.ts
```

### Useful Compiler Flags

| Flag | Purpose |
|------|---------|
| `--checkJs` | Also type-checks `.js` files, not just `.ts` files |
| `--noEmit` | Type-checks without producing compiled output |
| `--target es2020` | Sets the JavaScript version the compiled output should target |
| `--strict` | Enables all strict type-checking options |

---

## 6. Configuring TypeScript (`tsconfig.json`)

### Installing TypeScript as a Project Dependency

```bash
npm i -D typescript
```

### Generating a Config File

```bash
tsc --init
```

This creates a `tsconfig.json` file where compiler options, included files, and project settings are defined.

### Frontend Setup (React + Vite)

```bash
npm i -D @tsconfig/recommended
npm i -D @tsconfig/vite-react
```

`tsconfig.json`:

```json
{
  "extends": [
    "@tsconfig/recommended/tsconfig.json",
    "@tsconfig/vite-react/tsconfig.json"
  ]
}
```

### Backend Setup (Node.js + Express)

```bash
npm i -D @tsconfig/node-lts
npm i -D @types/node
```

`tsconfig.json`:

```json
{
  "extends": ["@tsconfig/node-lts"],
  "compilerOptions": {
    "types": ["node"]
  }
}
```

### Installing Type Declarations for Package Dependencies

Many JavaScript packages don't ship their own types. Install community-maintained type declarations from `@types`:

```bash
npm i -D @types/<packageName>
```

Example — Express:

```bash
npm i -D @types/express
```

### Inspecting the Resolved Configuration

`tsc --showconfig` prints the fully resolved configuration, including options inherited from any `extends` entries. This is useful for debugging unexpected compiler behavior.

```bash
tsc --showconfig
```

Reference: [Choosing Compiler Options](https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html)

---

## 7. Interfaces

Interfaces describe the **shape of an object**: which properties it has and their types.

```typescript
interface User {
  id: number;
}
```

### Extending Interfaces

Interfaces can extend other interfaces, inheriting their members and adding new ones.

```typescript
interface Human extends User {
  name: string;
  age: number;
}
```

### Interfaces vs. Type Aliases

Both `type` and `interface` can describe object shapes, and in many cases are interchangeable. The key practical difference:

- **`interface`** can be extended (`extends`) and can be merged via repeated declarations — making it a natural fit for object shapes that may grow or be implemented by classes.
- **`type`** can describe object shapes too, but is also required for unions, intersections, primitives, and other non-object type compositions (e.g. `type StringOrNumber = string | number`).

A common convention: use `interface` for object/class shapes, and `type` for everything else (unions, aliases, utility type compositions).

---

## 8. Generics

Generics let you write reusable types and functions with a **type variable** that is specified later, when the type or function is actually used.

```typescript
type Nullable<T> = T | null;

let something: Nullable<string> = "hello";
```

Here, `T` is a placeholder filled in with `string` at the point of use, making `Nullable<T>` reusable for any type.

---

## 9. Built-in Utility Types

TypeScript ships with several utility types for transforming existing types:

| Utility | Description |
|---------|-------------|
| `Readonly<T>` | Makes all properties of `T` read-only after initialization |
| `Partial<T>` | Makes all properties of `T` optional |
| `Pick<T, K>` | Creates a type with only the selected properties `K` from `T` |
| `Omit<T, K>` | Creates a type with all properties of `T` **except** `K` |

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type ReadonlyUser = Readonly<User>;     // all properties immutable
type PartialUser = Partial<User>;       // all properties optional
type UserPreview = Pick<User, "id" | "name">;  // only id and name
type UserWithoutEmail = Omit<User, "email">;   // everything except email
```

---

## 10. Type Assertions

A type assertion (`as`) overrides the type the compiler would otherwise infer. It does **not** perform any runtime conversion or validation — it only affects type checking at compile time.

```typescript
const someValue: unknown = "this is a string";
const strLength: number = (someValue as string).length;
```

> Use type assertions sparingly. Because they bypass the compiler's own checks, an incorrect assertion can hide real bugs that would otherwise be caught at compile time.

---

## 11. Runtime Validation with Zod

TypeScript types only exist at compile time — they are erased once your code is compiled to JavaScript. This means TypeScript **cannot** validate the shape of data arriving at runtime (e.g. API responses, form input, or file contents). [Zod](https://zod.dev) fills this gap by defining schemas that validate data while it runs.

```typescript
import { z } from "zod";

const UserSchema = z.object({
  username: z.string(),
});

// Throws an error if validation fails
const user = UserSchema.parse({ username: "john_doe" });

// Returns a result object instead of throwing
const result = UserSchema.safeParse({ username: "john_doe" });
if (result.success) {
  console.log(result.data);
}
```

Zod can also derive a static TypeScript type directly from a schema, keeping runtime validation and compile-time types in sync:

```typescript
type User = z.infer<typeof UserSchema>;
```

---

## 12. Further Resources

- TypeScript Handbook: <https://www.typescriptlang.org/docs>
- TypeScript Playground: <https://www.typescriptlang.org/play>
- Choosing Compiler Options: <https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html>
- Zod documentation: <https://zod.dev>
- Curated list of TypeScript resources: <https://github.com/dzharii/awesome-typescript>
