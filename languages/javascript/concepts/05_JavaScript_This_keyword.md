# JavaScript `this` Keyword – Interview Revision Guide 🚀

⚠️ **The `this` keyword is one of the most confusing and most asked JavaScript interview topics.**

Many developers know JavaScript syntax but struggle to explain:

- What `this` means
- How `this` changes
- Arrow function behavior
- `call()`, `apply()`, `bind()`

Understanding `this` properly can solve many advanced interview questions.

---

# Table of Contents

1. What is `this`?
2. Global Context
3. Function Context
4. Object Methods
5. Arrow Function Behavior
6. `call()`
7. `apply()`
8. `bind()`
9. call vs apply vs bind
10. Common Interview Questions
11. Interview Quick Revision

---

# 1. What is `this`?

`this` is a special keyword.

It refers to **the object that is currently executing the function.**

The most important thing to remember:

> **`this` is determined by HOW a function is called, not where it is written.**

---

Example:

```js
const user = {
  name: "Vinay",

  greet() {
    console.log(this.name);
  }
};

user.greet();
```

Output:

```js
Vinay
```

Here:

```js
this === user
```

---

# Golden Rule

When you see `this`, ask:

```text
Who called this function?
```

That answer usually tells you what `this` is.

---

# 2. Global Context

Global context means code running outside any function.

---

## In Browser

```js
console.log(this);
```

Output:

```js
window
```

Because:

```js
this === window
```

in browser global scope.

---

### Example

```js
var name = "Vinay";

console.log(this.name);
```

Output:

```js
Vinay
```

Because:

```js
window.name
```

exists.

---

## In Node.js

```js
console.log(this);
```

Output:

```js
{}
```

(or module object)

Not `window`.

---

# Interview Point

Global `this` behaves differently in:

- Browser
- Node.js

---

# 3. Function Context

Regular functions have their own `this`.

---

### Example

```js
function show() {
  console.log(this);
}

show();
```

---

## Browser (Non-Strict Mode)

Output:

```js
window
```

---

## Strict Mode

```js
"use strict";

function show() {
  console.log(this);
}

show();
```

Output:

```js
undefined
```

---

# Interview Question

What is `this` inside a normal function?

Answer:

```text
Depends on how the function is called.
```

---

# 4. Object Methods

Most common usage of `this`.

---

### Example

```js
const user = {
  name: "Vinay",

  greet() {
    console.log(this.name);
  }
};

user.greet();
```

Output:

```js
Vinay
```

---

Here:

```js
this = user
```

because:

```js
user.greet()
```

called the function.

---

# Another Example

```js
const car = {
  brand: "BMW",

  showBrand() {
    console.log(this.brand);
  }
};

car.showBrand();
```

Output:

```js
BMW
```

---

# Important Interview Example

```js
const user = {
  name: "Vinay",

  greet() {
    console.log(this.name);
  }
};

const fn = user.greet;

fn();
```

Output:

```js
undefined
```

(or window.name)

---

Why?

Because:

```js
fn()
```

is called as a normal function.

Not:

```js
user.greet()
```

---

So:

```js
this ≠ user
```

---

# Key Rule

```js
obj.method()
```

Then:

```js
this = obj
```

---

# 5. Arrow Function Behavior

⚠️ Most important interview topic after closures.

---

## Arrow Functions Do NOT Have Their Own `this`

Normal functions create their own `this`.

Arrow functions do not.

Instead:

```text
Arrow functions inherit `this`
from their surrounding scope.
```

This is called:

```text
Lexical this
```

---

### Example

```js
const user = {
  name: "Vinay",

  greet: () => {
    console.log(this.name);
  }
};

user.greet();
```

Output:

```js
undefined
```

---

Why?

Arrow function doesn't use:

```js
user
```

as `this`.

It inherits from outer scope.

---

# Regular Function vs Arrow Function

### Regular Function

```js
const user = {
  name: "Vinay",

  greet() {
    console.log(this.name);
  }
};
```

Output:

```js
Vinay
```

---

### Arrow Function

```js
const user = {
  name: "Vinay",

  greet: () => {
    console.log(this.name);
  }
};
```

Output:

```js
undefined
```

---

# Real-World Example

Regular function problem:

```js
const user = {
  name: "Vinay",

  greet() {
    setTimeout(function() {
      console.log(this.name);
    }, 1000);
  }
};

user.greet();
```

Output:

```js
undefined
```

---

Because inside:

```js
setTimeout()
```

`this` changes.

---

# Solution Using Arrow Function

```js
const user = {
  name: "Vinay",

  greet() {
    setTimeout(() => {
      console.log(this.name);
    }, 1000);
  }
};

user.greet();
```

Output:

```js
Vinay
```

---

Why?

Arrow function inherits:

```js
this
```

from:

```js
greet()
```

---

# Interview Rule

Use arrow functions when you want to preserve the surrounding `this`.

---

# 6. call()

`call()` lets you manually set `this`.

---

### Syntax

```js
fn.call(thisValue, arg1, arg2)
```

---

### Example

```js
function greet(city) {
  console.log(
    this.name,
    city
  );
}

const user = {
  name: "Vinay"
};

greet.call(user, "Hyderabad");
```

Output:

```js
Vinay Hyderabad
```

---

Here:

```js
this = user
```

---

# 7. apply()

Similar to `call()`.

Difference:

Arguments are passed as an array.

---

### Syntax

```js
fn.apply(thisValue, [args])
```

---

### Example

```js
function greet(city) {
  console.log(
    this.name,
    city
  );
}

const user = {
  name: "Vinay"
};

greet.apply(user, ["Hyderabad"]);
```

Output:

```js
Vinay Hyderabad
```

---

# 8. bind()

`bind()` does not execute immediately.

Instead:

It returns a new function.

---

### Syntax

```js
const newFn =
  fn.bind(thisValue);
```

---

### Example

```js
function greet() {
  console.log(this.name);
}

const user = {
  name: "Vinay"
};

const boundFn =
  greet.bind(user);

boundFn();
```

Output:

```js
Vinay
```

---

# Why Use bind()?

Useful when passing methods as callbacks.

---

### Example

```js
const user = {
  name: "Vinay",

  greet() {
    console.log(this.name);
  }
};

setTimeout(
  user.greet.bind(user),
  1000
);
```

Output:

```js
Vinay
```

---

Without bind:

```js
setTimeout(user.greet, 1000);
```

Output:

```js
undefined
```

---

# 9. call vs apply vs bind

| Method | Executes Immediately? | Arguments |
|----------|--------------------|------------|
| call() | ✅ Yes | Separate |
| apply() | ✅ Yes | Array |
| bind() | ❌ No | Returns Function |

---

## call()

```js
fn.call(user, 1, 2);
```

---

## apply()

```js
fn.apply(user, [1, 2]);
```

---

## bind()

```js
const newFn =
  fn.bind(user);

newFn();
```

---

# Memory Trick

### call

```text
Call now
```

---

### apply

```text
Apply array
```

---

### bind

```text
Bind later
```

---

# Common Interview Questions

---

## Q1: What is `this`?

`this` refers to the object currently executing the function.

---

## Q2: How is `this` determined?

By how a function is called.

Not where it is written.

---

## Q3: What is `this` inside an object method?

```js
obj.method()
```

Then:

```js
this = obj
```

---

## Q4: Do Arrow Functions Have Their Own `this`?

❌ No.

They inherit `this` from the surrounding scope.

---

## Q5: Difference Between call, apply, and bind?

### call()

Executes immediately.

```js
fn.call(obj);
```

---

### apply()

Executes immediately.

Arguments passed as array.

```js
fn.apply(obj, []);
```

---

### bind()

Returns new function.

```js
fn.bind(obj);
```

---

# Most Asked Interview Trap

```js
const user = {
  name: "Vinay",

  greet: () => {
    console.log(this.name);
  }
};

user.greet();
```

Output:

```js
undefined
```

Reason:

Arrow functions don't have their own `this`.

---

# Interview Quick Revision 🚀

## Global Context

Browser:

```js
this === window
```

---

## Object Method

```js
user.greet()
```

↓

```js
this === user
```

---

## Arrow Function

```js
() => {}
```

↓

```text
No own this
```

Uses outer `this`.

---

## call()

```js
fn.call(obj, a, b);
```

Execute now.

---

## apply()

```js
fn.apply(obj, [a, b]);
```

Execute now.

Arguments in array.

---

## bind()

```js
const fn2 =
  fn.bind(obj);
```

Returns new function.

Execute later.

---

# Golden Interview Tips ⭐

1. Always remember: **`this` depends on how a function is called.**
2. Arrow functions do not have their own `this`.
3. Arrow functions inherit `this` from surrounding scope (lexical `this`).
4. Use regular methods inside objects, not arrow functions.
5. Remember:
   - `call()` → execute now
   - `apply()` → execute now with array arguments
   - `bind()` → execute later
6. Practice `setTimeout()` examples because interviewers love them.
7. If you understand Closures + Scope + `this`, you've mastered the hardest JavaScript fundamentals.

🎯 **Interview Formula**

```text
Normal Function
      ↓
this depends on caller

Arrow Function
      ↓
this depends on surrounding scope
```

Memorize that rule and most `this` questions become easy.