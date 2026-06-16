# JavaScript Browser APIs – Interview Revision Guide 🚀

⚠️ **Browser APIs are frequently asked in Frontend, React, JavaScript, and Full Stack interviews.**

JavaScript alone cannot:

- Access HTML elements
- Store data in the browser
- Get user location
- Make network requests
- Open WebSocket connections

The browser provides special APIs for these tasks.

---

# Table of Contents

1. What are Browser APIs?
2. DOM (Document Object Model)
3. BOM (Browser Object Model)
4. localStorage
5. sessionStorage
6. fetch API
7. WebSockets
8. Geolocation API
9. Event APIs
10. Common Interview Questions
11. Interview Quick Revision
12. Custom Events

---

# 1. What are Browser APIs?

Browser APIs are features provided by the browser that JavaScript can use.

Examples:

```text
DOM
localStorage
fetch
WebSockets
Geolocation
Events
```

---

### Visualization

```text
JavaScript
     ↓
Browser APIs
     ↓
Browser Features
```

---

Example:

```js
document.getElementById()

fetch()

localStorage.setItem()
```

These are not part of core JavaScript.

They are provided by the browser.

---

# 2. DOM (Document Object Model)

Most important Browser API.

---

## What is DOM?

DOM is a JavaScript representation of an HTML page.

---

### HTML

```html
<h1>Hello</h1>
```

---

### DOM

```text
Document
   ↓
  h1
```

Browser converts HTML into a tree structure.

---

## Why DOM?

Allows JavaScript to:

- Read HTML
- Modify HTML
- Add Elements
- Remove Elements

---

# Selecting Elements

---

## getElementById()

```js
const title =
  document.getElementById("title");
```

---

## querySelector()

```js
const title =
  document.querySelector("#title");
```

---

## querySelectorAll()

```js
const items =
  document.querySelectorAll(".item");
```

---

# Changing Content

```js
title.textContent =
  "Welcome";
```

---

Output:

```html
<h1>Welcome</h1>
```

---

# Changing HTML

```js
title.innerHTML =
  "<span>Hello</span>";
```

---

# Changing Styles

```js
title.style.color = "red";
```

---

# Creating Elements

```js
const div =
  document.createElement("div");
```

---

# Adding Elements

```js
document.body.appendChild(div);
```

---

# DOM Tree Example

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

# 3. BOM (Browser Object Model)

BOM provides access to browser features.

---

## What is BOM?

Allows interaction with:

- Browser Window
- URL
- History
- Navigator

---

### Main Object

```js
window
```

---

# Browser Window

```js
window.alert("Hello");
```

---

```js
window.confirm("Delete?");
```

---

```js
window.prompt("Enter Name");
```

---

# Location Object

Current URL information.

```js
console.log(
  window.location.href
);
```

---

Output:

```text
https://example.com
```

---

# Redirect User

```js
window.location.href =
  "https://google.com";
```

---

# Navigator Object

Browser information.

```js
console.log(
  navigator.userAgent
);
```

---

# History Object

Navigate browser history.

```js
history.back();
```

---

```js
history.forward();
```

---

# BOM Overview

```text
window
 ├── location
 ├── history
 ├── navigator
 └── screen
```

---

# 4. localStorage

Most asked storage API.

---

## What is localStorage?

Stores data in browser permanently.

Data remains after:

```text
Refresh
Close Browser
Restart Browser
```

---

# Save Data

```js
localStorage.setItem(
  "name",
  "Vinay"
);
```

---

# Get Data

```js
localStorage.getItem(
  "name"
);
```

Output:

```js
Vinay
```

---

# Remove Data

```js
localStorage.removeItem(
  "name"
);
```

---

# Clear All

```js
localStorage.clear();
```

---

# Store Objects

localStorage stores strings only.

---

### Save Object

```js
const user = {
  name: "Vinay"
};

localStorage.setItem(
  "user",
  JSON.stringify(user)
);
```

---

### Read Object

```js
const user =
JSON.parse(
  localStorage.getItem("user")
);
```

---

# 5. sessionStorage

Similar to localStorage.

---

## Difference

Data is removed when browser tab closes.

---

### Save Data

```js
sessionStorage.setItem(
  "name",
  "Vinay"
);
```

---

### Get Data

```js
sessionStorage.getItem(
  "name"
);
```

---

# localStorage vs sessionStorage

| localStorage | sessionStorage |
|--------------|----------------|
| Persistent | Temporary |
| Survives restart | Removed on tab close |
| Shared across tabs | Tab specific |

---

# 6. fetch API

Most important API for frontend interviews.

Used to make HTTP requests.

---

# GET Request

```js
fetch(
  "https://api.example.com/users"
)
.then(response =>
  response.json()
)
.then(data => {
  console.log(data);
});
```

---

# Async/Await Version

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

# POST Request

```js
fetch(url, {
  method: "POST",

  headers: {
    "Content-Type":
      "application/json"
  },

  body: JSON.stringify({
    name: "Vinay"
  })
});
```

---

# Common HTTP Methods

```text
GET
POST
PUT
PATCH
DELETE
```

---

# 7. WebSockets

Used for real-time communication.

---

## Why Not fetch?

fetch:

```text
Request
 ↓
Response
```

One-time communication.

---

WebSocket:

```text
Client ↔ Server
```

Continuous communication.

---

# Create Connection

```js
const socket =
  new WebSocket(
    "ws://localhost:3000"
  );
```

---

# Connection Open

```js
socket.onopen = () => {
  console.log("Connected");
};
```

---

# Receive Message

```js
socket.onmessage =
(event) => {
  console.log(event.data);
};
```

---

# Send Message

```js
socket.send("Hello");
```

---

# Real Uses

```text
Chat Apps
Stock Market
Live Scores
Notifications
Online Games
```

---

# WebSocket Flow

```text
Browser
    ↕
WebSocket
    ↕
Server
```

---

# 8. Geolocation API

Allows browser to access user location.

---

⚠️ Requires user permission.

---

# Get Current Location

```js
navigator.geolocation
  .getCurrentPosition(
    position => {

      console.log(
        position.coords.latitude
      );

      console.log(
        position.coords.longitude
      );

    }
  );
```

---

Output Example

```text
17.3850
78.4867
```

---

# Real Uses

```text
Maps
Food Delivery
Ride Sharing
Location Tracking
Weather Apps
```

---

# Geolocation Flow

```text
Browser
 ↓
Permission
 ↓
Location
```

---

# 9. Event APIs

One of the most used Browser APIs.

---

## What is an Event?

An action performed by user or browser.

Examples:

```text
Click
Input
Submit
Scroll
Mouse Move
Key Press
```

---

# Add Event Listener

```js
button.addEventListener(
  "click",
  () => {
    console.log("Clicked");
  }
);
```

---

# Common Events

### Click

```js
button.addEventListener(
  "click",
  handler
);
```

---

### Input

```js
input.addEventListener(
  "input",
  handler
);
```

---

### Change

```js
input.addEventListener(
  "change",
  handler
);
```

---

### Submit

```js
form.addEventListener(
  "submit",
  handler
);
```

---

### Keydown

```js
document.addEventListener(
  "keydown",
  handler
);
```

---

# Event Object

```js
button.addEventListener(
  "click",
  event => {
    console.log(event);
  }
);
```

---

Access:

```js
event.target
```

---

Example

```js
button.addEventListener(
  "click",
  event => {

    console.log(
      event.target
    );

  }
);
```

---

# Prevent Default

Useful in forms.

```js
form.addEventListener(
  "submit",
  event => {

    event.preventDefault();

  }
);
```

---

Stops page refresh.

---

# Common Interview Questions

---

## Q1: What is DOM?

A JavaScript representation of an HTML document.

---

## Q2: What is BOM?

Browser APIs provided through the `window` object.

---

## Q3: Difference Between DOM and BOM?

### DOM

```text
HTML Document
```

---

### BOM

```text
Browser Window
```

---

## Q4: Difference Between localStorage and sessionStorage?

### localStorage

Persistent.

---

### sessionStorage

Cleared when tab closes.

---

## Q5: Does localStorage Store Objects?

No.

Must use:

```js
JSON.stringify()
```

and

```js
JSON.parse()
```

---

## Q6: What is fetch?

API used for HTTP requests.

---

## Q7: What is WebSocket?

Persistent two-way communication between client and server.

---

## Q8: Why Use WebSockets Instead of fetch?

Because WebSockets support real-time communication.

---

## Q9: Does Geolocation Require Permission?

Yes.

Browser asks user permission.

---

## Q10: What is addEventListener()?

Method used to attach event handlers.

---

# Interview Quick Revision 🚀

## DOM

```js
document
```

Manipulates HTML.

---

## BOM

```js
window
```

Controls browser features.

---

## localStorage

```js
localStorage.setItem()
```

Permanent storage.

---

## sessionStorage

```js
sessionStorage.setItem()
```

Removed on tab close.

---

## fetch API

```js
fetch(url)
```

Makes HTTP requests.

---

## WebSocket

```js
new WebSocket(url)
```

Real-time communication.

---

## Geolocation

```js
navigator.geolocation
```

Gets user location.

---

## Events

```js
addEventListener()
```

Handles user actions.

---


## Event

```js
new Event("login")
```

Simple custom event.

---

## CustomEvent

```js
new CustomEvent(
  "login",
  {
    detail: {
      name: "Vinay"
    }
  }
)
```

Custom event with data.

---

## Trigger Event

```js
dispatchEvent()
```

---

## Listen Event

```js
addEventListener()
```

---

## Access Data

```js
event.detail
```

# Most Important Interview Diagram ⭐

```text
Browser APIs

DOM
 ↓
HTML Manipulation

BOM
 ↓
Browser Features

Storage
 ↓
localStorage
sessionStorage

Network
 ↓
fetch
WebSocket

Location
 ↓
Geolocation

Events
 ↓
addEventListener()
```

---

# Golden Interview Tips ⭐

1. DOM is used to manipulate HTML elements.
2. BOM provides browser-related features through `window`.
3. localStorage persists even after browser restart.
4. sessionStorage is cleared when the tab closes.
5. fetch is used for HTTP requests.
6. WebSockets are used for real-time communication.
7. Geolocation requires user permission.
8. Most user interactions are handled using Event APIs.
9. Know the difference between:
   ```text
   fetch → Request/Response

   WebSocket → Continuous Connection
   ```
10. Browser APIs are heavily used in React, Angular, Vue, and modern web applications.

🚀 Master Browser APIs and you'll be comfortable building real-world frontend applications and answering practical interview questions.