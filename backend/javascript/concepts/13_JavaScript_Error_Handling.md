# JavaScript Error Handling – Interview Revision Guide 🚀

⚠️ **Error Handling is a very important topic in JavaScript, React, Node.js, and Full Stack interviews.**

Applications can fail because of:

- Invalid input
- API failures
- Network issues
- Database errors
- Programming mistakes

Error Handling helps applications fail gracefully instead of crashing.

---

# Table of Contents

1. What is Error Handling?
2. Types of Errors
3. try/catch
4. throw Statement
5. Custom Errors
6. Error Object
7. finally Block
8. Async Error Handling
9. Promise Error Handling
10. Unhandled Promise Rejection
11. Common Interview Questions
12. Interview Quick Revision

---

# 1. What is Error Handling?

Error Handling means:

> Detecting and managing errors so the application can continue working safely.

---

Example

```js
const user = null;

console.log(user.name);
```

Output:

```text
TypeError:
Cannot read properties of null
```

---

Without error handling:

```text
Application crashes
```

---

With error handling:

```text
Show user-friendly message
Log error
Continue execution
```

---

# 2. Types of Errors

Common JavaScript errors.

---

## SyntaxError

Invalid JavaScript syntax.

```js
if (true {
  console.log("Hello");
}
```

Output:

```text
SyntaxError
```

---

## ReferenceError

Using variable that doesn't exist.

```js
console.log(name);
```

Output:

```text
ReferenceError
```

---

## TypeError

Using value incorrectly.

```js
const user = null;

user.name;
```

Output:

```text
TypeError
```

---

## RangeError

Value outside allowed range.

```js
const arr =
  new Array(-1);
```

Output:

```text
RangeError
```

---

# 3. try/catch

Most common error handling mechanism.

---

## Syntax

```js
try {

  // risky code

}
catch(error) {

  // handle error

}
```

---

# Example

```js
try {

  const user = null;

  console.log(user.name);

}
catch(error) {

  console.log(
    "Something went wrong"
  );

}
```

Output:

```text
Something went wrong
```

---

# Without try/catch

```js
const user = null;

console.log(user.name);
```

Application stops.

---

# With try/catch

```js
try {

  console.log(user.name);

}
catch(error) {

  console.log("Handled");

}
```

Application continues.

---

# Execution Flow

```text
try
 ↓
Error?
 ↓
catch
```

---

# Access Error Object

```js
try {

  const user = null;

  console.log(user.name);

}
catch(error) {

  console.log(error);

}
```

---

Output

```text
TypeError...
```

---

# Useful Properties

```js
error.name
```

---

```js
error.message
```

---

Example

```js
catch(error) {

  console.log(
    error.name
  );

  console.log(
    error.message
  );

}
```

---

# 4. throw Statement

Used to manually create errors.

---

## Syntax

```js
throw new Error(
  "Message"
);
```

---

# Example

```js
function divide(a, b) {

  if (b === 0) {

    throw new Error(
      "Cannot divide by zero"
    );

  }

  return a / b;

}
```

---

Usage

```js
try {

  divide(10, 0);

}
catch(error) {

  console.log(
    error.message
  );

}
```

Output:

```text
Cannot divide by zero
```

---

# Why throw?

Used for validation.

---

Example

```js
function login(user) {

  if (!user) {

    throw new Error(
      "User required"
    );

  }

}
```

---

# 5. Custom Errors

Very common in large applications.

---

Instead of generic:

```js
Error
```

create specific errors.

---

# Custom Error Class

```js
class ValidationError
  extends Error {

  constructor(message) {

    super(message);

    this.name =
      "ValidationError";

  }

}
```

---

# Throw Custom Error

```js
throw new ValidationError(
  "Email is required"
);
```

---

# Handle Custom Error

```js
try {

  throw new ValidationError(
    "Email required"
  );

}
catch(error) {

  console.log(
    error.name
  );

}
```

Output:

```text
ValidationError
```

---

# Why Custom Errors?

More meaningful.

---

Example

```text
ValidationError
AuthError
DatabaseError
NetworkError
```

---

Instead of:

```text
Error
```

---

# 6. Error Object

Every JavaScript error is an object.

---

Example

```js
try {

  throw new Error(
    "Something wrong"
  );

}
catch(error) {

  console.log(error);

}
```

---

Properties

---

## error.name

```js
console.log(
  error.name
);
```

Output:

```text
Error
```

---

## error.message

```js
console.log(
  error.message
);
```

Output:

```text
Something wrong
```

---

## error.stack

Shows execution trace.

```js
console.log(
  error.stack
);
```

---

Useful for debugging.

---

# 7. finally Block

Runs whether error occurs or not.

---

## Syntax

```js
try {

}
catch(error) {

}
finally {

}
```

---

# Example

```js
try {

  console.log("Try");

}
catch(error) {

  console.log("Catch");

}
finally {

  console.log("Finally");

}
```

Output:

```text
Try
Finally
```

---

# Example With Error

```js
try {

  throw new Error();

}
catch(error) {

  console.log("Catch");

}
finally {

  console.log("Finally");

}
```

Output:

```text
Catch
Finally
```

---

# Common Use Cases

```text
Close Database Connection
Stop Loader
Release Resources
Cleanup
```

---

# 8. Async Error Handling

⚠️ Very important interview topic.

---

# Problem

```js
async function getData() {

  const response =
    await fetch(url);

}
```

---

What if:

```text
Network Fails?
```

---

Solution:

```js
try/catch
```

---

# Example

```js
async function getData() {

  try {

    const response =
      await fetch(url);

    const data =
      await response.json();

    console.log(data);

  }
  catch(error) {

    console.log(
      error.message
    );

  }

}
```

---

# Interview Rule

Use:

```js
try/catch
```

with:

```js
async/await
```

---

# 9. Promise Error Handling

Promises use:

```js
.catch()
```

---

Example

```js
fetchData()
  .then(data => {

    console.log(data);

  })
  .catch(error => {

    console.log(error);

  });
```

---

# Promise Chain Example

```js
fetchUser()
  .then(user => {

    return fetchOrders(
      user.id
    );

  })
  .then(orders => {

    console.log(orders);

  })
  .catch(error => {

    console.log(error);

  });
```

---

# Important Rule

Errors automatically travel down the chain.

---

Example

```js
Promise.resolve()
  .then(() => {

    throw new Error(
      "Boom"
    );

  })
  .catch(error => {

    console.log(
      error.message
    );

  });
```

Output:

```text
Boom
```

---

# 10. Unhandled Promise Rejection

⚠️ Frequently asked interview topic.

---

# Example

```js
Promise.reject(
  "Something failed"
);
```

---

Output:

```text
Unhandled Promise Rejection
```

---

Why?

Because no:

```js
.catch()
```

exists.

---

# Another Example

```js
async function test() {

  throw new Error(
    "Failed"
  );

}

test();
```

---

Output:

```text
Unhandled Promise Rejection
```

---

Because returned Promise was not handled.

---

# Correct Version

```js
test()
  .catch(error => {

    console.log(
      error.message
    );

  });
```

---

# Why Dangerous?

Can cause:

```text
Application crashes
Silent failures
Unexpected behavior
```

---

# Best Practice

Always handle:

```js
Promise
```

using:

```js
.catch()
```

or

```js
try/catch
```

---

# Common Interview Questions

---

## Q1: What is Error Handling?

Managing errors so applications don't crash.

---

## Q2: What does try/catch do?

Handles runtime errors safely.

---

## Q3: What is throw?

Creates a custom error manually.

---

Example

```js
throw new Error(
  "Invalid input"
);
```

---

## Q4: Why Use Custom Errors?

To create meaningful error types.

Example:

```text
ValidationError
AuthError
NetworkError
```

---

## Q5: What is finally?

Runs regardless of success or failure.

---

## Q6: How Do You Handle Async Errors?

```js
try/catch
```

with:

```js
await
```

---

## Q7: How Do You Handle Promise Errors?

```js
.catch()
```

---

## Q8: What is Unhandled Promise Rejection?

A rejected Promise with no error handler.

---

# Interview Quick Revision 🚀

## try/catch

```js
try {

}
catch(error) {

}
```

---

## throw

```js
throw new Error(
  "Something wrong"
);
```

---

## Custom Error

```js
class ValidationError
  extends Error {}
```

---

## Error Properties

```js
error.name

error.message

error.stack
```

---

## finally

```js
finally {

}
```

Always executes.

---

## Async Error Handling

```js
try {

  await fetch();

}
catch(error) {

}
```

---

## Promise Error Handling

```js
promise
  .catch(error => {});
```

---

## Unhandled Promise Rejection

```js
Promise.reject();
```

without:

```js
.catch()
```

---

# Most Important Interview Diagram ⭐

### Synchronous Error Handling

```text
try
 ↓
Error?
 ↓
catch
 ↓
finally
```

---

### Promise Error Flow

```text
Promise
   ↓
then()
   ↓
then()
   ↓
catch()
```

---

### Async/Await Error Flow

```text
async
  ↓
await
  ↓
try/catch
```

---

# Golden Interview Tips ⭐

1. Use `try/catch` for synchronous and async code.
2. Use `throw` to create custom validation errors.
3. Create custom error classes in large applications.
4. Learn `error.name`, `error.message`, and `error.stack`.
5. Always handle Promise rejections.
6. Use `.catch()` for Promise chains.
7. Use `try/catch` with Async/Await.
8. Never ignore rejected Promises.
9. Remember:
   ```text
   throw → create error
   catch → handle error
   finally → cleanup
   ```
10. Error Handling is one of the core skills expected in production JavaScript applications.

🚀 Master Error Handling and you'll write safer, more reliable JavaScript applications.