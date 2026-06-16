# JavaScript Event Loop Internals – Interview Revision Guide 🚀

⚠️ **This is one of the MOST IMPORTANT JavaScript interview topics.**

Many companies ask Event Loop questions because it tests whether you truly understand:

- How JavaScript executes code
- Why async code works
- Promises vs setTimeout
- Execution order questions

If you understand the Event Loop, you can explain:

✅ Async/Await

✅ Promises

✅ setTimeout

✅ API Calls

✅ JavaScript Runtime

---

# Table of Contents

1. JavaScript is Single-Threaded
2. What is the Call Stack?
3. What is the Web API Environment?
4. What is the Callback Queue?
5. What are Microtasks?
6. What are Macrotasks?
7. What is the Event Loop?
8. Execution Order
9. Promise vs setTimeout
10. Event Loop Flow
11. Common Interview Questions
12. Interview Quick Revision

---

# 1. JavaScript is Single-Threaded

Most important interview statement.

JavaScript has:

```text
One Call Stack
```

Meaning:

```text
One task at a time
```

---

Example:

```js
console.log("A");

console.log("B");

console.log("C");
```

Output:

```js
A
B
C
```

---

### Visualization

```text
A
 ↓
B
 ↓
C
```

---

JavaScript executes code sequentially.

---

# Question

If JavaScript can do only one thing at a time...

How does this work?

```js
setTimeout(() => {
  console.log("Timer");
}, 1000);
```

while the page remains responsive?

---

Answer:

```text
Event Loop
```

---

# 2. What is the Call Stack?

The Call Stack is where JavaScript executes functions.

Think of it as a stack of plates.

---

### Example

```js
function first() {
  second();
}

function second() {
  third();
}

function third() {
  console.log("Hello");
}

first();
```

---

### Stack Execution

```text
Call Stack

first()
```

↓

```text
first()
second()
```

↓

```text
first()
second()
third()
```

↓

Execute:

```js
console.log("Hello")
```

↓

Remove:

```text
third()
```

↓

Remove:

```text
second()
```

↓

Remove:

```text
first()
```

↓

Stack Empty

---

# Visualization

```text
third()
--------
second()
--------
first()
```

Last In

First Out

(LIFO)

---

# Important Rule

JavaScript can execute only one stack frame at a time.

---

# 3. What is the Web API Environment?

JavaScript itself does NOT handle:

```text
setTimeout()
DOM Events
fetch()
```

---

The browser provides these features.

They are called:

```text
Web APIs
```

---

Examples:

```js
setTimeout()

fetch()

addEventListener()
```

---

# Example

```js
setTimeout(() => {
  console.log("Timer");
}, 2000);
```

---

What happens?

Step 1

```js
setTimeout()
```

enters Call Stack.

---

Step 2

Browser handles timer.

---

Step 3

Call Stack becomes free.

---

Step 4

After timer finishes:

Callback moves to queue.

---

# 4. What is the Callback Queue?

Also called:

```text
Task Queue
```

or

```text
Macrotask Queue
```

---

Stores callbacks waiting to execute.

Example:

```js
setTimeout(() => {
  console.log("Hello");
}, 1000);
```

After timer finishes:

```js
() => {
  console.log("Hello");
}
```

moves into:

```text
Callback Queue
```

---

Visualization

```text
Callback Queue

Timer Callback
```

Waiting for:

```text
Call Stack Empty
```

---

# 5. What are Microtasks?

Most important interview topic after Promises.

---

Microtasks have HIGHER priority than callbacks.

---

Examples:

```js
Promise.then()

catch()

finally()

queueMicrotask()
```

---

Example

```js
Promise.resolve()
  .then(() => {
    console.log("Promise");
  });
```

---

This callback enters:

```text
Microtask Queue
```

NOT Callback Queue.

---

# Important Rule

Before executing any callback queue task:

JavaScript empties ALL microtasks.

---

### Priority

```text
Microtasks
     ↑
Higher Priority

Macrotasks
     ↓
Lower Priority
```

---

# 6. What are Macrotasks?

Macrotasks are normal async tasks.

Examples:

```js
setTimeout()

setInterval()

DOM Events
```

---

Example

```js
setTimeout(() => {
  console.log("Timer");
}, 0);
```

---

Moves to:

```text
Macrotask Queue
```

---

# Microtask vs Macrotask

| Microtask | Macrotask |
|------------|------------|
| Promise.then() | setTimeout() |
| catch() | setInterval() |
| finally() | DOM Events |
| queueMicrotask() | Message Events |

---

# Interview Rule

Microtasks ALWAYS execute before Macrotasks.

---

# 7. What is the Event Loop?

The Event Loop is the coordinator.

Its job:

```text
Check Call Stack
Check Queues
Move Tasks
```

---

### Simple Definition

> Event Loop continuously checks whether the Call Stack is empty and moves pending tasks into it.

---

### Visualization

```text
Call Stack
     ↑
     │
Event Loop
     │
     ↓

Queues
```

---

# Event Loop Algorithm

Step 1

Execute synchronous code.

---

Step 2

Stack becomes empty.

---

Step 3

Execute ALL microtasks.

---

Step 4

Execute ONE macrotask.

---

Step 5

Repeat forever.

---

# Event Loop Flow

```text
Sync Code
    ↓
Microtasks
    ↓
Macrotask
    ↓
Microtasks
    ↓
Macrotask
    ↓
Repeat
```

---

# 8. Execution Order Examples

Most interview questions are output-based.

---

## Example 1

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

```js
A
C
B
```

---

Why?

```text
A → Sync
C → Sync
B → Callback Queue
```

---

# Example 2

```js
console.log("A");

Promise.resolve()
  .then(() => {
    console.log("B");
  });

console.log("C");
```

Output:

```js
A
C
B
```

---

Why?

Promise callback becomes Microtask.

Runs after sync code.

---

# Example 3 (VERY IMPORTANT)

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("C");
  });

console.log("D");
```

Output:

```js
A
D
C
B
```

---

### Why?

Step 1

Sync code:

```js
A
D
```

---

Step 2

Microtasks:

```js
C
```

---

Step 3

Macrotasks:

```js
B
```

---

# Interview Rule

```text
Sync
 ↓
Microtasks
 ↓
Macrotasks
```

Always.

---

# Example 4

```js
setTimeout(() => {
  console.log("Timeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("Promise");
  });
```

Output:

```js
Promise
Timeout
```

---

Because:

```text
Microtask > Macrotask
```

---

# 9. Promise vs setTimeout

Very common interview question.

---

Example

```js
setTimeout(() => {
  console.log("Timeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("Promise");
  });
```

Output:

```js
Promise
Timeout
```

---

Why?

Promise callbacks go to:

```text
Microtask Queue
```

---

Timeout goes to:

```text
Macrotask Queue
```

---

Microtasks run first.

---

# 10. Event Loop Full Flow

```text
┌─────────────┐
│ Call Stack  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Event Loop  │
└──────┬──────┘
       │
       ▼

Microtask Queue
(Promises)

       ▼

Macrotask Queue
(setTimeout)
```

---

# Full Execution Priority

```text
1. Synchronous Code

2. Microtasks
   - Promise.then
   - catch
   - finally

3. Macrotasks
   - setTimeout
   - setInterval
   - Events
```

---

# Common Interview Questions

---

## Q1: Is JavaScript Single-Threaded?

Yes.

JavaScript has one Call Stack.

---

## Q2: Then How Is It Asynchronous?

Because:

```text
Web APIs
+
Event Loop
+
Queues
```

handle async work.

---

## Q3: What is the Call Stack?

A structure where JavaScript executes functions.

---

## Q4: What is the Event Loop?

A mechanism that moves tasks from queues to the Call Stack when it becomes empty.

---

## Q5: What is the Callback Queue?

Stores callbacks waiting to execute.

---

## Q6: What is a Microtask?

High-priority async task.

Examples:

```js
Promise.then()
```

---

## Q7: What is a Macrotask?

Regular async task.

Examples:

```js
setTimeout()
```

---

## Q8: Which Runs First?

```js
Promise.then()
```

or

```js
setTimeout()
```

Answer:

```js
Promise.then()
```

Because Microtask > Macrotask.

---

# Most Asked Interview Question

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("3");
  });

console.log("4");
```

Output:

```js
1
4
3
2
```

---

Explanation:

```text
1 → Sync
4 → Sync
3 → Microtask
2 → Macrotask
```

---

# Interview Quick Revision 🚀

## Call Stack

Executes functions.

---

## Callback Queue

Stores:

```js
setTimeout()
```

callbacks.

---

## Microtask Queue

Stores:

```js
Promise.then()
catch()
finally()
```

---

## Macrotask Queue

Stores:

```js
setTimeout()
setInterval()
Events
```

---

## Event Loop

Moves tasks into Call Stack when it becomes empty.

---

## Priority

```text
Sync
 ↓
Microtasks
 ↓
Macrotasks
```

---

## Promise vs Timeout

```js
Promise.then()
```

always runs before:

```js
setTimeout()
```

---

# Most Important Interview Diagram ⭐

```text
JavaScript Runtime

        ┌─────────────┐
        │ Call Stack  │
        └──────┬──────┘
               │
               ▼

         Event Loop

       ↙         ↘

Microtasks     Macrotasks

(Promises)    (setTimeout)
```

---

# Golden Interview Tips ⭐

1. JavaScript is single-threaded.
2. Only one function executes at a time on the Call Stack.
3. Async behavior comes from Web APIs + Event Loop.
4. Promise callbacks are Microtasks.
5. setTimeout callbacks are Macrotasks.
6. Microtasks always execute before Macrotasks.
7. Memorize:
   ```text
   Sync
    ↓
   Microtasks
    ↓
   Macrotasks
   ```
8. Most interview output questions are solved using that rule.
9. If confused, draw:
   ```text
   Call Stack
   Event Loop
   Microtask Queue
   Macrotask Queue
   ```
10. Mastering the Event Loop makes Promises, Async/Await, and JavaScript runtime behavior much easier to understand.

🚀 Event Loop is one of the highest-value JavaScript interview topics. Understanding it separates intermediate developers from advanced developers.