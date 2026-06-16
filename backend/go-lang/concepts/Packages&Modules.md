# Go Packages & Modules – Complete Interview Revision Guide

> Packages and Modules are the foundation of organizing Go applications.
>
> Every real-world Go backend project uses:
>
> - Packages
> - Modules
> - Dependencies
> - go.mod
>
> Understanding these concepts is essential for interviews and production development.

---

# 1. Packages

A package is a way to organize and reuse code.

Think of a package as a folder containing related Go files.

---

# Why Do We Need Packages?

Imagine everything is inside one file:

```go
main.go
```

As the project grows:

```text
10 files
50 files
100 files
```

Managing code becomes difficult.

Packages help organize code logically.

---

# Example Project

```text
project/
│
├── main.go
├── user/
│   └── user.go
├── database/
│   └── db.go
└── utils/
    └── helper.go
```

Each folder can be a package.

---

# Package Declaration

Every Go file starts with:

```go
package packageName
```

Example:

```go
package main
```

or

```go
package user
```

---

# Main Package

Every executable Go application starts with:

```go
package main
```

Example:

```go
package main

func main() {

}
```

---

# Why Special?

Go looks for:

```go
package main
```

and

```go
func main()
```

as the application entry point.

---

# Custom Package Example

Folder:

```text
user/
    user.go
```

File:

```go
package user

func GetUser() {
    fmt.Println("User Found")
}
```

---

# Importing Packages

Use:

```go
import
```

to use another package.

---

## Example

```go
import "fmt"
```

Now:

```go
fmt.Println("Hello")
```

works.

---

# Multiple Imports

```go
import (
    "fmt"
    "math"
)
```

---

# Custom Package Import

Project:

```text
myapp/
│
├── main.go
└── user/
    └── user.go
```

user.go

```go
package user

func GetUser() {
    fmt.Println("User Found")
}
```

main.go

```go
package main

import "myapp/user"

func main() {
    user.GetUser()
}
```

---

# Package Naming Rules

Good:

```go
user
config
database
service
```

Bad:

```go
UserPackage
MyUserPackage
```

Use:

```text
short
lowercase
meaningful
```

---

# Exported vs Unexported

One of the Most Asked Interview Questions.

---

# Exported

A name starting with Capital Letter is exported.

Meaning:

```text
Accessible from other packages
```

---

## Example

```go
package user

func GetUser() {
}
```

Because:

```go
GetUser
```

starts with:

```go
G
```

it is exported.

---

# Usage

```go
user.GetUser()
```

works.

---

# Unexported

A name starting with lowercase letter is unexported.

Meaning:

```text
Only accessible inside same package
```

---

## Example

```go
package user

func getUser() {
}
```

Usage:

```go
user.getUser()
```

Output:

```text
Compilation Error
```

---

# Exported Struct

```go
type User struct {
    Name string
}
```

Can be used outside package.

---

# Unexported Struct

```go
type user struct {
    Name string
}
```

Cannot be used outside package.

---

# Exported Fields

```go
type User struct {
    Name string
}
```

Accessible:

```go
u.Name
```

---

# Unexported Fields

```go
type User struct {
    name string
}
```

Not accessible outside package.

---

# Example

user package:

```go
package user

type User struct {
    Name string
    age  int
}
```

main package:

```go
u.Name
```

Works.

```go
u.age
```

Error.

---

# Export Rule Summary

Capital Letter:

```go
Name
GetUser
UserService
```

Exported.

---

Small Letter:

```go
name
getUser
userService
```

Unexported.

---

# Modules

Most Important Modern Go Concept.

---

# What is a Module?

A module is a collection of Go packages managed together.

Introduced in:

```text
Go 1.11
```

---

# Why Modules?

Before modules:

```text
GOPATH
```

was used.

Dependency management was difficult.

Modules solve:

- Dependency versioning
- Package management
- Reproducible builds

---

# Module Root

Every module contains:

```text
go.mod
```

---

# Example Project

```text
myapp/
│
├── go.mod
├── main.go
└── user/
```

---

# go.mod

Most Important File.

---

## Example

```go
module myapp

go 1.24
```

---

# Meaning

```go
module myapp
```

declares module name.

---

```go
go 1.24
```

declares Go version.

---

# Creating go.mod

Command:

```bash
go mod init myapp
```

---

# Example

```bash
mkdir myapp

cd myapp

go mod init myapp
```

Output:

```text
go: creating new go.mod
```

Generated:

```go
module myapp

go 1.24
```

---

# Dependency Management

Modern Go uses:

```go
go.mod
```

to manage dependencies.

---

# Example Dependency

Import:

```go
import "github.com/gin-gonic/gin"
```

When installed:

```go
go.mod
```

stores version information.

---

# go get

Used to download dependencies.

---

## Example

```bash
go get github.com/gin-gonic/gin
```

---

# What Happens?

Go:

1. Downloads package
2. Adds dependency
3. Updates go.mod
4. Updates go.sum

---

# Example go.mod

```go
require (
    github.com/gin-gonic/gin v1.10.0
)
```

---

# go.sum

Automatically generated.

Contains:

```text
Dependency checksums
```

Used for:

```text
Security
Integrity Verification
```

---

# Should We Commit go.sum?

Yes.

Always commit:

```text
go.mod
go.sum
```

to Git.

---

# go mod tidy

Most Used Go Command.

---

# Problem

Sometimes:

```go
go.mod
```

contains unused dependencies.

---

# Solution

Run:

```bash
go mod tidy
```

---

# What Does It Do?

1. Removes unused dependencies
2. Adds missing dependencies
3. Cleans go.mod
4. Updates go.sum

---

## Example

```bash
go mod tidy
```

---

# Real Interview Answer

If dependency list looks messy:

```bash
go mod tidy
```

fixes it.

---

# Common Workflow

---

## Create Project

```bash
mkdir myapp
```

---

## Initialize Module

```bash
go mod init myapp
```

---

## Install Dependency

```bash
go get github.com/gin-gonic/gin
```

---

## Clean Dependencies

```bash
go mod tidy
```

---

## Run Application

```bash
go run .
```

---

# Standard Library Packages

Go comes with many built-in packages.

---

## fmt

```go
import "fmt"
```

Used for printing.

---

## strings

```go
import "strings"
```

String operations.

---

## time

```go
import "time"
```

Date and time.

---

## net/http

```go
import "net/http"
```

HTTP servers and clients.

---

## encoding/json

```go
import "encoding/json"
```

JSON handling.

---

# Package Alias

Sometimes package names conflict.

Use alias.

---

## Example

```go
import f "fmt"
```

Usage:

```go
f.Println("Hello")
```

---

# Blank Import

Used for side effects only.

---

## Example

```go
import _ "github.com/lib/pq"
```

---

# Why?

Package initialization runs.

But package functions are not used directly.

Common in:

```text
Database Drivers
Plugins
```

---

# Dot Import (Rarely Used)

```go
import . "fmt"
```

Usage:

```go
Println("Hello")
```

instead of

```go
fmt.Println("Hello")
```

Not recommended.

Can make code confusing.

---

# Package Initialization

Special function:

```go
func init()
```

Runs automatically before main().

---

## Example

```go
func init() {
    fmt.Println("Initializing")
}
```

Output:

```text
Initializing
```

before:

```go
main()
```

runs.

---

# Most Asked Interview Questions

---

## What is a Package?

A package is a collection of related Go files.

---

## What is a Module?

A module is a collection of Go packages managed together using go.mod.

---

## What is go.mod?

The file that manages:

- Module name
- Go version
- Dependencies

---

## What is Exported vs Unexported?

Exported:

```go
Name
GetUser
UserService
```

Starts with uppercase letter.

Accessible outside package.

---

Unexported:

```go
name
getUser
userService
```

Starts with lowercase letter.

Accessible only within package.

---

## What Does go mod init Do?

Creates a new module and generates:

```go
go.mod
```

---

## What Does go get Do?

Downloads and installs dependencies.

---

## What Does go mod tidy Do?

- Removes unused dependencies
- Adds missing dependencies
- Cleans go.mod and go.sum

---

## What is go.sum?

Stores dependency checksums for integrity verification.

---

## Should go.mod and go.sum Be Committed?

Yes.

Always commit both.

---

# Quick Revision Cheat Sheet

## Package

```go
package user
```

---

## Import

```go
import "fmt"
```

---

## Exported Function

```go
func GetUser()
```

Accessible outside package.

---

## Unexported Function

```go
func getUser()
```

Accessible only inside package.

---

## Module Initialization

```bash
go mod init myapp
```

---

## Install Dependency

```bash
go get github.com/gin-gonic/gin
```

---

## Clean Dependencies

```bash
go mod tidy
```

---

## go.mod

```go
module myapp

go 1.24
```

---

## go.sum

Stores dependency checksums.

---

## init Function

```go
func init() {}
```

Runs before main().

---

# Golden Interview Answer

### Why Were Go Modules Introduced?

Go Modules were introduced to solve dependency management and versioning problems.

They provide:

- Reproducible builds
- Dependency version control
- Better project organization
- Independence from GOPATH

Modern Go development is completely module-based and revolves around:

```text
go.mod
go.sum
```

---

# End of Packages & Modules Revision