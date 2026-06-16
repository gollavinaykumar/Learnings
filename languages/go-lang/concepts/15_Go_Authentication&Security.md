# Go Authentication & Security – Complete Interview Revision Guide

> Authentication and Security are critical for backend development.
>
> Almost every production API requires:
>
> - Authentication
> - Authorization
> - Password Security
> - API Protection
> - Attack Prevention
>
> Common interview topics:
>
> - JWT
> - OAuth2
> - API Keys
> - bcrypt
> - CORS
> - CSRF
> - Rate Limiting

---

# 1. Authentication vs Authorization

Most Asked Interview Question.

---

# Authentication

Authentication answers:

```text
Who are you?
```

Example:

```text
Username + Password
JWT Token
Google Login
```

---

# Authorization

Authorization answers:

```text
What are you allowed to do?
```

Example:

```text
Admin
User
Moderator
```

---

# Example

User logs in.

Authentication:

```text
User identity verified
```

---

Authorization:

```text
Can user delete accounts?
```

---

# Simple Interview Answer

```text
Authentication = Identity Verification

Authorization = Permission Verification
```

---

# 2. JWT (JSON Web Token)

Most Important Authentication Topic.

---

# What is JWT?

JWT is a token-based authentication mechanism.

Instead of storing sessions on server:

```text
Server issues token
Client stores token
Client sends token on every request
```

---

# JWT Flow

```text
Login
  ↓
Verify Credentials
  ↓
Generate JWT
  ↓
Send JWT
  ↓
Client Stores JWT
  ↓
Client Sends JWT
  ↓
Protected API Access
```

---

# JWT Structure

A JWT contains 3 parts.

```text
Header
Payload
Signature
```

---

Visual:

```text
xxxxx.yyyyy.zzzzz
```

---

# Header

Contains algorithm.

Example:

```json
{
  "alg":"HS256",
  "typ":"JWT"
}
```

---

# Payload

Contains claims.

Example:

```json
{
  "user_id":101,
  "role":"admin"
}
```

---

# Signature

Used to verify token integrity.

---

# JWT Example

Payload:

```json
{
  "user_id":101,
  "email":"vinay@gmail.com"
}
```

---

# Generate JWT

Common Library:

```go
github.com/golang-jwt/jwt/v5
```

---

Example

```go
token := jwt.NewWithClaims(
    jwt.SigningMethodHS256,
    jwt.MapClaims{
        "user_id": 101,
    },
)
```

---

# Sign Token

```go
signedToken, err :=
    token.SignedString(
        []byte(secretKey),
    )
```

---

# Verify JWT

```go
token, err :=
    jwt.Parse(
        tokenString,
        func(
            token *jwt.Token,
        ) (interface{}, error) {

            return []byte(secretKey), nil
        },
    )
```

---

# Why JWT?

Benefits:

```text
Stateless
Fast
Scalable
No Session Storage
```

---

# JWT Best Practices

---

## Use Expiration

```go
"exp"
```

claim.

---

## Strong Secret Key

Bad:

```go
secret
```

Good:

```text
Long Random Secret
```

---

## Use HTTPS

Never send JWT over HTTP.

---

## Short Token Lifetime

Example:

```text
15 Minutes
1 Hour
```

---

# Access Token vs Refresh Token

Very Common Interview Question.

---

## Access Token

```text
Short-lived
```

Example:

```text
15 Minutes
```

---

## Refresh Token

```text
Long-lived
```

Example:

```text
30 Days
```

Used to generate new access tokens.

---

# 3. OAuth2

Very Important Modern Authentication Topic.

---

# What is OAuth2?

OAuth2 allows users to login using:

```text
Google
GitHub
Facebook
Microsoft
```

without sharing passwords.

---

# Example

```text
Login With Google
```

---

# OAuth Flow

```text
User
 ↓
Google Login
 ↓
Google Verifies User
 ↓
Authorization Code
 ↓
Access Token
 ↓
Application
```

---

# Why OAuth2?

Benefits:

```text
No Password Storage
Better Security
Easy User Experience
```

---

# Example

```text
Sign In With Google
```

uses OAuth2.

---

# Common Terms

---

## Client

Your application.

---

## Resource Owner

User.

---

## Authorization Server

Google.

---

## Access Token

Proof of authentication.

---

# OAuth vs JWT

| Feature | JWT | OAuth2 |
|----------|------|---------|
| Authentication | Yes | Yes |
| Third-Party Login | No | Yes |
| Google Login | No | Yes |
| Stateless | Yes | Usually |

---

# 4. API Keys

Simple authentication mechanism.

---

# What is API Key?

Unique secret string used to identify clients.

---

Example:

```text
x-api-key:
abc123xyz
```

---

# Flow

```text
Request
   ↓
API Key
   ↓
Validation
   ↓
Response
```

---

# Example

```http
GET /users

x-api-key: abc123
```

---

# Verification

```go
apiKey :=
    r.Header.Get(
        "x-api-key",
    )
```

---

# Use Cases

```text
Internal Services
Third-Party APIs
Partner Integrations
```

---

# API Key Limitations

Cannot identify users.

Only identifies:

```text
Applications
Clients
Services
```

---

# 5. bcrypt Password Hashing

Most Important Password Topic.

---

# Never Store Passwords

Bad:

```text
password123
```

inside database.

---

If database leaks:

```text
All passwords exposed
```

---

# Solution

Hash passwords.

---

# What is Hashing?

Convert:

```text
password123
```

into:

```text
$2a$10$...
```

---

# Common Library

```go
golang.org/x/crypto/bcrypt
```

---

# Generate Hash

```go
hash, err :=
    bcrypt.GenerateFromPassword(
        []byte(password),
        bcrypt.DefaultCost,
    )
```

---

# Store

```go
string(hash)
```

inside database.

---

# Verify Password

```go
err :=
    bcrypt.CompareHashAndPassword(
        []byte(hash),
        []byte(password),
    )
```

---

# Why bcrypt?

Features:

```text
Salted
Slow By Design
Resistant To Brute Force
```

---

# Password Flow

```text
Register
   ↓
Hash Password
   ↓
Store Hash
```

---

Login:

```text
User Password
   ↓
Compare With Hash
   ↓
Success
```

---

# 6. CORS

Very Common Frontend + Backend Interview Topic.

---

# What is CORS?

Cross-Origin Resource Sharing.

Browser security mechanism.

---

# Problem

Frontend:

```text
localhost:3000
```

Backend:

```text
localhost:8080
```

Different origins.

Browser blocks request.

---

# Solution

Enable CORS.

---

# Example Header

```http
Access-Control-Allow-Origin:
*
```

---

# Go Example

```go
w.Header().
Set(
 "Access-Control-Allow-Origin",
 "*",
)
```

---

# Better Production Config

```text
Allow Specific Domains
```

Example:

```text
https://myapp.com
```

---

# Why CORS Exists?

Protect users from malicious websites.

---

# 7. CSRF

Cross-Site Request Forgery.

Very Common Security Interview Question.

---

# What is CSRF?

Attack where a user unknowingly performs an action.

---

# Example

User logged into:

```text
bank.com
```

---

Visits malicious site.

Site triggers:

```text
Transfer Money
```

using user's session.

---

# Why Does It Work?

Browser automatically sends:

```text
Cookies
```

---

# Prevention

---

## CSRF Tokens

Generate unique token.

Verify every request.

---

## SameSite Cookies

```http
SameSite=Strict
```

---

## Double Submit Cookies

Common technique.

---

# JWT vs CSRF

Important Interview Question.

---

## Session Cookies

Vulnerable to:

```text
CSRF
```

---

## JWT In Authorization Header

Usually safer.

Example:

```http
Authorization:
Bearer token
```

---

# 8. Rate Limiting

Very Important Production Topic.

---

# What is Rate Limiting?

Restricts how many requests a client can make.

---

# Example

```text
100 Requests / Minute
```

---

# Why Needed?

Prevents:

```text
Abuse
DDoS
Brute Force
Spam
```

---

# Example

Without limit:

```text
1 Million Requests
```

possible.

---

With limit:

```text
100 Requests/Minute
```

allowed.

---

# Rate Limiting Flow

```text
Request
   ↓
Count Requests
   ↓
Limit Reached?
   ↓
Reject
```

---

# Response

```http
429 Too Many Requests
```

---

# Common Algorithms

---

## Fixed Window

```text
100 Requests/Minute
```

Simple.

---

## Sliding Window

More accurate.

---

## Token Bucket

Most popular.

---

# Example

Bucket:

```text
100 Tokens
```

Each request consumes token.

---

# Go Library

```go
golang.org/x/time/rate
```

---

# Example

```go
limiter :=
    rate.NewLimiter(
        1,
        5,
    )
```

Meaning:

```text
1 request/sec
Burst 5
```

---

# Security Best Practices

---

## Use HTTPS

Never use plain HTTP.

---

## Hash Passwords

Use:

```go
bcrypt
```

---

## Validate Input

Prevent:

```text
SQL Injection
XSS
```

---

## Use JWT Expiration

Always set:

```go
exp
```

claim.

---

## Rate Limit APIs

Prevent abuse.

---

## Secure Secrets

Store in:

```text
Environment Variables
Secret Managers
```

---

Not:

```go
hardcoded strings
```

---

# Most Asked Interview Questions

---

## Authentication vs Authorization?

Authentication:

```text
Who are you?
```

Authorization:

```text
What can you do?
```

---

## What is JWT?

Stateless authentication token containing claims.

---

## JWT Structure?

```text
Header
Payload
Signature
```

---

## What is OAuth2?

Protocol allowing third-party authentication.

Example:

```text
Login With Google
```

---

## What is API Key?

Secret identifier for applications or services.

---

## Why Use bcrypt?

Secure password hashing.

---

## What is CORS?

Browser mechanism controlling cross-origin requests.

---

## What is CSRF?

Attack that tricks users into performing unwanted actions.

---

## How Prevent CSRF?

- CSRF Tokens
- SameSite Cookies
- JWT Headers

---

## What is Rate Limiting?

Restricting number of requests to prevent abuse.

---

# Quick Revision Cheat Sheet

## JWT

```text
Header
Payload
Signature
```

---

## OAuth2

```text
Google Login
GitHub Login
```

---

## API Key

```http
x-api-key:
abc123
```

---

## Hash Password

```go
bcrypt.GenerateFromPassword()
```

---

## Verify Password

```go
bcrypt.CompareHashAndPassword()
```

---

## CORS

```http
Access-Control-Allow-Origin
```

---

## CSRF Protection

```text
CSRF Token
SameSite Cookie
```

---

## Rate Limiting

```go
rate.NewLimiter()
```

---

## HTTP Status

```http
429 Too Many Requests
```

---

# Golden Interview Answer

### How Do You Secure a Production Go API?

A secure Go API typically uses:

- bcrypt for password hashing
- JWT or OAuth2 for authentication
- Authorization checks for permissions
- CORS configuration
- CSRF protection (for cookie-based auth)
- Rate limiting to prevent abuse
- HTTPS for encrypted communication

Together, these mechanisms protect user identities, API endpoints, and backend services from common security threats.

---

# End of Authentication & Security Revision