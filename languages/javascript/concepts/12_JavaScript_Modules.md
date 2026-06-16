# JavaScript Modules – Interview Revision Guide 🚀

⚠️ **Modules are a very important topic in modern JavaScript, React, Node.js, and Full Stack interviews.**

Before modules existed, all code was written in one file.

Problems:

❌ Large files

❌ Naming conflicts

❌ Difficult maintenance

❌ Poor code organization

Modules solve these problems by splitting code into multiple files.

---

# Table of Contents

1. What are Modules?
2. Why Modules?
3. CommonJS Modules
4. module.exports
5. require()
6. ES Modules (ESM)
7. export
8. export default
9. import
10. Named Export vs Default Export
11. CommonJS vs ES Modules
12. Common Interview Questions
13. Interview Quick Revision

---

# 1. What are Modules?

A module is simply:

```text
A JavaScript file with reusable code.
```

Example:

```text
math.js

user.js

app.js
```

Each file becomes its own module.

---

# Why Use Modules?

Without modules:

```js
// one huge file

function add() {}

function subtract() {}

function multiply() {}

function divide() {}
```

Imagine:

```text
10,000 lines
```

in one file.

Hard to maintain.

---

With modules:

```text
math.js
user.js
auth.js
api.js
```

Code becomes organized.

---

# 2. Module Example

### math.js

```js
export function add(a, b) {
  return a + b;
}
```

---

### app.js

```js
import { add } from "./math.js";

console.log(add(2, 3));
```

Output:

```js
5
```

---

# 3. CommonJS

CommonJS is the module system used traditionally in Node.js.

---

## Export

```js
module.exports
```

---

## Import

```js
require()
```

---

# CommonJS Example

### math.js

```js
function add(a, b) {
  return a + b;
}

module.exports = add;
```

---

### app.js

```js
const add =
  require("./math");

console.log(add(2, 3));
```

Output:

```js
5
```

---

# CommonJS Flow

```text
module.exports
       ↓
require()
```

---

# Export Multiple Values

### math.js

```js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = {
  add,
  subtract
};
```

---

### app.js

```js
const math =
  require("./math");

console.log(
  math.add(5, 3)
);
```

Output:

```js
8
```

---

# 4. module.exports

Used to expose data from a module.

---

Example

```js
module.exports = {
  add,
  subtract
};
```

---

You can export:

```js
Functions
Objects
Arrays
Variables
Classes
```

---

Example

```js
const user = {
  name: "Vinay"
};

module.exports = user;
```

---

# 5. require()

Used to import CommonJS modules.

---

Example

```js
const user =
  require("./user");
```

---

JavaScript loads:

```text
user.js
```

and returns exported value.

---

# 6. ES Modules (ESM)

Modern JavaScript module system.

Introduced in ES6.

---

Uses:

```js
export
import
```

---

# Why ES Modules?

Cleaner syntax.

Works in:

✅ Modern Browsers

✅ React

✅ Angular

✅ Vue

✅ Modern Node.js

---

# ES Module Flow

```text
export
    ↓
import
```

---

# 7. export

Used to share variables, functions, or classes.

---

## Named Export

### math.js

```js
export function add(a, b) {
  return a + b;
}
```

---

Import:

```js
import { add }
from "./math.js";
```

---

# Export Multiple Values

```js
export const PI = 3.14;

export function add(a, b) {
  return a + b;
}
```

---

Import:

```js
import {
  PI,
  add
}
from "./math.js";
```

---

# Alternative Syntax

```js
function add() {}

function subtract() {}

export {
  add,
  subtract
};
```

---

# 8. export default

Used when a module exports one primary value.

---

### math.js

```js
export default function add(
  a,
  b
) {
  return a + b;
}
```

---

Import

```js
import add
from "./math.js";
```

---

Output

```js
add(2, 3);

5
```

---

# Important Rule

Default export:

```js
No curly braces
```

---

Named export:

```js
Curly braces required
```

---

# 9. import

Used to load modules.

---

# Import Named Export

```js
import { add }
from "./math.js";
```

---

# Import Multiple

```js
import {
  add,
  subtract
}
from "./math.js";
```

---

# Import Default

```js
import add
from "./math.js";
```

---

# Import Everything

```js
import * as math
from "./math.js";
```

---

Usage

```js
math.add(1, 2);
```

---

# Rename Imports

```js
import {
  add as sum
}
from "./math.js";
```

---

Usage

```js
sum(1, 2);
```

---

# 10. Named Export vs Default Export

Very common interview question.

---

## Named Export

### Export

```js
export const PI = 3.14;
```

---

### Import

```js
import { PI }
from "./math.js";
```

---

Uses:

```text
Multiple exports
```

---

# Default Export

### Export

```js
export default function() {}
```

---

### Import

```js
import myFunction
from "./file.js";
```

---

Uses:

```text
Single primary export
```

---

# Quick Comparison

| Named Export | Default Export |
|--------------|---------------|
| Multiple exports | One main export |
| Uses {} | No {} |
| Exact name needed | Any name allowed |

---

# Example

### Export

```js
export default function add() {}
```

---

Import

```js
import myAdd
from "./math.js";
```

Works.

---

Because default imports can be renamed.

---

# 11. CommonJS vs ES Modules

Most asked interview question.

---

| CommonJS | ES Modules |
|-----------|------------|
| require() | import |
| module.exports | export |
| Node.js traditional | Modern JS |
| Dynamic loading | Static analysis |
| Older syntax | Modern syntax |

---

## CommonJS

```js
const math =
require("./math");
```

---

## ES Modules

```js
import math
from "./math.js";
```

---

# Which One Should You Use?

Modern JavaScript:

```text
ES Modules
```

Preferred.

---

# Real Project Structure

```text
src
│
├── math.js
├── api.js
├── auth.js
└── app.js
```

---

### math.js

```js
export function add(a, b) {
  return a + b;
}
```

---

### app.js

```js
import { add }
from "./math.js";

console.log(add(5, 3));
```

---

Output

```js
8
```

---

# Common Interview Questions

---

## Q1: What is a Module?

A reusable JavaScript file.

---

## Q2: Why Use Modules?

- Reusability
- Maintainability
- Better organization

---

## Q3: What is CommonJS?

Node.js module system using:

```js
require()
module.exports
```

---

## Q4: What is ES Modules?

Modern JavaScript module system using:

```js
import
export
```

---

## Q5: Difference Between Named and Default Export?

### Named

```js
export const PI = 3.14;
```

Import:

```js
import { PI }
from "./file.js";
```

---

### Default

```js
export default function() {}
```

Import:

```js
import myFunc
from "./file.js";
```

---

## Q6: Can a File Have Multiple Named Exports?

Yes.

```js
export const PI = 3.14;

export function add() {}
```

---

## Q7: Can a File Have Multiple Default Exports?

❌ No.

Only one.

---

## Q8: Which Is Preferred Today?

```text
ES Modules
```

---

# Interview Quick Revision 🚀

## CommonJS Export

```js
module.exports = value;
```

---

## CommonJS Import

```js
require("./file");
```

---

## Named Export

```js
export const PI = 3.14;
```

---

## Named Import

```js
import { PI }
from "./file.js";
```

---

## Default Export

```js
export default add;
```

---

## Default Import

```js
import add
from "./file.js";
```

---

## Import Everything

```js
import * as math
from "./file.js";
```

---

# Most Important Interview Diagram ⭐

### CommonJS

```text
module.exports
       ↓
require()
```

---

### ES Modules

```text
export
    ↓
import
```

---

### Named Export

```js
export const PI = 3.14;
```

↓

```js
import { PI }
from "./file.js";
```

---

### Default Export

```js
export default add;
```

↓

```js
import add
from "./file.js";
```

---

# Golden Interview Tips ⭐

1. CommonJS uses:
   ```js
   require()
   module.exports
   ```
2. ES Modules uses:
   ```js
   import
   export
   ```
3. ES Modules are preferred in modern JavaScript.
4. Named exports require `{}` during import.
5. Default exports do not require `{}`.
6. A file can have many named exports.
7. A file can have only one default export.
8. React projects primarily use ES Modules.
9. Modern Node.js also supports ES Modules.
10. Remember:

```text
CommonJS
     ↓
require()

ES Modules
     ↓
import
```

🚀 Master Modules before learning bundlers (Webpack/Vite), React, and advanced frontend architecture.