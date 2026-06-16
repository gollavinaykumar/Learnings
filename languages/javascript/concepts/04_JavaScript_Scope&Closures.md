# JavaScript Scope & Closures – Interview Revision Guide 🚀

⚠️ **This is one of the MOST IMPORTANT JavaScript interview topics.**

Many interview questions directly or indirectly test your understanding of:

- Scope
- Lexical Scope
- Closures
- Variable Lookup
- Memory Behavior

If you understand this chapter well, you can answer many advanced JavaScript questions.

---

# Table of Contents

1. What is Scope?
2. Global Scope
3. Function Scope
4. Block Scope
5. Scope Chain
6. Lexical Scope
7. Closures
8. Closure Internals
9. Real-World Closure Examples
10. Common Interview Questions
11. Interview Quick Revision

---

# 1. What is Scope?

Scope determines:

> "Where a variable can be accessed in your code."

Think of scope as the visibility area of a variable.

Example:

```js
let name = "Vinay";

console.log(name);
```

Output:

```js
Vinay
```

The variable is accessible because it is inside its scope.

---

# Why Do We Need Scope?

Without scope:

```js
let user = "John";
let user = "David";
```

Variables would conflict with each other.

Scope helps:

✅ Organize code

✅ Prevent conflicts

✅ Protect variables

---

# 2. Global Scope

Variables declared outside all functions and blocks belong to the global scope.

Example:

```js
let name = "Vinay";

function greet() {
  console.log(name);
}

greet();
```

Output:

```js
Vinay
```

---

## Why?

Because `name` is global.

Everyone can access it.

---

### Visualization

```text
Global Scope
│
├── name
│
└── greet()
```

---

## Another Example

```js
const appName = "Netflix";

function showApp() {
  console.log(appName);
}

showApp();
```

Output:

```js
Netflix
```

---

## Global Variables Can Be Accessed Everywhere

```js
let city = "Hyderabad";

function one() {
  console.log(city);
}

function two() {
  console.log(city);
}
```

Both functions can access:

```js
city
```

---

# Problem With Global Variables

Too many global variables can create bugs.

Example:

```js
let count = 0;
```

Any function can modify it.

```js
count++;
count--;
count = 100;
```

Hard to track changes.

---

# Interview Rule

✅ Minimize global variables.

---

# 3. Function Scope

Variables declared inside a function exist only inside that function.

Example:

```js
function test() {
  let age = 25;

  console.log(age);
}

test();
```

Output:

```js
25
```

---

Outside:

```js
console.log(age);
```

Output:

```js
ReferenceError
```

---

### Visualization

```text
test()
│
└── age
```

Outside cannot access it.

---

# Function Scope Example

```js
function greet() {
  let message = "Hello";
}

console.log(message);
```

Output:

```js
ReferenceError
```

Because:

```js
message
```

exists only inside the function.

---

# 4. Block Scope

A block is code inside:

```js
{
}
```

Examples:

```js
if ()
for ()
while ()
switch ()
```

---

## let and const Are Block Scoped

```js
{
  let name = "Vinay";
}

console.log(name);
```

Output:

```js
ReferenceError
```

---

### Example

```js
if (true) {
  const age = 25;

  console.log(age);
}
```

Output:

```js
25
```

Outside:

```js
console.log(age);
```

Output:

```js
ReferenceError
```

---

# var Is NOT Block Scoped

Important interview question.

Example:

```js
if (true) {
  var name = "Vinay";
}

console.log(name);
```

Output:

```js
Vinay
```

---

Why?

Because:

```js
var
```

is function scoped, not block scoped.

---

# Quick Comparison

| Keyword | Block Scoped |
|----------|-------------|
| var | ❌ No |
| let | ✅ Yes |
| const | ✅ Yes |

---

# 5. Scope Chain

JavaScript searches variables in a specific order.

This is called the Scope Chain.

---

Example:

```js
let country = "India";

function outer() {

  function inner() {
    console.log(country);
  }

  inner();
}

outer();
```

Output:

```js
India
```

---

## Variable Lookup Process

JavaScript searches:

```text
inner()
   ↓
outer()
   ↓
Global Scope
```

Found:

```js
country
```

---

### Visualization

```text
Global Scope
│
└── outer()
      │
      └── inner()
```

Inner can access everything above it.

---

# Important Rule

Child scope can access parent variables.

Parent scope CANNOT access child variables.

---

Example:

```js
function outer() {

  let secret = "ABC";

  function inner() {
    console.log(secret);
  }

}
```

Works.

---

But:

```js
function outer() {

  function inner() {
    let secret = "ABC";
  }

  console.log(secret);
}
```

Output:

```js
ReferenceError
```

---

# 6. Lexical Scope

One of the most important interview topics.

---

## Definition

Lexical Scope means:

> A function can access variables from the scope where it was CREATED.

Not where it is called.

---

Example:

```js
let language = "JavaScript";

function outer() {

  function inner() {
    console.log(language);
  }

  inner();
}

outer();
```

Output:

```js
JavaScript
```

---

Why?

When `inner()` was created, it remembered its surrounding scope.

This is lexical scope.

---

## Key Rule

JavaScript determines scope during code writing (creation time), not execution time.

---

### Example

```js
let x = 10;

function first() {
  console.log(x);
}

function second() {
  let x = 20;

  first();
}

second();
```

Output:

```js
10
```

---

Many people expect:

```js
20
```

But answer is:

```js
10
```

Because `first()` was created in the global scope.

---

### Interview Trick Question

```js
let name = "Global";

function showName() {
  console.log(name);
}

function execute() {
  let name = "Local";

  showName();
}

execute();
```

Output:

```js
Global
```

Reason:

Lexical Scope.

---

# 7. Closures

Most frequently asked JavaScript interview topic.

---

## Definition

A Closure is created when:

> An inner function remembers variables from its outer function even after the outer function has finished execution.

---

Example:

```js
function outer() {

  let count = 0;

  function inner() {
    count++;
    console.log(count);
  }

  return inner;
}

const counter = outer();

counter();
counter();
counter();
```

Output:

```js
1
2
3
```

---

# Why Does This Work?

Normally:

```js
outer()
```

finishes execution.

Its variables should disappear.

But:

```js
inner()
```

still remembers:

```js
count
```

This memory is called a Closure.

---

# Closure Visualization

```text
outer()
│
├── count = 0
│
└── returns inner()
```

Even after outer ends:

```text
inner()
  ↑
 remembers count
```

---

# 8. Closure Internals

This is how interviewers often expect you to explain closures.

---

When JavaScript creates:

```js
function inner() {}
```

It stores:

```text
Function Code
+
Reference to Outer Scope
```

---

Example:

```js
function outer() {

  let count = 0;

  return function inner() {
    count++;
    return count;
  };
}
```

---

JavaScript internally keeps:

```text
inner()
│
├── Function Code
│
└── [[Environment]]
        │
        └── count
```

---

This hidden reference is why closures work.

---

# Important Interview Statement

A closure is NOT a copy of variables.

A closure stores a reference to the variables.

---

Example:

```js
function outer() {

  let count = 0;

  return function() {
    count++;
    console.log(count);
  };
}
```

Every call updates the SAME variable.

---

# 9. Real-World Closure Examples

---

## Counter

```js
function createCounter() {

  let count = 0;

  return function() {
    return ++count;
  };
}

const counter = createCounter();

console.log(counter());
console.log(counter());
console.log(counter());
```

Output:

```js
1
2
3
```

---

## Data Privacy

```js
function createAccount() {

  let balance = 1000;

  return {
    getBalance() {
      return balance;
    }
  };
}
```

Outside cannot access:

```js
balance
```

directly.

---

## Event Handlers

```js
button.addEventListener(
  "click",
  function() {
    console.log("Clicked");
  }
);
```

Closures are heavily used behind the scenes.

---

## React Hooks

```js
useState()
```

internally relies on closure concepts.

---

# Common Interview Questions

---

## Q1: What is Scope?

Scope determines where variables can be accessed.

---

## Q2: What is Global Scope?

Variables declared outside all functions and blocks.

---

## Q3: What is Block Scope?

Variables declared with:

```js
let
const
```

exist only inside:

```js
{
}
```

---

## Q4: What is Lexical Scope?

Functions can access variables from where they were defined, not where they are called.

---

## Q5: What is a Closure?

A closure allows an inner function to remember variables from its outer function even after the outer function has finished executing.

---

## Q6: Why Are Closures Useful?

Used for:

- Data privacy
- Counters
- Event handlers
- Callbacks
- React Hooks
- State management

---

## Q7: Difference Between Scope and Closure?

### Scope

Determines accessibility of variables.

### Closure

Allows a function to remember variables from outer scope after execution.

---

# Interview Quick Revision 🚀

## Global Scope

```js
let name = "Vinay";
```

Accessible everywhere.

---

## Function Scope

```js
function test() {
  let age = 25;
}
```

Accessible only inside function.

---

## Block Scope

```js
{
  let x = 10;
}
```

Accessible only inside block.

---

## Scope Chain

```text
Current Scope
     ↓
Parent Scope
     ↓
Global Scope
```

---

## Lexical Scope

Function uses variables from where it was created.

---

## Closure

```js
function outer() {

  let count = 0;

  return function() {
    count++;
    return count;
  };
}
```

Remembers outer variables.

---

## var vs let

```js
var  -> Function Scoped

let  -> Block Scoped

const -> Block Scoped
```

---

# Golden Interview Tips ⭐

1. Scope and Closures are among the top 5 most asked JavaScript interview topics.
2. Always remember: **Lexical Scope is the foundation of Closures.**
3. Child scope can access parent variables, but parent cannot access child variables.
4. `var` is function scoped; `let` and `const` are block scoped.
5. A closure stores a reference to outer variables, not a copy.
6. Practice counter examples and lexical scope trick questions.
7. If asked "What is a Closure?" explain both the definition and the counter example.

🎯 **Interview Formula:**

```text
Lexical Scope
        +
Remembering Outer Variables
        =
Closure
```

If you can clearly explain that formula with an example, you'll answer most closure interview questions confidently.