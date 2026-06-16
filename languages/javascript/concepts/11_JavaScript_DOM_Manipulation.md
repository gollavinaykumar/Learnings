# JavaScript DOM Manipulation – Interview Revision Guide 🚀

⚠️ **DOM Manipulation is one of the most important Frontend interview topics.**

Almost every JavaScript and React interview contains questions related to:

- querySelector()
- Event Listeners
- Event Bubbling
- Event Delegation
- Reflow & Repaint

Understanding DOM Manipulation helps you build interactive web applications.

---

# Table of Contents

1. What is DOM Manipulation?
2. querySelector()
3. Event Listeners
4. Event Object
5. Event Bubbling
6. Event Capturing
7. Event Delegation
8. Reflow
9. Repaint
10. Reflow vs Repaint
11. Common Interview Questions
12. Interview Quick Revision

---

# 1. What is DOM Manipulation?

DOM Manipulation means:

> Using JavaScript to read, modify, create, or remove HTML elements.

---

### Example HTML

```html
<h1 id="title">
  Hello
</h1>
```

---

### JavaScript

```js
const title =
  document.getElementById(
    "title"
  );

title.textContent =
  "Welcome";
```

---

Result

```html
<h1 id="title">
  Welcome
</h1>
```

---

# DOM Tree

```html
<body>
  <h1>Hello</h1>
  <p>World</p>
</body>
```

---

Visualization

```text
Document
   ↓
  body
  ├── h1
  └── p
```

---

# 2. querySelector()

Most commonly used DOM selector.

---

## What is querySelector()?

Returns the FIRST matching element.

---

### Select by ID

```js
const title =
  document.querySelector(
    "#title"
  );
```

---

### Select by Class

```js
const item =
  document.querySelector(
    ".item"
  );
```

---

### Select by Tag

```js
const heading =
  document.querySelector(
    "h1"
  );
```

---

# querySelectorAll()

Returns ALL matching elements.

---

Example

```js
const items =
  document.querySelectorAll(
    ".item"
  );
```

---

Output

```js
NodeList
```

---

Example

```js
items.forEach(item => {
  console.log(item);
});
```

---

# Difference

| Method | Returns |
|----------|----------|
| querySelector() | First Match |
| querySelectorAll() | All Matches |

---

# 3. Event Listeners

Events allow JavaScript to respond to user actions.

---

Examples:

```text
click
input
change
submit
keydown
mouseover
```

---

## addEventListener()

Used to attach event handlers.

---

### Syntax

```js
element.addEventListener(
  event,
  handler
);
```

---

### Example

```js
button.addEventListener(
  "click",
  () => {
    console.log("Clicked");
  }
);
```

---

Output

```js
Clicked
```

---

# Multiple Listeners

```js
button.addEventListener(
  "click",
  firstHandler
);

button.addEventListener(
  "click",
  secondHandler
);
```

Both run.

---

# Remove Listener

```js
button.removeEventListener(
  "click",
  handler
);
```

---

# 4. Event Object

Every event handler receives an event object.

---

Example

```js
button.addEventListener(
  "click",
  event => {

    console.log(event);

  }
);
```

---

# Useful Properties

### event.target

Element that triggered event.

```js
console.log(
  event.target
);
```

---

### event.type

```js
console.log(
  event.type
);
```

Output:

```js
click
```

---

### event.currentTarget

Current element handling the event.

```js
console.log(
  event.currentTarget
);
```

---

### preventDefault()

Stops browser default behavior.

---

Example

```js
form.addEventListener(
  "submit",
  event => {

    event.preventDefault();

  }
);
```

---

Stops form refresh.

---

# 5. Event Bubbling

⚠️ Very common interview question.

---

## What is Event Bubbling?

An event starts at the target element and bubbles upward through parent elements.

---

### Example HTML

```html
<div id="parent">

  <button id="child">
    Click
  </button>

</div>
```

---

### JavaScript

```js
parent.addEventListener(
  "click",
  () => {
    console.log("Parent");
  }
);

child.addEventListener(
  "click",
  () => {
    console.log("Child");
  }
);
```

---

Click button.

Output:

```js
Child
Parent
```

---

# Flow

```text
Button
   ↑
Parent
   ↑
Body
   ↑
Document
```

---

This upward movement is called:

```text
Event Bubbling
```

---

# stopPropagation()

Stops bubbling.

---

Example

```js
child.addEventListener(
  "click",
  event => {

    event.stopPropagation();

    console.log("Child");

  }
);
```

---

Output:

```js
Child
```

---

Parent won't execute.

---

# 6. Event Capturing

Opposite of bubbling.

---

Flow:

```text
Document
 ↓
Body
 ↓
Parent
 ↓
Child
```

---

Enable capturing:

```js
parent.addEventListener(
  "click",
  handler,
  true
);
```

---

Third parameter:

```js
true
```

enables capture phase.

---

# Bubbling vs Capturing

### Bubbling

```text
Child → Parent
```

---

### Capturing

```text
Parent → Child
```

---

# 7. Event Delegation

⚠️ One of the MOST IMPORTANT interview topics.

---

## Problem

Imagine:

```html
<ul>
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
</ul>
```

---

Bad approach:

```js
item1.addEventListener(...);
item2.addEventListener(...);
item3.addEventListener(...);
```

---

If 1000 items exist:

```js
1000 listeners
```

---

# Better Solution

Use one listener on parent.

---

Example

```js
ul.addEventListener(
  "click",
  event => {

    console.log(
      event.target
    );

  }
);
```

---

Now:

```text
1 listener
```

handles all child clicks.

---

# Why Does This Work?

Because of:

```text
Event Bubbling
```

---

# Visualization

```text
li
 ↑
ul
 ↑
body
```

---

# Benefits

✅ Better performance

✅ Less memory usage

✅ Handles dynamic elements

---

# Real Example

```js
document
  .querySelector("#users")
  .addEventListener(
    "click",
    event => {

      if (
        event.target.tagName
        === "LI"
      ) {

        console.log(
          event.target.textContent
        );

      }

    }
  );
```

---

# 8. Reflow

⚠️ Advanced interview topic.

---

## What is Reflow?

When browser recalculates layout and element positions.

---

Example

```js
element.style.width =
  "500px";
```

---

Browser must recalculate:

```text
Width
Height
Position
Layout
```

---

This process is called:

```text
Reflow
```

---

# Reflow Triggers

### Width

```js
element.style.width
```

---

### Height

```js
element.style.height
```

---

### Margin

```js
element.style.margin
```

---

### Padding

```js
element.style.padding
```

---

### Adding Elements

```js
appendChild()
```

---

### Removing Elements

```js
remove()
```

---

# Reflow Is Expensive

Because browser recalculates layout.

---

# 9. Repaint

After layout calculation:

Browser redraws pixels.

This is called:

```text
Repaint
```

---

Example

```js
element.style.color =
  "red";
```

---

Layout doesn't change.

Only color changes.

---

Browser simply repaints.

---

# Repaint Triggers

### Color

```js
color
```

---

### Background

```js
background
```

---

### Visibility

```js
visibility
```

---

### Border Color

```js
border-color
```

---

# Repaint Is Cheaper

Compared to:

```text
Reflow
```

---

# 10. Reflow vs Repaint

| Reflow | Repaint |
|----------|----------|
| Recalculates layout | Redraws pixels |
| Expensive | Cheaper |
| Affects size/position | Affects appearance |
| Width/Height changes | Color changes |

---

### Reflow Example

```js
element.style.width =
  "500px";
```

---

### Repaint Example

```js
element.style.color =
  "red";
```

---

# Performance Tip

Avoid:

```js
for (...) {

  element.style.width = ...
}
```

Repeated layout changes.

---

Better:

```js
element.classList.add(
  "active"
);
```

Single update.

---

# Common Interview Questions

---

## Q1: Difference Between querySelector and querySelectorAll?

### querySelector()

Returns first match.

---

### querySelectorAll()

Returns all matches.

---

## Q2: What is Event Bubbling?

Events move upward from child to parent.

---

## Q3: What is Event Capturing?

Events move downward from parent to child.

---

## Q4: What is stopPropagation()?

Stops event bubbling.

---

## Q5: What is Event Delegation?

Using one parent listener to handle child events.

---

## Q6: Why Use Event Delegation?

- Better performance
- Fewer listeners
- Supports dynamic elements

---

## Q7: What is Reflow?

Layout recalculation.

---

## Q8: What is Repaint?

Redrawing pixels after visual changes.

---

## Q9: Which Is More Expensive?

```text
Reflow
```

Because layout must be recalculated.

---

# Interview Quick Revision 🚀

## Select Elements

```js
querySelector()
querySelectorAll()
```

---

## Event Listener

```js
addEventListener()
```

---

## Event Object

```js
event.target
event.type
event.currentTarget
```

---

## Stop Default Action

```js
event.preventDefault()
```

---

## Stop Bubbling

```js
event.stopPropagation()
```

---

## Event Bubbling

```text
Child
 ↑
Parent
 ↑
Document
```

---

## Event Capturing

```text
Document
 ↓
Parent
 ↓
Child
```

---

## Event Delegation

```js
Parent Listener
 +
event.target
```

---

## Reflow

```text
Layout Calculation
```

---

## Repaint

```text
Visual Redraw
```

---

# Most Important Interview Diagram ⭐

```text
Click Event

Child
  ↑
Parent
  ↑
Body
  ↑
Document
```

Event Bubbling.

---

### Event Delegation

```text
Many Children
       ↑
       │
One Parent Listener
```

---

### Browser Rendering

```text
DOM Change
    ↓
Reflow
    ↓
Repaint
```

---

# Golden Interview Tips ⭐

1. Learn `querySelector()` and `querySelectorAll()` thoroughly.
2. Understand Event Bubbling before learning Event Delegation.
3. Event Delegation is one of the most frequently asked frontend interview topics.
4. Use `event.target` in delegation questions.
5. Remember:
   ```text
   Bubbling = Bottom → Top
   Capturing = Top → Bottom
   ```
6. Reflow is more expensive than Repaint.
7. Too many DOM updates can hurt performance.
8. Use Event Delegation when handling large dynamic lists.
9. Know:
   ```js
   preventDefault()
   stopPropagation()
   ```
10. Most React event handling concepts are built on top of these DOM event fundamentals.

🚀 Master DOM Manipulation + Browser APIs + Event Loop and you'll have a strong foundation for React, Angular, Vue, and Frontend interviews.