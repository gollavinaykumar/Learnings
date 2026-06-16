# Go HTTP Servers & APIs – Complete Interview Revision Guide

> This is one of the most important backend development topics.
>
> Almost every Go backend application is an HTTP API.
>
> Common interview topics:
>
> - net/http
> - Routing
> - Middleware
> - JSON Encoding/Decoding
> - Request Lifecycle
> - Gin
> - Fiber
> - Echo

---

# 1. What is an HTTP Server?

An HTTP Server listens for requests and sends responses.

Example:

```text
Browser
    ↓
HTTP Request
    ↓
Go Server
    ↓
HTTP Response
```

---

# Go HTTP Package

Built-in package:

```go
import "net/http"
```

No framework required.

---

# Simplest HTTP Server

```go
package main

import (
    "fmt"
    "net/http"
)

func home(
    w http.ResponseWriter,
    r *http.Request,
) {

    fmt.Fprintln(w, "Hello World")
}

func main() {

    http.HandleFunc("/", home)

    http.ListenAndServe(":8080", nil)
}
```

Visit:

```text
http://localhost:8080
```

Output:

```text
Hello World
```

---

# Understanding HandleFunc

```go
http.HandleFunc("/", home)
```

Meaning:

```text
When "/" is requested,
execute home()
```

---

# Understanding ResponseWriter

```go
w http.ResponseWriter
```

Used to send response back.

Example:

```go
fmt.Fprintln(w, "Hello")
```

---

# Understanding Request

```go
r *http.Request
```

Contains request information.

Example:

```go
r.Method
r.URL
r.Header
r.Body
```

---

# Request Flow

```text
Client
   ↓
Router
   ↓
Handler
   ↓
Response
```

---

# 2. Routing

Most Important API Topic.

---

# What is Routing?

Routing maps URLs to handlers.

Example:

```text
GET /users
GET /products
POST /orders
```

Different routes.

Different handlers.

---

# Basic Routing

```go
http.HandleFunc("/", home)

http.HandleFunc("/users", users)

http.HandleFunc("/products", products)
```

---

# Example

```go
func users(
    w http.ResponseWriter,
    r *http.Request,
) {

    fmt.Fprintln(w, "Users")
}
```

---

# Route Matching

Request:

```text
GET /users
```

Go executes:

```go
users()
```

---

# REST API Routes

Common patterns:

```text
GET    /users
GET    /users/1

POST   /users

PUT    /users/1

DELETE /users/1
```

---

# Checking HTTP Method

```go
func users(
    w http.ResponseWriter,
    r *http.Request,
) {

    if r.Method == "GET" {
        fmt.Fprintln(w, "Get Users")
    }
}
```

---

# Better Way

```go
switch r.Method {

case http.MethodGet:

case http.MethodPost:
}
```

---

# Common HTTP Methods

| Method | Purpose |
|----------|---------|
| GET | Fetch Data |
| POST | Create Data |
| PUT | Update Data |
| PATCH | Partial Update |
| DELETE | Delete Data |

---

# 3. JSON Encoding

Most Asked API Interview Topic.

---

# Why JSON?

Most APIs communicate using JSON.

Example:

```json
{
  "name": "Vinay",
  "age": 25
}
```

---

# Struct

```go
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}
```

---

# JSON Encoding

Convert:

```go
Struct
```

to:

```json
JSON
```

---

## Example

```go
user := User{
    Name: "Vinay",
    Age: 25,
}

json.NewEncoder(w).
    Encode(user)
```

Output:

```json
{
  "name":"Vinay",
  "age":25
}
```

---

# Content-Type

Always set:

```go
w.Header().
    Set(
        "Content-Type",
        "application/json",
    )
```

---

# Complete Example

```go
func getUser(
    w http.ResponseWriter,
    r *http.Request,
) {

    user := User{
        Name: "Vinay",
        Age: 25,
    }

    w.Header().
        Set(
            "Content-Type",
            "application/json",
        )

    json.NewEncoder(w).
        Encode(user)
}
```

---

# JSON Decoding

Convert:

```json
JSON
```

to:

```go
Struct
```

---

# Example

Request Body:

```json
{
  "name":"Vinay",
  "age":25
}
```

---

## Decode

```go
var user User

err := json.NewDecoder(
    r.Body,
).Decode(&user)
```

---

# Result

```go
user.Name
user.Age
```

available.

---

# Most Common Pattern

```go
var user User

if err :=
    json.NewDecoder(
        r.Body,
    ).Decode(&user); err != nil {

    http.Error(
        w,
        "Invalid JSON",
        http.StatusBadRequest,
    )

    return
}
```

---

# 4. Request Lifecycle

Very Important Backend Topic.

---

# What Happens When Request Arrives?

---

## Step 1

Client sends request.

```text
Browser
Postman
Frontend
```

---

## Step 2

Server receives request.

---

## Step 3

Router matches route.

Example:

```text
/users
```

---

## Step 4

Middleware executes.

---

## Step 5

Handler executes.

---

## Step 6

Business Logic runs.

---

## Step 7

Database accessed.

---

## Step 8

Response returned.

---

# Visual

```text
Client
  ↓
Router
  ↓
Middleware
  ↓
Handler
  ↓
Service
  ↓
Database
  ↓
Response
```

---

# Real Backend Flow

```text
HTTP Request
      ↓
Auth Middleware
      ↓
Logging Middleware
      ↓
Handler
      ↓
Service Layer
      ↓
Repository Layer
      ↓
Database
      ↓
JSON Response
```

---

# 5. Middleware

Extremely Important Interview Topic.

---

# What is Middleware?

Middleware is code that runs:

```text
Before
or
After
```

the handler.

---

# Example Uses

```text
Authentication
Authorization
Logging
Rate Limiting
CORS
Request Tracking
```

---

# Middleware Flow

```text
Request
   ↓
Middleware
   ↓
Handler
   ↓
Response
```

---

# Example

```go
func Logger(
    next http.Handler,
) http.Handler {

    return http.HandlerFunc(
        func(
            w http.ResponseWriter,
            r *http.Request,
        ) {

            fmt.Println(
                r.Method,
                r.URL.Path,
            )

            next.ServeHTTP(w, r)
        },
    )
}
```

---

# Visual

```text
Request
   ↓
Logger
   ↓
Handler
```

---

# Why Middleware?

Avoid duplicate code.

Without middleware:

```go
func userHandler() {
    logRequest()
}
```

---

```go
func orderHandler() {
    logRequest()
}
```

Repeated everywhere.

---

Middleware centralizes it.

---

# 6. Gin Framework

Most Popular Go Web Framework.

---

## Import

```go
import "github.com/gin-gonic/gin"
```

---

# Create Router

```go
r := gin.Default()
```

---

# Route

```go
r.GET(
    "/",
    func(c *gin.Context) {

        c.JSON(
            200,
            gin.H{
                "message": "hello",
            },
        )
    },
)
```

---

# Run

```go
r.Run(":8080")
```

---

# Why Gin?

Features:

```text
Fast
Simple
Middleware Support
Validation
JSON Handling
Large Community
```

---

# Example Response

```json
{
  "message": "hello"
}
```

---

# Path Parameters

```go
r.GET("/users/:id",
    func(c *gin.Context) {

        id := c.Param("id")

        c.JSON(
            200,
            gin.H{
                "id": id,
            },
        )
})
```

Request:

```text
/users/10
```

Output:

```json
{
  "id":"10"
}
```

---

# 7. Fiber Framework

Inspired by Express.js.

---

## Import

```go
import "github.com/gofiber/fiber/v2"
```

---

# Create App

```go
app := fiber.New()
```

---

# Route

```go
app.Get("/",
    func(c *fiber.Ctx) error {

        return c.JSON(
            fiber.Map{
                "message": "hello",
            },
        )
})
```

---

# Run

```go
app.Listen(":8080")
```

---

# Why Fiber?

Features:

```text
Very Fast
Minimal
Express-Like Syntax
Easy To Learn
```

---

# Example

```go
app.Get("/users/:id")
```

Feels similar to:

```javascript
Express.js
```

---

# 8. Echo Framework

Another popular Go framework.

---

## Import

```go
import "github.com/labstack/echo/v4"
```

---

# Create Server

```go
e := echo.New()
```

---

# Route

```go
e.GET("/",
    func(c echo.Context) error {

        return c.JSON(
            200,
            map[string]string{
                "message": "hello",
            },
        )
})
```

---

# Run

```go
e.Start(":8080")
```

---

# Why Echo?

Features:

```text
Simple
Fast
Built-in Middleware
Validation Support
Clean API
```

---

# Gin vs Fiber vs Echo

Most Asked Interview Question.

| Feature | Gin | Fiber | Echo |
|----------|------|--------|------|
| Popularity | Highest | High | High |
| Learning Curve | Easy | Easy | Easy |
| Performance | Excellent | Excellent | Excellent |
| Community | Largest | Growing | Large |
| Enterprise Usage | Very Common | Common | Common |

---

# Interview Answer

For most production APIs:

```text
Gin
```

is the safest choice.

---

# HTTP Status Codes

Important for APIs.

---

## Success

```go
200 OK
201 Created
204 No Content
```

---

## Client Errors

```go
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

---

## Server Errors

```go
500 Internal Server Error
```

---

# Most Asked Interview Questions

---

## What is net/http?

Go's standard library package for building HTTP servers and clients.

---

## What is Routing?

Mapping URLs to handlers.

Example:

```go
/users -> userHandler
```

---

## What is Middleware?

Code executed before or after the handler.

Used for:

```text
Auth
Logging
Rate Limiting
```

---

## How Do You Encode JSON?

```go
json.NewEncoder(w).
    Encode(data)
```

---

## How Do You Decode JSON?

```go
json.NewDecoder(r.Body).
    Decode(&data)
```

---

## What is Request Lifecycle?

```text
Client
 ↓
Router
 ↓
Middleware
 ↓
Handler
 ↓
Database
 ↓
Response
```

---

## Most Popular Go Framework?

```text
Gin
```

---

## Why Use Gin?

- Fast
- Simple
- Production Ready
- Large Community

---

# Quick Revision Cheat Sheet

## Create Route

```go
http.HandleFunc("/", home)
```

---

## Start Server

```go
http.ListenAndServe(":8080", nil)
```

---

## JSON Encode

```go
json.NewEncoder(w).
    Encode(data)
```

---

## JSON Decode

```go
json.NewDecoder(r.Body).
    Decode(&data)
```

---

## Content Type

```go
w.Header().
    Set(
        "Content-Type",
        "application/json",
    )
```

---

## Middleware

```go
func Logger(
    next http.Handler,
)
```

---

## Gin

```go
r := gin.Default()
```

---

## Fiber

```go
app := fiber.New()
```

---

## Echo

```go
e := echo.New()
```

---

# Golden Interview Answer

### How Does an HTTP Request Flow Through a Go Backend?

```text
Client
  ↓
Router
  ↓
Middleware
  ↓
Handler
  ↓
Service Layer
  ↓
Repository Layer
  ↓
Database
  ↓
JSON Response
```

Go provides the `net/http` package for building servers, while frameworks like Gin, Fiber, and Echo add routing, middleware, validation, and developer productivity features.

For production APIs, Gin is currently the most commonly used Go web framework.

---

# End of HTTP Servers & APIs Revision