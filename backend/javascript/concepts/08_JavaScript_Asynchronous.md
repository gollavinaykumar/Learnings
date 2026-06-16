# JavaScript Asynchronous JavaScript – Interview Revision Guide 🚀

⚠️ **This is one of the MOST IMPORTANT JavaScript interview topics.**

Almost every JavaScript, React, Node.js, Frontend, and Full Stack interview will ask questions related to:

- Callbacks
- setTimeout()
- Promises
- Async/Await
- Promise.all()
- Promise.race()
- Error Handling

If Scope + Closures explain how JavaScript works,

👉 **Async JavaScript explains how JavaScript handles tasks that take time.**

---

# Table of Contents

1. Synchronous vs Asynchronous
2. Callbacks
3. setTimeout()
4. Callback Hell
5. Promises
6. Promise States
7. Promise Chaining
8. fetch()
9. Async/Await
10. Async Error Handling
11. Promise.all()
12. Promise.race()
13. Common Interview Questions
14. Interview Quick Revision

---

# 1. Synchronous vs Asynchronous

Before learning Promises, understand this first.

---

## Synchronous Code

Runs line by line.

Each line waits for the previous line.

Example:

```js
console.log("Start");

console.log("Middle");

console.log("End");
```

Output:

```js
Start
Middle
End
```

---

### Visualization

```text
Start
 ↓
Middle
 ↓
End
```

One task at a time.

---

## Asynchronous Code

Some tasks take time:

- API Calls
- Database Calls
- Timers
- File Reading

JavaScript doesn't wait.

It continues executing other code.

---

Example:

```js
console.log("Start");

setTimeout(() => {
  console.log("Timer");
}, 2000);

console.log("End");
```

Output:

```js
Start
End
Timer
```

---

### Why?

JavaScript schedules:

```js
setTimeout()
```

for later.

Meanwhile:

```js
console.log("End");
```

runs immediately.

---

# 2. Callbacks

A callback is a function passed as an argument to another function.

---

## Example

```js
function greet(name) {
  console.log("Hello " + name);
}

function processUser(callback) {
  callback("Vinay");
}

processUser(greet);
```

Output:

```js
Hello Vinay
```

---

# Async Callback Example

```js
setTimeout(() => {
  console.log("Executed");
}, 1000);
```

---

Here:

```js
() => {
  console.log("Executed");
}
```

is a callback.

---

# Why Use Callbacks?

Used when code should run later.

Examples:

- Timers
- Events
- API Responses

---

# 3. setTimeout()

Most common async function.

---

## Syntax

```js
setTimeout(callback, delay);
```

---

Example:

```js
setTimeout(() => {
  console.log("Hello");
}, 2000);
```

Output after 2 seconds:

```js
Hello
```

---

# Important Interview Question

```js
console.log("Start");

setTimeout(() => {
  console.log("Timer");
}, 0);

console.log("End");
```

Output:

```js
Start
End
Timer
```

---

Why?

Even:

```js
0 ms
```

does NOT mean immediate execution.

The callback still waits for the call stack to become empty.

---

# 4. Callback Hell

Problem with deeply nested callbacks.

---

Example:

```js
getUser(function(user) {

  getOrders(user.id, function(orders) {

    getPayment(orders.id, function(payment) {

      console.log(payment);

    });

  });

});
```

---

Visualization:

```text
Callback
  ↓
Callback
  ↓
Callback
  ↓
Callback
```

---

Problems:

❌ Hard to read

❌ Hard to debug

❌ Hard to maintain

---

This led to:

```text
Promises
```

---

# 5. Promises

A Promise represents a value that may be available:

```text
Now
Later
Never
```

---

Think of a Promise as:

```text
"I promise to give you a result later."
```

---

## Creating a Promise

```js
const promise =
  new Promise((resolve, reject) => {

    resolve("Success");

  });
```

---

Consuming Promise

```js
promise.then(result => {
  console.log(result);
});
```

Output:

```js
Success
```

---

# Real Example

```js
const promise =
  new Promise((resolve, reject) => {

    setTimeout(() => {
      resolve("Data Loaded");
    }, 2000);

  });
```

---

Output after 2 seconds:

```js
Data Loaded
```

---

# 6. Promise States

Every Promise has a state.

---

## Pending

Initial state.

```text
Still waiting
```

---

## Fulfilled

Success.

```text
Operation completed
```

---

## Rejected

Failure.

```text
Something went wrong
```

---

### Visualization

```text
Pending
   │
   ├── Fulfilled
   │
   └── Rejected
```

---

# Example

```js
const promise =
  new Promise((resolve) => {

    resolve("Done");

  });
```

State:

```text
Pending → Fulfilled
```

---

# Reject Example

```js
const promise =
  new Promise((resolve, reject) => {

    reject("Error");

  });
```

State:

```text
Pending → Rejected
```

---

# 7. Promise Chaining

One of the most common interview topics.

---

Instead of nested callbacks:

Use:

```js
.then()
```

---

Example

```js
fetchUser()
  .then(user => {
    return fetchOrders(user.id);
  })
  .then(orders => {
    return fetchPayment(orders.id);
  })
  .then(payment => {
    console.log(payment);
  });
```

---

Visualization

```text
User
 ↓
Orders
 ↓
Payment
```

---

Each `.then()` receives the result from the previous `.then()`.

---

# Important Rule

Always return values from `.then()`.

---

Example

```js
Promise.resolve(5)
  .then(num => num * 2)
  .then(result => {
    console.log(result);
  });
```

Output:

```js
10
```

---

# 8. fetch()

Most used API method in frontend interviews.

---

## Basic Example

```js
fetch("https://api.example.com/users")
  .then(response => response.json())
  .then(data => {
    console.log(data);
  });
```

---

### Flow

```text
Request
 ↓
Response
 ↓
JSON
 ↓
Data
```

---

# Why response.json()?

Response is not actual JSON.

Need conversion:

```js
response.json()
```

---

# 9. Async/Await

Modern way to work with Promises.

Cleaner and easier to read.

---

## Without Async/Await

```js
fetchData()
  .then(data => {
    console.log(data);
  });
```

---

## With Async/Await

```js
async function getData() {

  const data =
    await fetchData();

  console.log(data);

}
```

---

# async Keyword

Makes a function return a Promise.

Example:

```js
async function hello() {
  return "Hello";
}
```

Actually returns:

```js
Promise.resolve("Hello");
```

---

# await Keyword

Pauses execution until Promise resolves.

---

Example

```js
async function getData() {

  const data =
    await fetchData();

  console.log(data);

}
```

---

Visualization

```text
Wait
 ↓
Promise Resolves
 ↓
Continue Execution
```

---

# Real API Example

```js
async function getUsers() {

  const response =
    await fetch(url);

  const users =
    await response.json();

  console.log(users);

}
```

---

# 10. Async Error Handling

Most asked Async/Await interview question.

---

## Using try/catch

```js
async function getData() {

  try {

    const response =
      await fetch(url);

    const data =
      await response.json();

    console.log(data);

  } catch(error) {

    console.log(error);

  }

}
```

---

# Why?

Promises can fail.

Example:

```text
No Internet
Server Error
Network Error
```

---

Without:

```js
try/catch
```

application may crash.

---

# Promise Error Handling

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

# 11. Promise.all()

Runs multiple promises simultaneously.

---

## Example

```js
Promise.all([
  fetchUsers(),
  fetchOrders(),
  fetchProducts()
])
.then(results => {
  console.log(results);
});
```

---

### Visualization

```text
Users
Orders
Products

Run Together
```

---

Output:

```js
[
  users,
  orders,
  products
]
```

---

# Important Rule

If ANY promise fails:

```js
Promise.all()
```

fails immediately.

---

Example:

```text
Promise 1 ✅
Promise 2 ❌
Promise 3 ✅
```

Result:

```text
Rejected
```

---

# 12. Promise.race()

Returns the first settled promise.

---

Example

```js
Promise.race([
  fetchUsers(),
  fetchOrders()
])
.then(result => {
  console.log(result);
});
```

---

### Visualization

```text
Users
     \
      \
       First Wins
      /
Orders
```

---

Whoever finishes first wins.

---

# Example

```js
Promise.race([
  new Promise(resolve =>
    setTimeout(
      () => resolve("Fast"),
      1000
    )
  ),

  new Promise(resolve =>
    setTimeout(
      () => resolve("Slow"),
      3000
    )
  )
]);
```

Output:

```js
Fast
```

---

# Promise.all vs Promise.race

| Promise.all | Promise.race |
|-------------|-------------|
| Waits for all | First settles wins |
| Returns array | Returns single result |
| Fails if one fails | Returns first settled |

---

# Common Interview Questions

---

## Q1: What is Asynchronous JavaScript?

Code that doesn't block execution while waiting for tasks to finish.

---

## Q2: What is a Callback?

A function passed as an argument to another function.

---

## Q3: What is Callback Hell?

Deeply nested callbacks that are difficult to maintain.

---

## Q4: What is a Promise?

An object representing the eventual result of an asynchronous operation.

---

## Q5: Promise States?

```text
Pending
Fulfilled
Rejected
```

---

## Q6: Difference Between .then() and await?

### .then()

```js
fetchData().then(...)
```

---

### await

```js
const data =
  await fetchData();
```

Cleaner syntax.

---

## Q7: What Does async Do?

Makes a function return a Promise.

---

## Q8: What Does await Do?

Waits for a Promise to settle before continuing.

---

## Q9: What is Promise.all()?

Runs multiple promises in parallel and waits for all.

---

## Q10: What is Promise.race()?

Returns the first settled promise.

---

# Interview Quick Revision 🚀

## Callback

```js
setTimeout(() => {}, 1000);
```

Function passed to another function.

---

## Promise

```js
new Promise(
  (resolve, reject) => {}
);
```

---

## Promise States

```text
Pending
Fulfilled
Rejected
```

---

## Promise Chaining

```js
fetchData()
  .then(...)
  .then(...)
  .catch(...);
```

---

## Async Function

```js
async function getData() {}
```

Returns Promise.

---

## Await

```js
const data =
  await fetchData();
```

Waits for Promise.

---

## Error Handling

```js
try {
  await fetchData();
}
catch(error) {
  console.log(error);
}
```

---

## Promise.all()

```js
Promise.all([
  p1,
  p2,
  p3
]);
```

Waits for all.

---

## Promise.race()

```js
Promise.race([
  p1,
  p2
]);
```

First settles wins.

---

# Most Important Interview Diagram ⭐

```text
Callback
    ↓
Promises
    ↓
Async/Await
```

Evolution of Async JavaScript.

---

### Promise Lifecycle

```text
Pending
   │
   ├── Fulfilled
   │
   └── Rejected
```

---

### Async/Await Flow

```text
async function
       ↓
await Promise
       ↓
Result
```

---

# Golden Interview Tips ⭐

1. Master Promises and Async/Await before learning React.
2. Understand Promise states thoroughly.
3. Practice converting `.then()` chains into Async/Await.
4. Always use `try/catch` with Async/Await.
5. Learn the difference between `Promise.all()` and `Promise.race()`.
6. Know why Callback Hell is a problem.
7. Remember:
   ```text
   async → returns Promise
   await → waits for Promise
   ```
8. Be able to explain the output of `setTimeout(..., 0)`.
9. Most API calls today use:
   ```js
   fetch + async/await
   ```
10. If you master Async JavaScript, you'll be ready for React, Node.js, and Full Stack interview questions.

🚀 Async JavaScript is one of the highest-priority topics for modern JavaScript interviews.