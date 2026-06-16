# JavaScript Arrays & Objects – Interview Revision Guide 🚀

Arrays and Objects are the most used data structures in JavaScript.

Almost every JavaScript interview contains questions on:

- Array Methods
- Object Manipulation
- Destructuring
- Spread Operator
- Shallow Copy vs Deep Copy

Master these topics before moving to DOM and Async JavaScript.

---

# Table of Contents

1. Arrays
2. map()
3. filter()
4. find()
5. reduce()
6. some()
7. every()
8. Array Destructuring
9. Objects
10. Object References
11. Shallow Copy
12. Deep Copy
13. Spread Operator (...)
14. structuredClone()
15. Interview Quick Revision

---

# 1. Arrays

An array is used to store multiple values in a single variable.

Example:

```js
const nums = [1, 2, 3, 4, 5];
```

Access elements:

```js
console.log(nums[0]);
```

Output:

```js
1
```

---

## Why Arrays?

Instead of:

```js
let num1 = 10;
let num2 = 20;
let num3 = 30;
```

Use:

```js
const nums = [10, 20, 30];
```

Much cleaner and easier to manage.

---

# 2. map()

Most asked array method in interviews.

---

## What is map()?

`map()` creates a NEW array by transforming each element.

Original array remains unchanged.

---

### Syntax

```js
array.map(callback)
```

---

### Example

```js
const nums = [1, 2, 3];

const doubled = nums.map(num => num * 2);

console.log(doubled);
```

Output:

```js
[2, 4, 6]
```

---

### Visualization

```js
[1, 2, 3]
```

↓

Multiply by 2

↓

```js
[2, 4, 6]
```

---

### Real Example

```js
const users = [
  { name: "John" },
  { name: "David" }
];

const names = users.map(user => user.name);

console.log(names);
```

Output:

```js
["John", "David"]
```

---

# 3. filter()

Used to keep only elements that match a condition.

---

## Syntax

```js
array.filter(callback)
```

---

### Example

```js
const nums = [1, 2, 3, 4, 5];

const even = nums.filter(num => num % 2 === 0);

console.log(even);
```

Output:

```js
[2, 4]
```

---

### Visualization

```js
[1,2,3,4,5]
```

↓

Keep even numbers

↓

```js
[2,4]
```

---

### Real Example

```js
const users = [
  { active: true },
  { active: false },
  { active: true }
];

const activeUsers = users.filter(
  user => user.active
);

console.log(activeUsers);
```

---

# 4. find()

Returns the FIRST matching element.

---

### Example

```js
const nums = [10, 20, 30, 40];

const result = nums.find(
  num => num > 20
);

console.log(result);
```

Output:

```js
30
```

---

## Important

Returns:

```js
Single value
```

NOT an array.

---

### If Not Found

```js
const result = nums.find(
  num => num > 100
);

console.log(result);
```

Output:

```js
undefined
```

---

# 5. reduce()

Most confusing initially, but very important.

---

## What is reduce()?

Used to reduce an array into a single value.

Examples:

- Sum
- Product
- Count
- Grouping

---

### Syntax

```js
array.reduce(callback, initialValue)
```

---

### Sum Example

```js
const nums = [1, 2, 3, 4];

const total = nums.reduce(
  (accumulator, current) => {
    return accumulator + current;
  },
  0
);

console.log(total);
```

Output:

```js
10
```

---

## Step-by-Step

```js
acc = 0
current = 1

0 + 1 = 1
```

---

```js
acc = 1
current = 2

1 + 2 = 3
```

---

```js
acc = 3
current = 3

3 + 3 = 6
```

---

```js
acc = 6
current = 4

6 + 4 = 10
```

Final:

```js
10
```

---

### Short Version

```js
const total = nums.reduce(
  (acc, curr) => acc + curr,
  0
);
```

---

# 6. some()

Checks if AT LEAST ONE element satisfies a condition.

Returns:

```js
true
```

or

```js
false
```

---

### Example

```js
const nums = [1, 2, 3, 4];

const result = nums.some(
  num => num > 3
);

console.log(result);
```

Output:

```js
true
```

Because:

```js
4 > 3
```

---

### Example

```js
const nums = [1, 2, 3];

nums.some(num => num > 10);
```

Output:

```js
false
```

---

# 7. every()

Checks if ALL elements satisfy a condition.

---

### Example

```js
const nums = [2, 4, 6];

const result = nums.every(
  num => num % 2 === 0
);

console.log(result);
```

Output:

```js
true
```

---

### Example

```js
const nums = [2, 4, 5];

const result = nums.every(
  num => num % 2 === 0
);

console.log(result);
```

Output:

```js
false
```

---

# some() vs every()

| Method | Meaning |
|----------|----------|
| some() | At least one matches |
| every() | All must match |

---

Example:

```js
[1,2,3].some(x => x > 2);
```

Output:

```js
true
```

---

```js
[1,2,3].every(x => x > 2);
```

Output:

```js
false
```

---

# 8. Array Destructuring

Allows extracting values from arrays easily.

---

### Without Destructuring

```js
const nums = [10, 20];

const first = nums[0];
const second = nums[1];
```

---

### With Destructuring

```js
const nums = [10, 20];

const [first, second] = nums;
```

---

Output:

```js
first = 10
second = 20
```

---

### Skip Values

```js
const nums = [10, 20, 30];

const [first, , third] = nums;

console.log(third);
```

Output:

```js
30
```

---

# 9. Objects

Objects store data as key-value pairs.

---

### Example

```js
const user = {
  name: "Vinay",
  age: 25
};
```

---

### Access Values

Dot notation:

```js
console.log(user.name);
```

Output:

```js
Vinay
```

---

Bracket notation:

```js
console.log(user["name"]);
```

Output:

```js
Vinay
```

---

# Object Destructuring

Extract values directly.

---

### Without Destructuring

```js
const name = user.name;
const age = user.age;
```

---

### With Destructuring

```js
const { name, age } = user;
```

---

Output:

```js
name = "Vinay"
age = 25
```

---

# 10. Object References

Very important interview topic.

Objects are stored by reference.

---

### Example

```js
const user1 = {
  name: "Vinay"
};

const user2 = user1;
```

---

Now:

```js
user2.name = "John";
```

---

```js
console.log(user1.name);
```

Output:

```js
John
```

---

Why?

Both variables point to the SAME object in memory.

---

### Visualization

```js
user1 ----\
           ---> Object
user2 ----/
```

---

# 11. Shallow Copy

Creates a new object BUT nested objects are still shared.

---

### Using Spread Operator

```js
const user = {
  name: "Vinay"
};

const copy = {
  ...user
};
```

---

Modify copy:

```js
copy.name = "John";
```

Original remains unchanged.

```js
console.log(user.name);
```

Output:

```js
Vinay
```

---

## Problem with Nested Objects

```js
const user = {
  name: "Vinay",
  address: {
    city: "Hyderabad"
  }
};

const copy = {
  ...user
};
```

---

```js
copy.address.city = "Bangalore";
```

---

Output:

```js
user.address.city
```

↓

```js
"Bangalore"
```

---

Why?

Nested object is still shared.

This is called a Shallow Copy.

---

# 12. Deep Copy

Creates completely independent copies.

Nested objects are also copied.

---

### Example

```js
const user = {
  name: "Vinay",
  address: {
    city: "Hyderabad"
  }
};
```

Deep copy:

```js
const copy =
  structuredClone(user);
```

---

Now:

```js
copy.address.city = "Bangalore";
```

---

Original remains:

```js
Hyderabad
```

---

# 13. Spread Operator (...)

One of the most important ES6 features.

---

## Arrays

```js
const arr1 = [1, 2];

const arr2 = [...arr1];
```

Output:

```js
[1, 2]
```

---

### Merge Arrays

```js
const arr1 = [1, 2];
const arr2 = [3, 4];

const merged = [
  ...arr1,
  ...arr2
];
```

Output:

```js
[1,2,3,4]
```

---

## Objects

```js
const user = {
  name: "Vinay"
};

const copy = {
  ...user
};
```

---

### Add Properties

```js
const user = {
  name: "Vinay"
};

const updatedUser = {
  ...user,
  age: 25
};
```

Output:

```js
{
  name: "Vinay",
  age: 25
}
```

---

# 14. structuredClone()

Modern JavaScript method for deep copying.

---

### Syntax

```js
structuredClone(object)
```

---

### Example

```js
const user = {
  name: "Vinay",
  address: {
    city: "Hyderabad"
  }
};

const copy =
  structuredClone(user);
```

---

Modify copy:

```js
copy.address.city =
  "Bangalore";
```

---

Original:

```js
console.log(user.address.city);
```

Output:

```js
Hyderabad
```

---

## Why Better Than JSON Method?

Old approach:

```js
JSON.parse(
  JSON.stringify(obj)
);
```

Problems:

- Loses Dates
- Loses Maps
- Loses Sets
- Loses special object types

Modern approach:

```js
structuredClone(obj);
```

Preferred.

---

# Interview Quick Revision 🚀

## Array Methods

### map()

Transforms every element.

```js
arr.map(x => x * 2);
```

Returns:

```js
New Array
```

---

### filter()

Keeps matching elements.

```js
arr.filter(x => x > 10);
```

Returns:

```js
New Array
```

---

### find()

Returns first match.

```js
arr.find(x => x > 10);
```

Returns:

```js
Single Value
```

---

### reduce()

Reduces array to one value.

```js
arr.reduce(
  (acc, curr) => acc + curr,
  0
);
```

---

### some()

At least one matches.

```js
arr.some(x => x > 10);
```

Returns:

```js
true / false
```

---

### every()

All must match.

```js
arr.every(x => x > 10);
```

Returns:

```js
true / false
```

---

## Destructuring

Array:

```js
const [a, b] = arr;
```

Object:

```js
const { name } = user;
```

---

## Object References

```js
const a = {};
const b = a;
```

Both point to same object.

---

## Shallow Copy

```js
const copy = {
  ...obj
};
```

Nested objects are shared.

---

## Deep Copy

```js
const copy =
  structuredClone(obj);
```

Everything copied independently.

---

## Spread Operator

```js
...
```

Used for:

- copying arrays
- copying objects
- merging arrays
- merging objects

---

# Most Asked Interview Questions ⭐

### Q1: Difference between map() and filter()?

**map()**

Transforms elements.

```js
[1,2,3]
```

↓

```js
[2,4,6]
```

---

**filter()**

Removes unwanted elements.

```js
[1,2,3,4]
```

↓

```js
[2,4]
```

---

### Q2: Difference between find() and filter()?

**find()**

Returns first matching value.

```js
30
```

---

**filter()**

Returns all matching values.

```js
[30,40]
```

---

### Q3: What is Shallow Copy?

Top-level properties copied.

Nested objects still shared.

---

### Q4: What is Deep Copy?

Entire object copied including nested objects.

---

### Q5: Best way to Deep Copy?

```js
structuredClone(obj);
```

---

# Golden Interview Tips ⭐

1. Master map(), filter(), reduce() — these are asked everywhere.
2. Know the difference between find(), some(), and every().
3. Understand object references clearly.
4. Remember that spread operator creates a shallow copy.
5. Use structuredClone() for deep copying.
6. Practice array and object destructuring daily.
7. Be able to explain memory references with simple examples.

Arrays and Objects are the foundation of modern JavaScript. Most React, Node.js, and frontend interview questions build on these concepts.