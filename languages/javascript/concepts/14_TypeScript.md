# TypeScript – Interview Revision Guide 🚀

⚠️ **TypeScript is one of the MOST IMPORTANT skills in modern frontend and full-stack development.**

Today, most production applications are built using:

- React + TypeScript
- Angular + TypeScript
- Next.js + TypeScript
- NestJS + TypeScript

Many companies now expect TypeScript knowledge even for JavaScript roles.

---

# Table of Contents

1. What is TypeScript?
2. Why TypeScript?
3. Static Typing
4. Basic Types
5. Type Inference
6. Interfaces
7. Type Aliases
8. Generics
9. Utility Types
10. Decorators
11. Common Interview Questions
12. Interview Quick Revision

---

# 1. What is TypeScript?

TypeScript is a superset of JavaScript.

Meaning:

```text
JavaScript
     +
Type System
     =
TypeScript
```

---

TypeScript adds:

✅ Static Typing

✅ Interfaces

✅ Generics

✅ Better IDE Support

✅ Compile-Time Error Checking

---

# Example

JavaScript:

```js
let age = 25;

age = "Hello";
```

Valid JavaScript.

---

TypeScript:

```ts
let age: number = 25;

age = "Hello";
```

Output:

```text
Type Error
```

---

# Why?

Because:

```ts
age
```

must always be:

```ts
number
```

---

# 2. Why TypeScript?

Imagine a project with:

```text
500 files
50 developers
100,000+ lines
```

Without TypeScript:

```text
More Bugs
Hard Debugging
Runtime Errors
```

---

With TypeScript:

```text
Early Error Detection
Better Refactoring
Safer Code
```

---

# 3. Static Typing

Most important TypeScript concept.

---

## What is Static Typing?

Type checking before code runs.

---

### JavaScript

```js
function add(a, b) {
  return a + b;
}

add("5", 10);
```

Output:

```js
510
```

Unexpected.

---

### TypeScript

```ts
function add(
  a: number,
  b: number
) {
  return a + b;
}

add("5", 10);
```

Output:

```text
Compile Error
```

---

# Benefit

Catch errors before production.

---

# Basic Types

---

## string

```ts
let name: string = "Vinay";
```

---

## number

```ts
let age: number = 25;
```

---

## boolean

```ts
let isAdmin: boolean = true;
```

---

## array

```ts
let nums: number[] =
  [1, 2, 3];
```

---

## object

```ts
let user: object = {
  name: "Vinay"
};
```

---

## any

```ts
let value: any = 10;
```

---

Can hold anything.

Avoid when possible.

---

## unknown

Safer version of:

```ts
any
```

---

Example

```ts
let value: unknown;
```

Need checking before usage.

---

# 4. Type Inference

TypeScript often detects types automatically.

---

Example

```ts
let age = 25;
```

TypeScript infers:

```ts
number
```

---

Equivalent to:

```ts
let age: number = 25;
```

---

Example

```ts
let name = "Vinay";
```

Automatically:

```ts
string
```

---

# Interview Tip

Prefer inference when type is obvious.

---

# 5. Interfaces

⚠️ One of the MOST ASKED TypeScript interview topics.

---

## What is an Interface?

An interface defines the structure of an object.

---

Example

```ts
interface User {

  name: string;

  age: number;

}
```

---

Usage

```ts
const user: User = {

  name: "Vinay",

  age: 25

};
```

---

# Error Example

```ts
const user: User = {

  name: "Vinay"

};
```

Output:

```text
Property age is missing
```

---

# Optional Properties

```ts
interface User {

  name: string;

  age?: number;

}
```

---

Now:

```ts
const user: User = {
  name: "Vinay"
};
```

Valid.

---

# Readonly Properties

```ts
interface User {

  readonly id: number;

}
```

---

Cannot modify later.

---

```ts
user.id = 5;
```

Output:

```text
Error
```

---

# Interface for Functions

```ts
interface Add {

  (
    a: number,
    b: number
  ): number;

}
```

---

Usage

```ts
const add: Add =
  (a, b) => a + b;
```

---

# Interface Extension

```ts
interface Person {

  name: string;

}
```

---

```ts
interface Employee
extends Person {

  salary: number;

}
```

---

Now Employee has:

```ts
name
salary
```

---

# 6. Type Aliases

Alternative to interfaces.

---

Example

```ts
type User = {

  name: string;

  age: number;

};
```

---

Usage

```ts
const user: User = {
  name: "Vinay",
  age: 25
};
```

---

# Interface vs Type

| Interface | Type |
|------------|------|
| Object structures | Any type |
| Extendable | Flexible |
| Most common | Powerful |

---

# Interview Rule

Use:

```ts
interface
```

for objects.

---

# 7. Generics

⚠️ One of the MOST IMPORTANT TypeScript interview topics.

---

## Problem

Without generics:

```ts
function getValue(
  value: any
) {
  return value;
}
```

---

Lose type safety.

---

# Generic Solution

```ts
function getValue<T>(
  value: T
): T {

  return value;

}
```

---

Usage

```ts
getValue<string>(
  "Hello"
);
```

---

Output type:

```ts
string
```

---

# Another Example

```ts
getValue<number>(
  100
);
```

Output type:

```ts
number
```

---

# Generic Array

```ts
const nums:
Array<number> =
[1, 2, 3];
```

---

Equivalent

```ts
const nums:
number[] =
[1, 2, 3];
```

---

# Generic Interface

```ts
interface ApiResponse<T> {

  data: T;

}
```

---

Usage

```ts
const response:
ApiResponse<string> = {

  data: "Success"

};
```

---

# Why Generics?

Reusable code while keeping type safety.

---

# 8. Utility Types

⚠️ Frequently asked in interviews.

TypeScript provides built-in helper types.

---

# Partial<T>

Makes all properties optional.

---

Example

```ts
interface User {

  name: string;

  age: number;

}
```

---

```ts
type UpdateUser =
Partial<User>;
```

---

Result

```ts
{
  name?: string;
  age?: number;
}
```

---

# Required<T>

Makes all properties required.

---

```ts
type UserRequired =
Required<User>;
```

---

# Readonly<T>

Makes properties immutable.

---

```ts
type ReadonlyUser =
Readonly<User>;
```

---

Cannot modify fields.

---

# Pick<T>

Select specific properties.

---

```ts
type UserPreview =
Pick<
  User,
  "name"
>;
```

---

Result

```ts
{
  name: string;
}
```

---

# Omit<T>

Remove properties.

---

```ts
type UserWithoutAge =
Omit<
  User,
  "age"
>;
```

---

Result

```ts
{
  name: string;
}
```

---

# Record<K, T>

Create object types.

---

Example

```ts
type Users =
Record<
  string,
  number
>;
```

---

Result

```ts
{
  john: 1,
  mike: 2
}
```

---

# Common Utility Types

```ts
Partial
Required
Readonly
Pick
Omit
Record
```

---

# 9. Decorators

⚠️ Advanced TypeScript topic.

Common in:

```text
Angular
NestJS
```

---

## What is a Decorator?

A special function that adds behavior to classes, methods, or properties.

---

Example

```ts
@Controller()
class UserController {

}
```

---

Here:

```ts
@Controller
```

is a decorator.

---

# Class Decorator

```ts
function Logger(
  constructor: Function
) {

  console.log(
    "Class Created"
  );

}
```

---

Usage

```ts
@Logger

class User {

}
```

---

Output

```text
Class Created
```

---

# Method Decorator Example

```ts
class User {

  @Log

  getData() {}

}
```

---

Used heavily in:

```text
Angular
NestJS
```

---

# Why Decorators?

Used for:

```text
Logging
Validation
Authentication
Metadata
Dependency Injection
```

---

# Common Interview Questions

---

## Q1: What is TypeScript?

A superset of JavaScript that adds static typing.

---

## Q2: Why Use TypeScript?

- Type Safety
- Better IDE Support
- Early Error Detection

---

## Q3: What is Static Typing?

Type checking before execution.

---

## Q4: What is an Interface?

Defines object structure.

```ts
interface User {}
```

---

## Q5: What is a Generic?

Reusable type-safe code.

```ts
function test<T>() {}
```

---

## Q6: What is Partial<T>?

Makes all properties optional.

---

## Q7: Difference Between Interface and Type?

Interface is mainly for object shapes.

Type is more flexible.

---

## Q8: What Are Decorators?

Functions that add metadata or behavior to classes, methods, or properties.

---

# Interview Quick Revision 🚀

## Static Typing

```ts
let age: number = 25;
```

---

## Interface

```ts
interface User {

  name: string;

}
```

---

## Type Alias

```ts
type User = {

  name: string;

};
```

---

## Generic

```ts
function getValue<T>(
  value: T
) {
  return value;
}
```

---

## Utility Types

```ts
Partial<T>

Required<T>

Readonly<T>

Pick<T>

Omit<T>

Record<K, T>
```

---

## Decorator

```ts
@Controller()

class User {}
```

---

# Most Important Interview Diagram ⭐

### TypeScript

```text
JavaScript
     +
Static Typing
     +
Interfaces
     +
Generics
     =
TypeScript
```

---

### Generic Flow

```text
Input Type
      ↓
Generic T
      ↓
Output Same Type
```

---

### Utility Types

```text
User
 ↓
Partial<User>

Readonly<User>

Pick<User>

Omit<User>
```

---

# Golden Interview Tips ⭐

1. TypeScript is JavaScript + Types.
2. Interfaces are heavily used in React and APIs.
3. Generics are one of the most asked TypeScript interview topics.
4. Learn:
   ```ts
   Partial
   Readonly
   Pick
   Omit
   ```
5. Prefer interfaces for object shapes.
6. Avoid using `any` whenever possible.
7. Use `unknown` instead of `any` when type is uncertain.
8. Decorators are common in Angular and NestJS.
9. Most enterprise applications use TypeScript.
10. If you master Interfaces + Generics + Utility Types, you'll answer most TypeScript interview questions confidently.

🚀 TypeScript is considered a core skill for modern JavaScript developers and is heavily used in large-scale production applications.