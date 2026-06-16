# JavaScript Fundamentals – Interview Revision Guide 🚀

This README covers the most important JavaScript fundamentals for interviews with simple explanations and easy examples.

---

# Table of Contents

1. Variables (`var`, `let`, `const`)
2. Data Types
3. Type Coercion
4. Operators
5. Truthy vs Falsy Values
6. Strict Equality (`===`)
7. Template Literals
8. If / Else
9. Switch Statement
10. Loops
11. Break and Continue

---

# 1. Variables (var, let, const)

Variables are used to store data.

## var

Old way of declaring variables.

```js
var name = "John";
```

### Features

- Can be redeclared.
- Can be reassigned.
- Function scoped.
- Avoid using in modern JavaScript.

```js
var age = 20;
var age = 25; // Allowed

console.log(age); // 25
```

---

## let

Used when the value may change later.

```js
let score = 10;
score = 20;
```

### Features

- Cannot be redeclared in the same scope.
- Can be reassigned.
- Block scoped.

```js
let city = "Delhi";

city = "Mumbai";

console.log(city); // Mumbai
```

---

## const

Used when the value should not change.

```js
const PI = 3.14;
```

### Features

- Cannot be redeclared.
- Cannot be reassigned.
- Block scoped.

```js
const PI = 3.14;

// PI = 4; ❌ Error
```

### Important

Objects and arrays declared with `const` can still be modified.

```js
const user = {
  name: "John"
};

user.name = "David";

console.log(user.name); // David
```

---

# Quick Comparison

| Feature | var | let | const |
|----------|------|------|------|
| Redeclare | ✅ | ❌ | ❌ |
| Reassign | ✅ | ✅ | ❌ |
| Block Scoped | ❌ | ✅ | ✅ |
| Modern Usage | ❌ | ✅ | ✅ |

---

# 2. Data Types

Data types tell JavaScript what kind of value is stored.

---

## Primitive Data Types

### String

Text data.

```js
let name = "John";
```

```js
console.log(typeof name);
// string
```

---

### Number

Integers and decimal numbers.

```js
let age = 25;
let price = 99.99;
```

```js
console.log(typeof age);
// number
```

---

### Boolean

Represents true or false.

```js
let isLoggedIn = true;
```

```js
console.log(typeof isLoggedIn);
// boolean
```

---

### Undefined

Variable declared but no value assigned.

```js
let x;

console.log(x);
// undefined
```

---

### Null

Intentional empty value.

```js
let user = null;
```

Example:

```js
let selectedUser = null;
```

Meaning:
"No user selected yet."

---

### BigInt

Used for very large numbers.

```js
let big = 12345678901234567890n;
```

---

### Symbol

Creates unique identifiers.

```js
let id = Symbol("id");
```

---

## Non-Primitive Data Types

### Object

Stores key-value pairs.

```js
const person = {
  name: "John",
  age: 25
};
```

---

### Array

Stores multiple values.

```js
const colors = ["red", "blue", "green"];
```

---

### Function

Reusable block of code.

```js
function greet() {
  return "Hello";
}
```

---

# 3. Type Coercion

JavaScript automatically converts one data type into another when needed.

This is called Type Coercion.

---

## Example 1

```js
"5" + 2
```

Output:

```js
"52"
```

Why?

Number 2 becomes string "2".

---

## Example 2

```js
"5" - 2
```

Output:

```js
3
```

Why?

String "5" becomes number 5.

---

## Example 3

```js
true + 1
```

Output:

```js
2
```

Because:

```js
true = 1
```

---

## Explicit Conversion

Convert manually.

### String to Number

```js
Number("100");
```

Output:

```js
100
```

---

### Number to String

```js
String(100);
```

Output:

```js
"100"
```

---

### Value to Boolean

```js
Boolean(1);
```

Output:

```js
true
```

---

# 4. Operators

Operators perform operations on values.

---

## Arithmetic Operators

| Operator | Meaning |
|-----------|-----------|
| + | Addition |
| - | Subtraction |
| * | Multiplication |
| / | Division |
| % | Modulus |
| ** | Power |

Example:

```js
10 + 5; // 15
10 - 5; // 5
10 * 5; // 50
10 / 5; // 2
10 % 3; // 1
2 ** 3; // 8
```

---

## Assignment Operators

```js
let x = 10;

x += 5;
```

Equivalent to:

```js
x = x + 5;
```

Output:

```js
15
```

---

## Comparison Operators

| Operator | Meaning |
|-----------|-----------|
| == | Loose equality |
| === | Strict equality |
| != | Not equal |
| !== | Strict not equal |
| > | Greater than |
| < | Less than |
| >= | Greater than or equal |
| <= | Less than or equal |

Example:

```js
10 > 5; // true
10 < 5; // false
```

---

## Logical Operators

### AND (&&)

Both conditions must be true.

```js
true && true
```

Output:

```js
true
```

---

### OR (||)

At least one condition must be true.

```js
true || false
```

Output:

```js
true
```

---

### NOT (!)

Reverses the value.

```js
!true
```

Output:

```js
false
```

---

# 5. Truthy vs Falsy Values

In JavaScript every value is either Truthy or Falsy.

---

## Falsy Values (Only 8)

```js
false
0
-0
0n
""
null
undefined
NaN
```

Example:

```js
if ("") {
  console.log("Runs");
} else {
  console.log("Does not run");
}
```

Output:

```js
Does not run
```

---

## Truthy Values

Everything else is truthy.

Examples:

```js
"hello"
[]
{}
42
true
```

Example:

```js
if ("hello") {
  console.log("Runs");
}
```

Output:

```js
Runs
```

---

# 6. Strict Equality (===)

Most common interview question.

---

## Loose Equality (==)

Checks value only.

```js
5 == "5"
```

Output:

```js
true
```

Because JavaScript converts types automatically.

---

## Strict Equality (===)

Checks value AND data type.

```js
5 === "5"
```

Output:

```js
false
```

Because:

```js
number !== string
```

---

## Interview Rule

Always prefer:

```js
===
```

instead of:

```js
==
```

---

# 7. Template Literals

Used to create strings easily.

Uses backticks:

```js
`
`
```

---

## Old Way

```js
let name = "John";

console.log("Hello " + name);
```

---

## Template Literal

```js
let name = "John";

console.log(`Hello ${name}`);
```

Output:

```js
Hello John
```

---

## Multiple Variables

```js
let name = "John";
let age = 25;

console.log(`My name is ${name} and I am ${age} years old.`);
```

---

# 8. If / Else

Used for decision making.

---

## Simple If

```js
let age = 20;

if (age >= 18) {
  console.log("Adult");
}
```

---

## If Else

```js
let age = 16;

if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}
```

Output:

```js
Minor
```

---

## Else If

```js
let marks = 85;

if (marks >= 90) {
  console.log("A");
}
else if (marks >= 75) {
  console.log("B");
}
else {
  console.log("C");
}
```

Output:

```js
B
```

---

# 9. Switch Statement

Alternative to multiple if-else statements.

```js
let day = 2;

switch(day) {
  case 1:
    console.log("Monday");
    break;

  case 2:
    console.log("Tuesday");
    break;

  default:
    console.log("Invalid Day");
}
```

Output:

```js
Tuesday
```

---

# 10. Loops

Loops repeat code multiple times.

---

## For Loop

Most common loop.

```js
for (let i = 1; i <= 5; i++) {
  console.log(i);
}
```

Output:

```js
1
2
3
4
5
```

---

## While Loop

Runs while condition is true.

```js
let i = 1;

while (i <= 5) {
  console.log(i);
  i++;
}
```

---

## Do While Loop

Runs at least once.

```js
let i = 1;

do {
  console.log(i);
  i++;
}
while (i <= 5);
```

---

# 11. Break and Continue

Used inside loops.

---

## break

Stops the loop immediately.

```js
for (let i = 1; i <= 5; i++) {

  if (i === 3) {
    break;
  }

  console.log(i);
}
```

Output:

```js
1
2
```

---

## continue

Skips current iteration.

```js
for (let i = 1; i <= 5; i++) {

  if (i === 3) {
    continue;
  }

  console.log(i);
}
```

Output:

```js
1
2
4
5
```

---

# Interview Quick Revision

### Variables

```js
var  -> old
let  -> can change
const -> cannot reassign
```

---

### Data Types

```js
String
Number
Boolean
Undefined
Null
BigInt
Symbol
Object
Array
Function
```

---

### Equality

```js
5 == "5"   // true

5 === "5"  // false
```

Use:

```js
===
```

---

### Falsy Values

```js
false
0
-0
0n
""
null
undefined
NaN
```

---

### Template Literal

```js
`Hello ${name}`
```

---

### Loops

```js
for
while
do while
```

---

### Loop Control

```js
break     -> stop loop

continue  -> skip current iteration
```

---

# Golden Interview Tips ⭐

1. Prefer `let` and `const` over `var`.
2. Use `===` instead of `==`.
3. Know all falsy values.
4. Understand type coercion clearly.
5. Be comfortable writing `if`, `switch`, and loops without looking at notes.
6. Explain concepts with simple examples during interviews.
7. Practice writing code by hand because many interviews test fundamentals first.

Happy Learning! 🚀