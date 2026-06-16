# JavaScript Prototype & Inheritance – Interview Revision Guide 🚀

⚠️ **Prototype & Inheritance is one of the most important JavaScript interview topics.**

Many developers can write JavaScript code but struggle to explain:

- What is a Prototype?
- How does Inheritance work?
- What is the Prototype Chain?
- Constructor Functions
- ES6 Classes

The secret is:

> **JavaScript uses Prototypal Inheritance, not Classical Inheritance.**

Everything in this chapter builds on that idea.

---

# Table of Contents

1. Why Prototypes Exist
2. What is a Prototype?
3. Prototype Chain
4. Prototypal Inheritance
5. Constructor Functions
6. Constructor Function Inheritance
7. ES6 Classes
8. Class Inheritance
9. Class vs Prototype
10. Common Interview Questions
11. Interview Quick Revision

---

# 1. Why Prototypes Exist?

Imagine creating many users.

Without prototypes:

```js
const user1 = {
  name: "Vinay",

  greet() {
    console.log("Hello");
  }
};

const user2 = {
  name: "John",

  greet() {
    console.log("Hello");
  }
};
```

Problem:

```js
greet()
```

is duplicated in memory.

---

If we create:

```js
1000 users
```

then:

```js
1000 greet() functions
```

exist in memory.

Wasteful.

---

JavaScript solves this using:

```text
Prototype
```

---

# 2. What is a Prototype?

A prototype is an object that other objects can inherit properties and methods from.

Every JavaScript object has access to a prototype.

---

Example:

```js
const arr = [1, 2, 3];
```

You can do:

```js
arr.push(4);
arr.pop();
arr.map();
```

---

Question:

Where do these methods come from?

Not from the array itself.

They come from:

```js
Array.prototype
```

---

### Visualization

```text
arr
 ↓
Array.prototype
 ↓
Object.prototype
 ↓
null
```

This chain is called:

```text
Prototype Chain
```

---

# 3. Prototype Chain

One of the most asked interview topics.

---

Example:

```js
const user = {
  name: "Vinay"
};

console.log(user.name);
```

JavaScript checks:

```text
user
```

↓

Found?

```js
name
```

Yes.

Return it.

---

### If Not Found

```js
user.toString();
```

JavaScript checks:

```text
user
```

↓

Not found

↓

Prototype

```text
Object.prototype
```

↓

Found

↓

Execute

---

### Visualization

```text
user
 ↓
Object.prototype
 ↓
null
```

---

# How Lookup Works

Example:

```js
user.hasOwnProperty("name");
```

JavaScript searches:

```text
user
```

↓

Not found

↓

```text
Object.prototype
```

↓

Found

---

This searching process is called:

```text
Prototype Chain Lookup
```

---

# Important Interview Statement

When JavaScript cannot find a property on an object:

It walks up the prototype chain until:

```text
Property Found
```

or

```text
null Reached
```

---

# 4. Prototypal Inheritance

Objects can inherit properties from other objects.

---

### Example

```js
const animal = {
  eats: true
};

const dog = Object.create(animal);

console.log(dog.eats);
```

Output:

```js
true
```

---

Why?

JavaScript searches:

```text
dog
```

↓

Not found

↓

```text
animal
```

↓

Found

---

### Visualization

```text
dog
 ↓
animal
 ↓
Object.prototype
 ↓
null
```

---

# Another Example

```js
const person = {
  greet() {
    console.log("Hello");
  }
};

const user = Object.create(person);

user.greet();
```

Output:

```js
Hello
```

---

This is:

```text
Prototypal Inheritance
```

---

# 5. Constructor Functions

Before ES6 Classes, constructor functions were used to create objects.

---

### Syntax

```js
function User(name) {
  this.name = name;
}
```

Create object:

```js
const user1 =
  new User("Vinay");
```

---

Output:

```js
{
  name: "Vinay"
}
```

---

# What Does new Do?

When you write:

```js
new User("Vinay");
```

JavaScript:

### Step 1

Creates empty object.

```js
{}
```

---

### Step 2

Sets prototype.

```js
obj.__proto__ =
User.prototype
```

---

### Step 3

Calls function.

```js
User.call(obj);
```

---

### Step 4

Returns object.

---

# Constructor Example

```js
function User(name) {
  this.name = name;
}

const user1 =
  new User("Vinay");

console.log(user1.name);
```

Output:

```js
Vinay
```

---

# Adding Methods Using Prototype

Bad:

```js
function User(name) {
  this.name = name;

  this.greet = function() {
    console.log("Hello");
  };
}
```

Creates new function for every object.

---

Better:

```js
function User(name) {
  this.name = name;
}

User.prototype.greet =
  function() {
    console.log("Hello");
  };
```

---

Now:

```js
const user1 =
  new User("Vinay");

const user2 =
  new User("John");
```

Both share:

```js
greet()
```

---

### Memory Efficient

```text
user1
      \
       → greet()
      /
user2
```

---

# 6. Constructor Function Inheritance

---

### Parent

```js
function Animal(name) {
  this.name = name;
}
```

---

Add method:

```js
Animal.prototype.eat =
  function() {
    console.log("Eating");
  };
```

---

### Child

```js
function Dog(name) {
  Animal.call(this, name);
}
```

---

Link prototypes:

```js
Dog.prototype =
  Object.create(
    Animal.prototype
  );
```

---

Example:

```js
const dog =
  new Dog("Tommy");

dog.eat();
```

Output:

```js
Eating
```

---

# 7. ES6 Classes

Introduced in ES6.

Classes are easier syntax over prototypes.

Important:

> Classes do NOT replace prototypes.

They are just cleaner syntax.

---

### Class Syntax

```js
class User {

  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(
      `Hello ${this.name}`
    );
  }

}
```

---

Create Object

```js
const user =
  new User("Vinay");

user.greet();
```

Output:

```js
Hello Vinay
```

---

# Behind The Scenes

JavaScript internally converts:

```js
class User {}
```

into prototype-based code.

---

### Equivalent Constructor Function

```js
function User(name) {
  this.name = name;
}

User.prototype.greet =
  function() {
    console.log(
      `Hello ${this.name}`
    );
  };
```

---

Same result.

Different syntax.

---

# 8. Class Inheritance

ES6 provides:

```js
extends
```

for inheritance.

---

### Parent Class

```js
class Animal {

  eat() {
    console.log("Eating");
  }

}
```

---

### Child Class

```js
class Dog extends Animal {

  bark() {
    console.log("Woof");
  }

}
```

---

Usage

```js
const dog = new Dog();

dog.eat();
dog.bark();
```

Output:

```js
Eating
Woof
```

---

# Using super()

Used to call parent constructor.

---

### Parent

```js
class Animal {

  constructor(name) {
    this.name = name;
  }

}
```

---

### Child

```js
class Dog extends Animal {

  constructor(name) {

    super(name);

  }

}
```

---

Why?

```js
super()
```

calls parent constructor.

---

# 9. Class vs Prototype

| Class | Prototype |
|---------|-----------|
| Cleaner syntax | Older syntax |
| Easier to read | More verbose |
| Uses prototype internally | Actual implementation |
| Modern JavaScript | Traditional JavaScript |

---

Example:

### Class

```js
class User {
  greet() {}
}
```

---

### Prototype

```js
function User() {}

User.prototype.greet =
function() {};
```

---

Same concept.

Different syntax.

---

# Internal Prototype Chain Example

```js
const user =
  new User("Vinay");
```

---

Visualization:

```text
user
 ↓
User.prototype
 ↓
Object.prototype
 ↓
null
```

---

# Common Interview Questions

---

## Q1: What is a Prototype?

An object from which other objects inherit properties and methods.

---

## Q2: What is the Prototype Chain?

The chain JavaScript follows while looking for a property.

---

Example:

```text
Object
 ↓
Prototype
 ↓
Prototype
 ↓
null
```

---

## Q3: What is Prototypal Inheritance?

Objects inheriting properties and methods from other objects through the prototype chain.

---

## Q4: What is a Constructor Function?

A function used with `new` to create objects.

```js
function User(name) {
  this.name = name;
}
```

---

## Q5: What does `new` do?

1. Creates object
2. Links prototype
3. Executes constructor
4. Returns object

---

## Q6: Are Classes Different From Prototypes?

No.

Classes are syntax sugar over prototypes.

---

## Q7: What is `extends`?

Used to inherit from another class.

```js
class Dog extends Animal {}
```

---

## Q8: What is `super()`?

Calls the parent constructor.

---

# Interview Quick Revision 🚀

## Prototype

```js
Array.prototype
Object.prototype
```

Shared methods live here.

---

## Prototype Chain

```text
Object
 ↓
Prototype
 ↓
Prototype
 ↓
null
```

---

## Prototypal Inheritance

```js
Object.create(parent);
```

---

## Constructor Function

```js
function User(name) {
  this.name = name;
}
```

---

## Create Object

```js
const user =
  new User("Vinay");
```

---

## Add Shared Methods

```js
User.prototype.greet =
function() {};
```

---

## ES6 Class

```js
class User {

  constructor(name) {
    this.name = name;
  }

}
```

---

## Inheritance

```js
class Dog extends Animal {}
```

---

## Parent Constructor

```js
super();
```

---

# Most Important Interview Diagram ⭐

```text
const user = new User()

user
 ↓
User.prototype
 ↓
Object.prototype
 ↓
null
```

If a property is not found:

```text
user
 ↓
User.prototype
 ↓
Object.prototype
 ↓
null
```

JavaScript keeps searching upward.

This process is called:

```text
Prototype Chain Lookup
```

---

# Golden Interview Tips ⭐

1. Every object in JavaScript has a prototype.
2. Arrays inherit from `Array.prototype`.
3. Objects inherit from `Object.prototype`.
4. Classes are just a cleaner syntax over prototypes.
5. `new` creates an object and links prototypes.
6. Shared methods should be placed on prototypes.
7. Understand the prototype chain diagram thoroughly.
8. Be able to explain why JavaScript is called a **Prototype-Based Language**.
9. Memorize:

```text
Object
 ↓
Prototype
 ↓
Prototype
 ↓
null
```

10. If asked "How does inheritance work in JavaScript?" answer:

> JavaScript uses **Prototypal Inheritance**, where objects inherit properties and methods through the prototype chain.

🚀 Master Prototype + Scope + Closures + this = You've covered most advanced JavaScript interview fundamentals.