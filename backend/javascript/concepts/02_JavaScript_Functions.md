# JavaScript Functions – Interview Revision Guide 🚀

Functions are reusable blocks of code that perform a specific task.

Instead of writing the same code again and again, we write it once inside a function and call it whenever needed.

---

# Table of Contents

1. What is a Function?
2. Function Declaration
3. Function Expression
4. Arrow Functions
5. Parameters vs Arguments
6. Return Keyword
7. Callbacks
8. Higher-Order Functions
9. Closures
10. Recursion
11. Interview Quick Revision

---

# 1. What is a Function?

A function is a block of code that performs a task when called.

### Syntax

```js
function functionName() {
  // code
}
```

Example:

```js
function greet() {
  console.log("Hello");
}

greet();
```

Output:

```js
Hello
```

---

# Why Use Functions?

Without functions:

```js
console.log("Welcome");
console.log("Welcome");
console.log("Welcome");
```

With function:

```js
function welcome() {
  console.log("Welcome");
}

welcome();
welcome();
welcome();
```

Benefits:

✅ Reusable

✅ Cleaner code

✅ Easier maintenance

---

# 2. Function Declaration

Also called Function Statement.

This is the most common way to create a function.

### Syntax

```js
function add(a, b) {
  return a + b;
}
```

Example:

```js
function add(a, b) {
  return a + b;
}

console.log(add(5, 3));
```

Output:

```js
8
```

---

## Function Hoisting

Function declarations are hoisted.

Meaning:

You can call them before they are defined.

```js
sayHello();

function sayHello() {
  console.log("Hello");
}
```

Output:

```js
Hello
```

---

# 3. Function Expression

A function stored inside a variable.

### Syntax

```js
const add = function(a, b) {
  return a + b;
};
```

Example:

```js
const greet = function() {
  console.log("Hello");
};

greet();
```

Output:

```js
Hello
```

---

## Function Expressions Are NOT Hoisted

```js
sayHello();

const sayHello = function() {
  console.log("Hello");
};
```

Output:

```js
Error
```

Because the variable is not initialized yet.

---

# Function Declaration vs Function Expression

| Feature | Declaration | Expression |
|----------|------------|------------|
| Hoisted | ✅ Yes | ❌ No |
| Stored in Variable | ❌ No | ✅ Yes |
| Common Usage | ✅ Very Common | ✅ Common |

---

# 4. Arrow Functions

Introduced in ES6.

Shorter syntax for writing functions.

### Traditional Function

```js
function add(a, b) {
  return a + b;
}
```

---

### Arrow Function

```js
const add = (a, b) => {
  return a + b;
};
```

---

## Single Line Arrow Function

```js
const add = (a, b) => a + b;
```

Output:

```js
add(2, 3);

5
```

---

## One Parameter

```js
const square = num => num * num;
```

Output:

```js
square(4);

16
```

---

## No Parameters

```js
const greet = () => {
  console.log("Hello");
};
```

---

# When to Use Arrow Functions?

Mostly used for:

- callbacks
- array methods
- short functions

Example:

```js
const numbers = [1, 2, 3];

numbers.forEach(num => {
  console.log(num);
});
```

---

# 5. Parameters vs Arguments

This is a common interview question.

---

## Parameters

Variables defined in the function.

```js
function add(a, b) {
}
```

Here:

```js
a, b
```

are Parameters.

---

## Arguments

Actual values passed to the function.

```js
add(5, 10);
```

Here:

```js
5, 10
```

are Arguments.

---

# 6. Return Keyword

`return` sends a value back from a function.

Example:

```js
function add(a, b) {
  return a + b;
}

let result = add(5, 3);

console.log(result);
```

Output:

```js
8
```

---

## Without Return

```js
function add(a, b) {
  a + b;
}

console.log(add(5, 3));
```

Output:

```js
undefined
```

---

# 7. Callbacks

A callback is a function passed as an argument to another function.

---

## Example

```js
function greet(name) {
  console.log("Hello " + name);
}

function processUser(callback) {
  callback("John");
}

processUser(greet);
```

Output:

```js
Hello John
```

---

## Real World Example

```js
setTimeout(() => {
  console.log("Executed after 2 seconds");
}, 2000);
```

Explanation:

- Arrow function is passed as an argument.
- `setTimeout()` executes it later.

---

## Why Callbacks?

Used for:

- asynchronous operations
- event handling
- API calls
- timers

---

# 8. Higher-Order Functions (HOF)

A Higher-Order Function is a function that:

### Either

Accepts another function as an argument

OR

Returns a function

---

## Example 1

Function accepting another function.

```js
function greet(name) {
  console.log("Hello " + name);
}

function processUser(callback) {
  callback("John");
}

processUser(greet);
```

Here:

```js
processUser()
```

is a Higher-Order Function.

---

## Example 2

Array Methods

```js
const nums = [1, 2, 3];

nums.forEach(num => {
  console.log(num);
});
```

`forEach()` is a Higher-Order Function.

---

## Example 3

Returning a Function

```js
function multiplyBy(x) {
  return function(y) {
    return x * y;
  };
}

const double = multiplyBy(2);

console.log(double(5));
```

Output:

```js
10
```

---

# Common Higher-Order Functions

```js
forEach()
map()
filter()
reduce()
find()
sort()
```

Interviewers love these.

---

# 9. Closures

One of the MOST IMPORTANT JavaScript interview topics.

---

## Definition

A closure happens when an inner function remembers variables from its outer function even after the outer function has finished executing.

---

## Example

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

## What's Happening?

Step 1:

```js
outer()
```

creates:

```js
count = 0
```

Step 2:

```js
return inner
```

returns the inner function.

Step 3:

Even though `outer()` has finished,

`inner()` still remembers `count`.

This memory is called a Closure.

---

## Real World Example

Data Privacy

```js
function createBankAccount() {
  let balance = 1000;

  return {
    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount();

console.log(account.getBalance());
```

Output:

```js
1000
```

Notice:

```js
balance
```

cannot be accessed directly.

Closure protects it.

---

# Why Closures Matter?

Used in:

- data privacy
- counters
- caching
- event handlers
- React hooks

---

# 10. Recursion

A function calling itself.

---

## Basic Example

```js
function countdown(n) {
  if (n === 0) {
    return;
  }

  console.log(n);

  countdown(n - 1);
}

countdown(5);
```

Output:

```js
5
4
3
2
1
```

---

# Base Case

Every recursive function must have a stopping condition.

```js
if (n === 0)
```

This is called the Base Case.

Without it:

```js
function test() {
  test();
}
```

Output:

```js
Maximum call stack size exceeded
```

---

## Factorial Example

Factorial:

```js
5! = 5 × 4 × 3 × 2 × 1
```

---

### Recursive Solution

```js
function factorial(n) {
  if (n === 1) {
    return 1;
  }

  return n * factorial(n - 1);
}

console.log(factorial(5));
```

Output:

```js
120
```

---

# Recursion Flow

```js
factorial(5)

5 * factorial(4)

5 * 4 * factorial(3)

5 * 4 * 3 * factorial(2)

5 * 4 * 3 * 2 * factorial(1)

5 * 4 * 3 * 2 * 1

120
```

---

# Interview Questions

### Q1: Difference Between Function Declaration and Expression?

| Function Declaration | Function Expression |
|---------------------|---------------------|
| Hoisted | Not Hoisted |
| Defined with function keyword | Stored in variable |

---

### Q2: What is a Callback?

A function passed as an argument to another function.

```js
setTimeout(() => {
  console.log("Hello");
}, 1000);
```

---

### Q3: What is a Higher-Order Function?

A function that:

- accepts a function
- returns a function

Example:

```js
map()
filter()
reduce()
forEach()
```

---

### Q4: What is a Closure?

A closure allows an inner function to remember variables from its outer function even after the outer function finishes execution.

---

### Q5: What is Recursion?

A function calling itself until a base condition is met.

---

# Interview Quick Revision 🚀

### Function Declaration

```js
function add(a, b) {
  return a + b;
}
```

---

### Function Expression

```js
const add = function(a, b) {
  return a + b;
};
```

---

### Arrow Function

```js
const add = (a, b) => a + b;
```

---

### Callback

```js
setTimeout(() => {
  console.log("Hello");
}, 1000);
```

---

### Higher-Order Function

```js
arr.map()
arr.filter()
arr.reduce()
arr.forEach()
```

---

### Closure

```js
function outer() {
  let count = 0;

  return function() {
    count++;
    console.log(count);
  };
}
```

Remembers outer variables.

---

### Recursion

```js
function test(n) {
  if (n === 0) return;

  test(n - 1);
}
```

Function calling itself.

---

# Golden Interview Tips ⭐

1. Know the difference between Declaration, Expression, and Arrow Functions.
2. Understand callbacks thoroughly.
3. Learn Higher-Order Functions because they are used everywhere.
4. Practice Closure questions—they are frequently asked in interviews.
5. Always remember that recursion needs a Base Case.
6. Be able to explain closures with the counter example.
7. Be comfortable writing functions without looking at notes.

Functions are the heart of JavaScript. Master these topics before moving to Objects, Arrays, DOM, and Async JavaScript.