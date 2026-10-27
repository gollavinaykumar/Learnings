# JavaScript Concurrency & Asynchronous Programming — Basics to Advanced

> **Goal:** Understand how JavaScript handles asynchronous work,
> concurrency, and parallelism—from simple callbacks to workers and
> production system design.
>
> Every topic includes a **simple definition to remember**, a small
> example, a mental model, common mistakes, and interview revision.
>
> **Important:** JavaScript is used in browsers and Node.js. Some APIs
> are environment-specific, so each section notes the difference where
> it matters.

------------------------------------------------------------------------

## Table of Contents

1.  [The Big Picture](#1-the-big-picture)
2.  [Concurrency vs. Parallelism](#2-concurrency-vs-parallelism)
3.  [Synchronous vs. Asynchronous
    Code](#3-synchronous-vs-asynchronous-code)
4.  [Call Stack](#4-call-stack)
5.  [JavaScript Runtime](#5-javascript-runtime)
6.  [The Event Loop](#6-the-event-loop)
7.  [Tasks, Microtasks, and Queues](#7-tasks-microtasks-and-queues)
8.  [Callbacks](#8-callbacks)
9.  [Promises](#9-promises)
10. [Promise Chaining and Error
    Handling](#10-promise-chaining-and-error-handling)
11. [async and await](#11-async-and-await)
12. [Running Work Sequentially
    vs. Concurrently](#12-running-work-sequentially-vs-concurrently)
13. [Promise.all, allSettled, race, and
    any](#13-promiseall-allsettled-race-and-any)
14. [Timers: setTimeout and
    setInterval](#14-timers-settimeout-and-setinterval)
15. [Fetch, Network I/O, and
    AbortController](#15-fetch-network-io-and-abortcontroller)
16. [JavaScript Is Single-Threaded—What Does That
    Mean?](#16-javascript-is-single-threadedwhat-does-that-mean)
17. [Web Workers and Worker Threads](#17-web-workers-and-worker-threads)
18. [SharedArrayBuffer and Atomics](#18-sharedarraybuffer-and-atomics)
19. [Node.js Event Loop](#19-nodejs-event-loop)
20. [Streams and Backpressure](#20-streams-and-backpressure)
21. [Concurrency Limits and Worker
    Pools](#21-concurrency-limits-and-worker-pools)
22. [Race Conditions in JavaScript](#22-race-conditions-in-javascript)
23. [Cancellation, Timeouts, and
    Cleanup](#23-cancellation-timeouts-and-cleanup)
24. [Memory Leaks and Unhandled
    Rejections](#24-memory-leaks-and-unhandled-rejections)
25. [Performance and Responsiveness](#25-performance-and-responsiveness)
26. [Testing and Debugging](#26-testing-and-debugging)
27. [Real Production Scenarios](#27-real-production-scenarios)
28. [Common Mistakes](#28-common-mistakes)
29. [JavaScript vs. Go vs. Java](#29-javascript-vs-go-vs-java)
30. [Interview Questions and Simple
    Answers](#30-interview-questions-and-simple-answers)
31. [Learning Path](#31-learning-path)
32. [One-Minute Revision Cheat
    Sheet](#32-one-minute-revision-cheat-sheet)

------------------------------------------------------------------------

## 1. The Big Picture

### Simple definition to remember

**JavaScript asynchronous programming lets your program start slow work
and continue doing other work instead of waiting doing nothing.**

Examples of slow work: - Fetching data from an API. - Reading a file. -
Waiting for a database. - Waiting for a timer. - Uploading a document.

``` text
Without async:
Start request ───── wait 2 seconds ───── show result ───── next task

With async:
Start request ── continue other work ── result arrives ── handle result
```

JavaScript uses different mechanisms depending on the work: - **Call
stack:** executes JavaScript functions. - **Event loop:** coordinates
when queued callbacks can run. - **Promises and `async/await`:**
represent and organize asynchronous results. - **Web APIs / Node.js
APIs:** perform timers, network, file, and other environment-supported
operations. - **Workers:** run JavaScript on separate threads for
parallel CPU work.

**Remember:** Async does not automatically mean “another thread.” It
means the program does not have to block while waiting for the result.

------------------------------------------------------------------------

## 2. Concurrency vs. Parallelism

### Concurrency

**Simple definition:** Concurrency means multiple tasks can make
progress during the same time period.

### Parallelism

**Simple definition:** Parallelism means multiple tasks actually run at
the same time on different execution resources.

``` text
Concurrency — tasks overlap in time

Task A: [start][wait for API ........][continue]
Task B:       [validate][render][work]

Parallelism — tasks execute at the same time

Thread 1: [ CPU task A ][ CPU task A ]
Thread 2: [ CPU task B ][ CPU task B ]
```

In a typical browser page, one JavaScript thread runs one piece of
JavaScript at a time, but network operations and browser work can happen
while JavaScript is free. Web Workers can run JavaScript in parallel.

In Node.js, asynchronous I/O allows work to overlap; worker threads can
run JavaScript CPU tasks in parallel.

**Remember:** Concurrency = *taking turns / overlapping progress*.
Parallelism = *doing work together at the same instant*.

------------------------------------------------------------------------

## 3. Synchronous vs. Asynchronous Code

### Synchronous

**Simple definition:** Synchronous code runs step by step; the next step
waits for the current step to finish.

``` js
console.log("A");
console.log("B");
console.log("C");

// Output: A, B, C
```

### Asynchronous

**Simple definition:** Asynchronous code lets the program continue and
handle a result later.

``` js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");

// Output:
// A
// C
// B
```

Even with a delay of zero, the timer callback runs later, after the
current synchronous code completes and the runtime schedules the
callback.

### Why it matters

A long synchronous task can freeze a browser UI or block Node.js from
handling other callbacks. Async I/O helps, but a huge CPU loop still
blocks the JavaScript thread.

**Remember:** Sync = *wait here*. Async = *continue now, handle result
later*.

------------------------------------------------------------------------

## 4. Call Stack

### Simple definition to remember

**The call stack is the list of JavaScript functions currently
running.**

When a function is called, it is added to the stack. When it returns, it
is removed.

``` js
function first() {
  second();
}

function second() {
  console.log("Hello");
}

first();
```

``` text
Top
┌───────────────┐
│ second()      │  currently running
├───────────────┤
│ first()       │
├───────────────┤
│ global script │
└───────────────┘
Bottom
```

When `second()` finishes, it is popped off; then `first()` continues and
finishes.

### What is stack overflow?

**Simple definition:** Stack overflow happens when calls keep nesting
too deeply and the call stack runs out of space.

``` js
function repeatForever() {
  repeatForever();
}

// Eventually throws RangeError: Maximum call stack size exceeded
```

**Remember:** Call stack = *what JavaScript is doing right now*.

------------------------------------------------------------------------

## 5. JavaScript Runtime

### Simple definition to remember

**The runtime is the environment that runs JavaScript and provides extra
capabilities such as timers, networking, and file access.**

The JavaScript language itself defines core behavior, while the
environment provides APIs.

``` text
             Your JavaScript code
                     |
                     v
             +---------------+
             |  Call Stack   |
             +---------------+
                     |
        +------------+-------------+
        |                          |
        v                          v
 Browser APIs                 Node.js APIs
 timers, fetch, DOM           files, sockets, timers
        |                          |
        +------------+-------------+
                     |
                     v
                Event loop
                     |
                     v
              Queued callbacks
```

This diagram is conceptual. Browser and Node.js runtimes have different
internal implementations and scheduling details.

### Browser vs. Node.js

| Browser JavaScript      | Node.js JavaScript                                          |
|-------------------------|-------------------------------------------------------------|
| DOM and UI APIs         | File system and server APIs                                 |
| Web Workers             | Worker Threads                                              |
| Browser networking APIs | Node networking APIs                                        |
| Browser event loop      | Node event loop based on libuv plus V8 microtask processing |

**Remember:** JavaScript is the language; the runtime supplies the
environment and APIs.

------------------------------------------------------------------------

## 6. The Event Loop

### Simple definition to remember

**The event loop checks when JavaScript can run queued work after the
current code finishes.**

The call stack cannot execute two JavaScript functions at the same time
on the same thread. The event loop helps schedule callbacks when the
stack is clear.

``` text
        JavaScript runs current code
                    |
                    v
              Call stack clear?
                    |
                    v
         Run eligible queued callback
                    |
                    v
           Continue event processing
```

Example:

``` js
console.log("Start");

setTimeout(() => {
  console.log("Timer callback");
}, 0);

console.log("End");
```

Output:

``` text
Start
End
Timer callback
```

The timer callback cannot interrupt the currently running script.

### Does the event loop run code in parallel?

No. The event loop schedules when callbacks run on a JavaScript thread.
Other runtime services may do work separately, and workers can execute
JavaScript on other threads.

**Remember:** Event loop = *the traffic controller for queued JavaScript
work*.

------------------------------------------------------------------------

## 7. Tasks, Microtasks, and Queues

### Simple definitions

- **Task (often called macrotask):** a scheduled unit of work such as a
  timer callback or event callback.
- **Microtask:** a high-priority queued job, commonly a Promise
  reaction.
- **Queue:** a waiting line of work.
- **Microtask checkpoint:** a point where the runtime processes pending
  microtasks according to its scheduling rules.

In browsers, Promise callbacks run as microtasks. Timer callbacks are
tasks. Exact ordering details vary by environment and host rules, but
the common example below is reliable in modern browsers and Node.js for
this basic case.

``` js
console.log("1");

setTimeout(() => console.log("2 - timer"), 0);

Promise.resolve().then(() => console.log("3 - promise"));

console.log("4");
```

Output:

``` text
1
4
3 - promise
2 - timer
```

Why? 1. The current synchronous script runs first. 2. The Promise
reaction is queued as a microtask. 3. The timer callback runs later as a
task.

### Microtask starvation

**Simple definition:** Microtask starvation happens when code keeps
adding microtasks so other queued work cannot get a chance to run
promptly.

``` js
function keepQueueBusy() {
  queueMicrotask(keepQueueBusy);
}

// Do not run this: it can starve timers and rendering.
```

**Remember:** Sync first, then pending microtasks, then other eligible
event-loop work—while remembering that host scheduling has additional
details.

------------------------------------------------------------------------

## 8. Callbacks

### Simple definition to remember

**A callback is a function you give to other code so it can call your
function later.**

``` js
function greet(name, callback) {
  console.log(`Hello, ${name}`);
  callback();
}

greet("Vinay", () => {
  console.log("Finished greeting");
});
```

Callbacks can be synchronous or asynchronous. The word “callback” alone
does not mean asynchronous.

### Asynchronous callback example

``` js
setTimeout(() => {
  console.log("Runs later");
}, 500);
```

### Callback hell

**Simple definition:** Callback hell is deeply nested callbacks that
make flow and error handling difficult to follow.

``` js
getUser(id, (user) => {
  getOrders(user.id, (orders) => {
    getPayment(orders[0].id, (payment) => {
      console.log(payment);
    });
  });
});
```

Promises and `async/await` often make this flow easier to read.

**Remember:** Callback = *“call my function when you reach this point.”*

------------------------------------------------------------------------

## 9. Promises

### Simple definition to remember

**A Promise is an object representing a result that may be available
now, later, or never because the operation failed.**

A Promise has three states: - **pending:** still waiting; -
**fulfilled:** completed successfully; - **rejected:** failed.

``` text
             pending
             /     \
            v       v
       fulfilled  rejected
       success    failure
```

A Promise settles only once: it becomes fulfilled or rejected.

### Example

``` js
const promise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("Data loaded");
  } else {
    reject(new Error("Request failed"));
  }
});

promise
  .then((value) => console.log(value))
  .catch((error) => console.error(error));
```

### Important distinction

Creating a Promise does not automatically move its executor to a
background thread. The Promise executor runs synchronously when the
Promise is constructed. The asynchronous operation may be provided by an
API, and `.then()` handlers run later as microtasks.

``` js
console.log("A");

const p = new Promise((resolve) => {
  console.log("B"); // runs immediately
  resolve();
});

p.then(() => console.log("D"));

console.log("C");

// A, B, C, D
```

**Remember:** Promise = *a box for a future success value or failure*.

------------------------------------------------------------------------

## 10. Promise Chaining and Error Handling

### Simple definition to remember

**Promise chaining passes a result from one asynchronous step to the
next.**

``` js
fetch("/api/user")
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  })
  .then((user) => {
    console.log(user.name);
  })
  .catch((error) => {
    console.error("Could not load user:", error);
  });
```

Each `.then()` returns a new Promise. Returning a value passes it to the
next `.then()`. Returning a Promise makes the chain wait for that
Promise. Throwing an error rejects the chain.

### Common mistake

``` js
fetch("/api/user").then((response) => {
  response.json(); // forgot return
});
```

The parsed JSON is not passed to the next handler because the result was
not returned.

Correct:

``` js
fetch("/api/user").then((response) => {
  return response.json();
});
```

### Error handling rule

A `.catch()` handles rejections that flow into that point in the chain.
If the catch handler throws or returns a rejected Promise, the resulting
chain is rejected again.

**Remember:** Return the next result; throw to send the chain into error
handling.

------------------------------------------------------------------------

## 11. async and await

### Simple definition to remember

**`async/await` lets you write Promise-based code in a style that reads
like step-by-step code.**

An `async` function always returns a Promise. `await` waits for a
Promise’s result **inside that async function**, without blocking the
whole JavaScript thread.

``` js
async function loadUser() {
  try {
    const response = await fetch("/api/user");

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    const user = await response.json();
    return user;
  } catch (error) {
    console.error("Loading failed:", error);
    throw error;
  }
}
```

### What does `await` actually do?

It suspends the current async function until the awaited value settles,
then resumes it later. Other JavaScript work can run in the meantime.

It does not make CPU-heavy synchronous code parallel.

### Common mistake: forgetting `await`

``` js
async function example() {
  const result = fetch("/api/data"); // result is a Promise
  console.log(result);
}
```

If you need the response, use `await fetch(...)`.

### Common mistake: forgetting error handling

`await` throws when the Promise rejects. Use `try/catch` or let the
rejection propagate to a caller that handles it.

**Remember:** `async` = *this function returns a Promise*. `await` =
*pause this function until the Promise settles*.

------------------------------------------------------------------------

## 12. Running Work Sequentially vs. Concurrently

### Sequential

**Simple definition:** Sequential work waits for each operation before
starting the next.

``` js
const user = await fetchUser();
const orders = await fetchOrders();
const alerts = await fetchAlerts();
```

This is correct if later steps depend on earlier results. If all
operations are independent, it may waste time.

### Concurrent

**Simple definition:** Concurrent work starts independent operations
before waiting for all their results.

``` js
const userPromise = fetchUser();
const ordersPromise = fetchOrders();
const alertsPromise = fetchAlerts();

const user = await userPromise;
const orders = await ordersPromise;
const alerts = await alertsPromise;
```

Or more compactly:

``` js
const [user, orders, alerts] = await Promise.all([
  fetchUser(),
  fetchOrders(),
  fetchAlerts(),
]);
```

### Visual comparison

``` text
Sequential:
User   [------]
Orders        [------]
Alerts               [------]

Concurrent:
User   [------]
Orders [------]
Alerts [------]
       total time can be shorter
```

This helps most when independent operations spend time waiting on I/O.
It does not guarantee they run on multiple CPU cores.

**Remember:** If task B needs task A’s result, sequence them. If they
are independent, consider starting them together.

------------------------------------------------------------------------

## 13. Promise.all, allSettled, race, and any

### `Promise.all`

**Remember:** *All succeed, or the combined Promise rejects.*

``` js
const [profile, orders] = await Promise.all([
  fetchProfile(),
  fetchOrders(),
]);
```

It rejects when one input rejects. It does not automatically cancel the
other operations.

### `Promise.allSettled`

**Remember:** *Wait for everyone and report each outcome.*

``` js
const outcomes = await Promise.allSettled([
  fetchProfile(),
  fetchOrders(),
]);

for (const outcome of outcomes) {
  if (outcome.status === "fulfilled") {
    console.log("Success:", outcome.value);
  } else {
    console.error("Failure:", outcome.reason);
  }
}
```

### `Promise.race`

**Remember:** *The first Promise to settle wins—success or failure.*

``` js
const result = await Promise.race([
  fetchData(),
  timeoutPromise(3000),
]);
```

A race alone does not stop the losing operation. Use cancellation if you
need to stop it.

### `Promise.any`

**Remember:** *The first successful Promise wins.*

``` js
const result = await Promise.any([
  fetchFromReplicaA(),
  fetchFromReplicaB(),
]);
```

It rejects with an `AggregateError` if all input Promises reject.

| Method       | Easy memory rule      | Rejects when                                   |
|--------------|-----------------------|------------------------------------------------|
| `all`        | Everyone must succeed | Any input rejects                              |
| `allSettled` | Report every result   | Normally does not reject due to input failures |
| `race`       | First to settle wins  | The first settled result is a rejection        |
| `any`        | First success wins    | Every input rejects                            |

**Production note:** Start only as much concurrent work as your services
can handle. `Promise.all` over 100,000 tasks can create a large amount
of in-flight work.

------------------------------------------------------------------------

## 14. Timers: setTimeout and setInterval

### `setTimeout`

**Simple definition:** Schedule a callback to run no earlier than a
requested delay, when the runtime can process it.

``` js
setTimeout(() => {
  console.log("Timer fired");
}, 1000);
```

A delay of 1000 ms is not a promise that it runs at exactly one second.
Busy JavaScript, browser throttling, and runtime scheduling can delay
it.

### `setInterval`

**Simple definition:** Ask the runtime to schedule a callback repeatedly
at an approximate interval.

``` js
const id = setInterval(() => {
  console.log("Heartbeat");
}, 1000);

// Later:
clearInterval(id);
```

### Common problem: overlapping async interval work

``` js
setInterval(async () => {
  await refreshData();
}, 1000);
```

If `refreshData()` takes longer than one second, calls may overlap.

A sequential loop can avoid overlap:

``` js
async function refreshLoop() {
  while (true) {
    await refreshData();

    await new Promise((resolve) => setTimeout(resolve, 1000));
  }
}
```

In production, add cancellation and error handling so a failed refresh
does not crash the loop or cause a tight retry cycle.

**Remember:** Timers schedule work; they do not guarantee exact
execution time.

------------------------------------------------------------------------

## 15. Fetch, Network I/O, and AbortController

### Simple definition to remember

**`fetch()` starts an HTTP request and returns a Promise for the
response; the request can progress while JavaScript does other work.**

``` js
async function loadLoads() {
  const response = await fetch("/api/loads");

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}
```

`fetch` usually does not reject for HTTP status codes such as 404 or
500, so check `response.ok`.

### Cancellation with AbortController

``` js
const controller = new AbortController();

try {
  const response = await fetch("/api/loads", {
    signal: controller.signal,
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  const data = await response.json();
  console.log(data);
} catch (error) {
  if (error.name === "AbortError") {
    console.log("Request cancelled");
  } else {
    throw error;
  }
}

// Call when you no longer need the request:
controller.abort();
```

For a real timeout, arrange for `abort()` to run after the deadline and
clean up the timer. Modern runtimes also provide convenience APIs such
as `AbortSignal.timeout()`; verify support in the browsers/Node.js
versions you target.

**Remember:** `fetch` gives you a Promise; `AbortController` gives you a
way to request cancellation.

------------------------------------------------------------------------

## 16. JavaScript Is Single-Threaded—What Does That Mean?

### Simple definition to remember

**On a normal browser page, one JavaScript thread runs one piece of
JavaScript at a time.**

That does not mean the entire browser or Node.js process does only one
thing. The environment can perform I/O, rendering, and other work
separately, and worker threads can execute JavaScript elsewhere.

### Why async I/O works

``` text
JavaScript thread: start request -> handle another event -> process response
Network system:         [request in progress]
```

While the network is waiting, JavaScript can process other work. When
the response is ready, a callback or Promise continuation becomes
eligible to run.

### What blocks the thread?

``` js
// Bad for a browser UI or a Node.js server:
const start = Date.now();
while (Date.now() - start < 5000) {
  // busy loop blocks this JavaScript thread for ~5 seconds
}
```

While this runs, the same thread cannot process other JavaScript
callbacks. In a browser it can freeze interaction; in Node.js it can
delay requests.

**Remember:** Async I/O lets the thread stay useful while waiting.
CPU-heavy JavaScript still needs chunking or workers if it must not
block the main thread.

------------------------------------------------------------------------

## 17. Web Workers and Worker Threads

### Web Worker

**Simple definition:** A Web Worker runs JavaScript in a background
thread separate from the browser page’s main JavaScript thread.

Useful for: - CPU-heavy calculations; - parsing large files; -
image/data transformations; - keeping the UI responsive.

A normal Web Worker cannot directly manipulate the page DOM. It
communicates using messages.

Main page (`main.js`):

``` js
const worker = new Worker("./worker.js");

worker.postMessage({ numbers: [1, 2, 3, 4] });

worker.onmessage = (event) => {
  console.log("Result:", event.data);
};

worker.onerror = (event) => {
  console.error("Worker failed:", event.message);
};
```

Worker (`worker.js`):

``` js
self.onmessage = (event) => {
  const total = event.data.numbers.reduce((sum, n) => sum + n, 0);
  self.postMessage(total);
};
```

### Node.js Worker Threads

**Simple definition:** Node.js `worker_threads` lets JavaScript run
CPU-intensive work on separate threads.

``` js
// main.js
const { Worker } = require("node:worker_threads");

const worker = new Worker("./cpu-worker.js", {
  workerData: { limit: 100_000 },
});

worker.on("message", (result) => console.log(result));
worker.on("error", console.error);
worker.on("exit", (code) => {
  if (code !== 0) console.error(`Worker stopped with code ${code}`);
});
```

``` js
// cpu-worker.js
const { parentPort, workerData } = require("node:worker_threads");

let total = 0;
for (let i = 0; i < workerData.limit; i++) {
  total += i;
}

parentPort.postMessage(total);
```

These are separate files; run `node main.js` in a Node.js project.

### When should you use workers?

- **Use async I/O** for network and ordinary file/database waiting.
- **Use workers** for substantial CPU-heavy JavaScript that must run
  independently.
- **Do not use a worker for every tiny task**: communication, startup,
  and data-transfer costs can outweigh the benefit.

**Remember:** Promise = organize asynchronous results. Worker = execute
JavaScript on another thread.

------------------------------------------------------------------------

## 18. SharedArrayBuffer and Atomics

### Simple definition to remember

**`SharedArrayBuffer` lets workers share a block of memory; `Atomics`
provides safe atomic operations on supported shared typed-array
values.**

Normally, messages between workers are the simpler way to exchange data.
Shared memory can be faster for specialized workloads but is harder to
reason about.

Conceptual example:

``` js
const shared = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT);
const counter = new Int32Array(shared);

Atomics.add(counter, 0, 1);
console.log(Atomics.load(counter, 0));
```

This example illustrates atomic access to shared-compatible memory. It
does not show two workers; both workers would need access to the same
shared buffer.

### Why not use shared memory everywhere?

- You must reason about synchronization.
- Unsynchronized reads/writes can create race-like logic errors.
- Browser availability can depend on security requirements, commonly
  cross-origin isolation.
- Some `Atomics` operations, such as waiting, have environment-specific
  restrictions.

**Remember:** Shared memory is powerful but complex. Prefer message
passing unless shared memory solves a measured problem.

------------------------------------------------------------------------

## 19. Node.js Event Loop

### Simple definition to remember

**The Node.js event loop lets Node.js process callbacks for timers, I/O,
and other events while keeping JavaScript execution on its event-loop
thread.**

Node.js uses the libuv library for event-loop and I/O-related
facilities. V8 executes JavaScript. Some operations use the operating
system; some APIs use a thread pool; details depend on the operation and
platform.

### Commonly discussed phases

Node.js event-loop phases include timers, pending callbacks, poll,
check, and close callbacks. The exact behavior depends on Node.js
version and execution context.

- `setTimeout` / `setInterval`: timer callbacks.
- I/O callbacks: often run as I/O becomes ready.
- `setImmediate`: callback scheduled for the check phase.
- `process.nextTick`: Node-specific queue processed before the event
  loop continues in common situations.
- Promise reactions: microtasks processed by V8 according to Node’s
  integration rules.

Do not rely on one universal output ordering for `setTimeout(fn, 0)`
vs. `setImmediate(fn)` in every context.

### Example

``` js
const fs = require("node:fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
```

Inside this I/O callback, `setImmediate` is commonly observed before the
zero-delay timer. Treat scheduling details as
environment/context-specific, not as a general timing guarantee.

### `process.nextTick` caution

Recursively scheduling `process.nextTick` callbacks can prevent the
event loop from making progress on I/O. Avoid unbounded next-tick
recursion.

**Remember:** Node’s event loop is not a worker pool for all JavaScript.
CPU-heavy JavaScript on the main thread still blocks that thread.

------------------------------------------------------------------------

## 20. Streams and Backpressure

### Simple definition to remember

**A stream processes data in pieces instead of loading the entire thing
into memory at once.**

Examples: - downloading a large file; - processing logs; - reading a
large CSV; - streaming a response to a client.

### Backpressure

**Simple definition:** Backpressure means a slow consumer signals that
the producer should slow down.

``` text
Fast producer ─────> Queue ─────> Slow consumer
                       |
                 queue fills up
                       |
               producer slows down
```

Without backpressure, a fast producer can fill memory faster than the
consumer can process data.

### Node.js stream example

``` js
const fs = require("node:fs");
const { pipeline } = require("node:stream/promises");

async function copyFile() {
  await pipeline(
    fs.createReadStream("large-input.txt"),
    fs.createWriteStream("large-output.txt"),
  );
}

copyFile().catch(console.error);
```

`pipeline` helps connect streams and propagate errors. In production,
also handle cancellation and filesystem permissions/errors.

**Remember:** Stream = *process pieces*. Backpressure = *slow the
producer when the consumer cannot keep up*.

------------------------------------------------------------------------

## 21. Concurrency Limits and Worker Pools

### Simple definition to remember

**A concurrency limit puts a cap on how many operations can be in
progress at once.**

This matters when calling a database, a third-party API, or processing
many files.

### Why not `Promise.all` on a huge array?

``` js
await Promise.all(millionItems.map(processItem));
```

This creates a Promise/task for every item at once. It can use lots of
memory and overload downstream services.

### A small concurrency-limited helper

``` js
async function mapLimit(items, limit, mapper) {
  if (!Number.isInteger(limit) || limit < 1) {
    throw new Error("limit must be a positive integer");
  }

  const results = new Array(items.length);
  let nextIndex = 0;

  async function worker() {
    while (true) {
      const index = nextIndex++;
      if (index >= items.length) return;

      results[index] = await mapper(items[index], index);
    }
  }

  const workerCount = Math.min(limit, items.length);
  await Promise.all(
    Array.from({ length: workerCount }, () => worker()),
  );

  return results;
}

// At most 5 mapper calls are active at once:
const results = await mapLimit(loads, 5, async (load) => {
  return checkCarrierAvailability(load);
});
```

This helper keeps result order aligned with input order. If `mapper`
rejects, `Promise.all` rejects, but other workers may already be
processing items; this helper does not cancel them. Add
cancellation/error semantics if required in production.

### Rate limit vs. concurrency limit

- **Concurrency limit:** max in-flight operations, e.g. 5 at a time.
- **Rate limit:** max starts per time unit, e.g. 20 requests per second.

They are different controls; some systems need both.

**Remember:** `Promise.all` coordinates results; a worker pool or
limiter controls how much work starts at once.

------------------------------------------------------------------------

## 22. Race Conditions in JavaScript

### Simple definition to remember

**A race condition happens when the result depends on which of multiple
operations finishes first.**

JavaScript on one thread can still have logical race conditions when
asynchronous operations overlap and complete in an unexpected order.

### Example: old search result overwrites new result

``` js
let currentQuery = "";

async function search(query) {
  currentQuery = query;
  const results = await fetchSearchResults(query);

  // Bug: older request can finish after newer request.
  renderResults(results);
}
```

If the user types `c`, then `ca`, then `car`, the `c` request might
finish last and overwrite the `car` results.

### Fix with a request ID

``` js
let latestRequestId = 0;

async function search(query) {
  const requestId = ++latestRequestId;
  const results = await fetchSearchResults(query);

  if (requestId !== latestRequestId) return;
  renderResults(results);
}
```

Or cancel the previous request using `AbortController`.

### Shared state and workers

With multiple workers and shared memory, actual data races can occur if
shared memory is accessed without appropriate synchronization. On a
single JavaScript thread, many races are instead ordering bugs between
asynchronous tasks.

**Remember:** Race condition = *wrong winner because completion order
changed*.

------------------------------------------------------------------------

## 23. Cancellation, Timeouts, and Cleanup

### Simple definition to remember

**Cancellation tells work it is no longer needed; cleanup releases
resources when work stops.**

JavaScript Promises do not have a universal built-in cancellation
method. Cancellation usually comes from the API, such as
`AbortController`.

### Cancel a fetch

``` js
const controller = new AbortController();

const request = fetch("/api/loads", {
  signal: controller.signal,
});

// When user navigates away or request is no longer useful:
controller.abort();

try {
  const response = await request;
  console.log(await response.json());
} catch (error) {
  if (error.name === "AbortError") {
    console.log("Cancelled");
  } else {
    throw error;
  }
}
```

### Timeout pattern

``` js
async function fetchWithTimeout(url, timeoutMs = 5000) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), timeoutMs);

  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timer);
  }
}
```

The caller should still check `response.ok`. In environments that
support `AbortSignal.timeout`, that API may be a simpler alternative.

### Cleanup examples

- Clear timers.
- Remove event listeners.
- Abort requests.
- Close streams and sockets.
- Terminate workers when no longer needed.
- Unsubscribe from subscriptions.

**Remember:** Start work with an exit plan: *how will it stop, and who
cleans it up?*

------------------------------------------------------------------------

## 24. Memory Leaks and Unhandled Rejections

### Memory leak

**Simple definition:** A memory leak occurs when the program keeps
references to data it no longer needs, so memory cannot be reclaimed.

Common causes: - forgotten timers; - event listeners never removed; -
long-lived caches without limits; - closures retaining large objects; -
unresolved work retaining request data; - growing queues.

### Unhandled Promise rejection

**Simple definition:** An unhandled rejection is a Promise failure that
no code handles appropriately.

``` js
async function main() {
  await saveData();
}

main().catch((error) => {
  console.error("Application operation failed:", error);
});
```

Inside an async function, an awaited rejected Promise throws. If the
async function’s returned Promise is ignored, its rejection may become
unhandled.

### Avoid detached Promise errors

Risky:

``` js
doImportantWork(); // if it rejects, caller ignores the Promise
```

Better:

``` js
await doImportantWork();
```

Or, for deliberate background work:

``` js
void doBackgroundWork().catch((error) => {
  logger.error({ error }, "Background work failed");
});
```

Do not silence errors with an empty catch.

**Remember:** Every Promise failure needs an owner; every long-lived
resource needs cleanup.

------------------------------------------------------------------------

## 25. Performance and Responsiveness

### Simple definition to remember

**Performance work means finding the real bottleneck and improving it
without breaking correctness.**

### CPU-bound vs. I/O-bound

- **CPU-bound:** calculation dominates; more async syntax will not make
  it run on another core.
- **I/O-bound:** waiting on network, disk, or database dominates;
  concurrency may overlap waiting time.

### Browser: avoid blocking the UI

Break long work into smaller chunks or move CPU-heavy processing to a
Web Worker.

### Node.js: avoid blocking the event loop

Use async APIs for supported I/O and Worker Threads for sufficiently
expensive CPU work. Do not turn every small operation into a worker
task.

### Measure

Look at: - response time and p95/p99 latency; - event-loop delay; - CPU
and memory; - number of in-flight requests; - queue depth; - error
rate; - database and external API wait times.

Node.js provides performance APIs and diagnostic tooling; browsers
provide performance panels and profiles.

**Remember:** Async helps waiting. Workers help CPU parallelism.
Measurement tells you which one you need.

------------------------------------------------------------------------

## 26. Testing and Debugging

### Simple definition to remember

**A concurrency test checks behavior when operations overlap, not just
when they run one after another.**

### Test important timing cases

- Two requests complete in the opposite order from which they started.
- A request is canceled while in flight.
- One operation fails while others are still running.
- A consumer is slower than its producer.
- A component unmounts while a request is pending.
- Shutdown starts while background work is active.

### Avoid arbitrary sleeps in tests

Fragile:

``` js
startBackgroundWork();
await new Promise((resolve) => setTimeout(resolve, 100));
expect(done).toBe(true);
```

Better: expose a Promise, event, or test hook that represents actual
completion, and await it with a test timeout.

### Browser debugging

Use browser developer tools to inspect: - call stacks; - network
timing; - long tasks; - worker activity; - memory snapshots; -
performance profiles.

### Node.js debugging

Use: - `node --inspect`; - CPU and heap profiles; - `node:perf_hooks`; -
event-loop delay monitoring; - heap snapshots; - logging with request
IDs.

**Remember:** Test the difficult orderings and failure paths, not only
the happy path.

------------------------------------------------------------------------

## 27. Real Production Scenarios

### Scenario A: GoodSeva dashboard loading

A dashboard needs load counts, carrier counts, recent trips, and payment
summaries.

If these API calls are independent:

``` js
const [loads, carriers, trips, payments] = await Promise.all([
  getLoadStats(),
  getCarrierStats(),
  getTripStats(),
  getPaymentStats(),
]);
```

If one panel may fail independently, consider `Promise.allSettled` and
render partial data. Add timeouts and avoid launching too many duplicate
requests on every render.

### Scenario B: Carrier matching

1.  Fetch candidate carriers.
2.  Run independent availability checks with a concurrency limit.
3.  Apply API rate limits.
4.  Cancel obsolete checks when the load is canceled.
5.  Collect errors and sort results deterministically.
6.  Revalidate critical decisions on the server/database.

Do not rely on browser JavaScript alone to enforce business rules.

### Scenario C: Live GPS updates

- Use WebSockets or another suitable transport for updates.
- Process messages without blocking the event loop with heavy
  calculations.
- Throttle or batch UI rendering when updates are frequent.
- Define how to handle stale or out-of-order positions.
- Bound queues and discard or coalesce obsolete display updates when the
  product semantics allow it.

### Scenario D: Large CSV import

Use streaming to avoid loading the entire file into memory. Validate
rows in bounded batches, handle failures explicitly, and apply
backpressure to database writes.

### Scenario E: Search-as-you-type

Debounce input to reduce requests; cancel or ignore stale requests so an
older result cannot overwrite a newer one.

**Remember:** Production concurrency is not just starting tasks. It is
controlling volume, ordering, cancellation, errors, and resource limits.

------------------------------------------------------------------------

## 28. Common Mistakes

- [ ] Assuming `async` makes CPU-heavy code run on another thread.
- [ ] Forgetting to `await` a Promise.
- [ ] Starting independent tasks sequentially for no reason.
- [ ] Using `Promise.all` for an unbounded number of jobs.
- [ ] Assuming `Promise.all` cancels operations after one rejects.
- [ ] Assuming `setTimeout(fn, 0)` runs immediately.
- [ ] Using `setInterval` with slow async work and accidentally
  overlapping calls.
- [ ] Forgetting that `fetch` can fulfill for HTTP 404/500 responses.
- [ ] Ignoring a Promise that can reject.
- [ ] Forgetting to cancel requests or clean up timers/listeners.
- [ ] Using a Web Worker for work so small that overhead dominates.
- [ ] Assuming single-threaded JavaScript cannot have race conditions.
- [ ] Blocking Node.js with long CPU loops.
- [ ] Assuming completion order matches start order.
- [ ] Relying on a particular `setTimeout` vs. `setImmediate` order
  everywhere in Node.js.

------------------------------------------------------------------------

## 29. JavaScript vs. Go vs. Java

| Concept               | JavaScript                                                                 | Go                                          | Java                                         |
|-----------------------|----------------------------------------------------------------------------|---------------------------------------------|----------------------------------------------|
| Main async model      | Event loop, Promises, async APIs                                           | Goroutines, channels, runtime scheduler     | Threads, executors, futures, virtual threads |
| Start async work      | Call an async API / schedule callback                                      | `go fn()`                                   | Executor or thread API                       |
| Async I/O             | Runtime APIs + Promises/callbacks                                          | Standard/runtime APIs with goroutines       | Libraries, futures, virtual threads          |
| CPU parallelism       | Web Workers / Node Worker Threads                                          | Goroutines across available processors      | Platform/virtual threads plus CPU executors  |
| Coordinate results    | Promises, `Promise.all`                                                    | Channels, `WaitGroup`, `errgroup`           | Futures, queues, locks                       |
| Protect shared memory | Usually isolate state; workers need messages or shared-memory coordination | Mutexes, atomics, channels                  | `synchronized`, locks, atomics               |
| Cancellation          | API-specific, often `AbortController`                                      | `context.Context`                           | Interrupts and API-specific cancellation     |
| Common bug            | Stale async result, event-loop blocking                                    | Goroutine leak, channel deadlock, data race | Race, deadlock, executor starvation          |

### Easy mental difference

- **JavaScript:** start asynchronous operations and process their
  results through the event loop; use workers for CPU parallelism.
- **Go:** start goroutines and coordinate them using channels, locks,
  and contexts.
- **Java:** use threads, executors, futures, locks, and modern virtual
  threads according to the task.

------------------------------------------------------------------------

## 30. Interview Questions and Simple Answers

**1. What is asynchronous programming?**  
Starting work without blocking the current flow while waiting for the
result.

**2. What is the event loop?**  
The runtime mechanism that schedules queued callbacks when JavaScript
can run them.

**3. Is JavaScript single-threaded?**  
A normal page’s JavaScript runs one piece at a time on its main thread,
but the runtime performs other work and workers can run JavaScript on
separate threads.

**4. Does `async` create a new thread?**  
No. It makes a function return a Promise and lets it use `await`.

**5. What does `await` do?**  
Suspends the current async function until the awaited value settles; it
does not block the whole JavaScript thread.

**6. What is a Promise?**  
An object representing a future success value or failure.

**7. What are Promise states?**  
Pending, fulfilled, and rejected.

**8. What is a microtask?**  
A queued job such as a Promise reaction that the runtime processes at a
microtask checkpoint.

**9. Why does Promise output appear before a timer?**  
Promise reactions are microtasks; timer callbacks are scheduled as
tasks, so in the standard example the Promise reaction runs first after
synchronous code.

**10. `Promise.all` vs. `Promise.allSettled`?**  
`all` rejects when an input rejects; `allSettled` waits for all
outcomes.

**11. `Promise.race` vs. `Promise.any`?**  
`race` returns the first settled result; `any` returns the first
fulfilled result.

**12. Does `Promise.all` cancel remaining requests?**  
No. It rejects its combined Promise but does not automatically cancel
other operations.

**13. What is callback hell?**  
Deeply nested callbacks that make flow and error handling difficult.

**14. What is a race condition in JavaScript?**  
An incorrect result caused by asynchronous operations finishing in an
unexpected order.

**15. How do you prevent stale search results?**  
Use request IDs or cancel the previous request with `AbortController`.

**16. How do you cancel `fetch`?**  
Pass an `AbortSignal` from an `AbortController`, then call `abort()`.

**17. What is a Web Worker?**  
A browser worker that executes JavaScript on a separate thread and
communicates by messages.

**18. When do you use Node Worker Threads?**  
For CPU-intensive JavaScript that should not block the Node.js
event-loop thread.

**19. What is backpressure?**  
A way to prevent a fast producer from overwhelming a slow consumer.

**20. Why can `setInterval(async ...)` be dangerous?**  
A new run may begin before the previous async run finishes.

**21. What blocks the Node.js event loop?**  
Long-running synchronous JavaScript, such as a large CPU loop.

**22. What is the difference between concurrency and parallelism?**  
Concurrency is overlapping progress; parallelism is simultaneous
execution.

**23. Why check `response.ok` after `fetch`?**  
HTTP error statuses such as 404 or 500 do not normally make `fetch`
reject.

**24. What is `AbortController`?**  
An API for signaling cancellation to operations that support an abort
signal.

**25. How do you process 100,000 items safely?**  
Use bounded concurrency, streaming or batching where appropriate,
cancellation, and an explicit error policy instead of starting every
operation at once.

### Practice exercises

1.  Predict the output of a program mixing synchronous logs, Promise
    callbacks, and timers.
2.  Fetch two APIs sequentially, then rewrite them to run concurrently.
3.  Use `Promise.allSettled` to display partial dashboard results.
4.  Build search-as-you-type with request cancellation.
5.  Write `mapLimit` and test it with a maximum of three active
    operations.
6.  Use a Web Worker to calculate a large sum without freezing a page.
7.  Stream-copy a large file in Node.js.
8.  Reproduce a stale-response race and fix it.
9.  Add timeout and cancellation to an API client.
10. Design a graceful shutdown for background tasks in Node.js.

------------------------------------------------------------------------

## 31. Learning Path

### Stage 1 — Foundations

- Synchronous vs. asynchronous code
- Call stack and runtime
- Event loop
- Callbacks
- Promises

### Stage 2 — Everyday async JavaScript

- Promise chaining
- `async/await`
- Error handling
- `Promise.all` and related combinators
- Fetch and `AbortController`

### Stage 3 — Production reliability

- Sequential vs. concurrent operations
- Concurrency limits
- Race conditions and stale results
- Timeouts and cleanup
- Streams and backpressure

### Stage 4 — Advanced runtime

- Browser Web Workers
- Node.js Worker Threads
- Node.js event-loop phases
- SharedArrayBuffer and Atomics
- Profiling and performance

### Suggested mini-projects

1.  **Dashboard loader:** fetch four independent APIs concurrently and
    support partial failure.
2.  **Search component:** debounce input, cancel old requests, prevent
    stale results.
3.  **Batch API processor:** process thousands of records with bounded
    concurrency.
4.  **Large file processor:** use streams and backpressure in Node.js.
5.  **CPU worker:** move a costly calculation to a Web Worker or Worker
    Thread.

------------------------------------------------------------------------

## 32. One-Minute Revision Cheat Sheet

| Need                                          | Use                               |
|-----------------------------------------------|-----------------------------------|
| Run code after current stack                  | Callback / scheduled task         |
| Represent a future result                     | Promise                           |
| Write Promise code in step-by-step style      | `async/await`                     |
| Start independent Promise operations together | `Promise.all`                     |
| Collect every success/failure                 | `Promise.allSettled`              |
| First result, success or failure              | `Promise.race`                    |
| First successful result                       | `Promise.any`                     |
| Delay or repeat work                          | Timers, with cleanup              |
| Cancel supported async work                   | `AbortController` / `AbortSignal` |
| Run CPU work on another thread in browser     | Web Worker                        |
| Run CPU work on another thread in Node.js     | `worker_threads`                  |
| Process large data piece by piece             | Streams                           |
| Stop producer overwhelming consumer           | Backpressure                      |
| Cap in-flight tasks                           | Concurrency limiter / worker pool |
| Prevent old request overwriting new UI        | Request ID or cancellation        |
| Diagnose blocked Node.js execution            | Profile event loop and CPU        |
| Keep async failures visible                   | `await`, `try/catch`, `.catch()`  |

### Five rules to remember forever

1.  **Async does not automatically mean parallel.**
2.  **`await` pauses one async function, not the entire JavaScript
    thread.**
3.  **Start independent I/O work together, but limit how much you
    start.**
4.  **Every async operation needs an error plan, and long-running work
    needs a cancellation/cleanup plan.**
5.  **Completion order can differ from start order—protect your state
    from stale results.**

------------------------------------------------------------------------

## Final mental model

``` text
Call stack   = what JavaScript is running right now
Runtime APIs = timers, network, file and other environment features
Event loop   = when queued work gets a chance to run
Promise      = a future result or failure
async/await  = a readable way to work with Promises
Worker       = JavaScript executing on another thread
Backpressure = keep producers from overwhelming consumers
AbortSignal  = tell supported work it is no longer needed
```

**Interview-ready summary:** JavaScript handles asynchronous work using
runtime APIs, Promises, and the event loop. This allows I/O tasks to
overlap without blocking the JavaScript thread. For CPU-heavy work, use
workers; for high-volume operations, use bounded concurrency; and for
reliable production systems, handle ordering, errors, cancellation,
cleanup, and backpressure explicitly.
