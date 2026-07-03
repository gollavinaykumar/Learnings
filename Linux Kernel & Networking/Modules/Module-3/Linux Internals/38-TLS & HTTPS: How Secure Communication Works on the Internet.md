Most developers type:

```text
https://google.com
```

instead of

```text
http://google.com
```

The website opens.

Everything works.

But very few developers understand what actually happens before the webpage loads.

Questions you may have:

- Why do we need HTTPS?
- What is TLS?
- What is SSL?
- How does encryption work?
- What is a certificate?
- Who issues certificates?
- What are Public Keys and Private Keys?
- What is the TLS Handshake?
- How does the browser know the server is real?
- How are session keys created?

The answer is:

**TLS (Transport Layer Security).**

TLS is one of the most important security protocols on the Internet.

Every secure website, API, cloud platform, Docker registry, Kubernetes API Server, and banking application depends on it.

After this chapter, you'll understand exactly how secure communication works from your browser to a server.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 38 — TLS & HTTPS: How Secure Communication Works on the Internet

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why HTTPS exists
> - HTTP vs HTTPS
> - What is TLS?
> - SSL vs TLS
> - Encryption Basics
> - Symmetric Encryption
> - Asymmetric Encryption
> - Public & Private Keys
> - Digital Certificates
> - Certificate Authorities (CA)
> - TLS Handshake
> - Session Keys
> - Perfect Forward Secrecy (PFS)

---

# 📖 Why Do We Need HTTPS?

Suppose you log into your bank.

Browser sends:

```
Username

↓

Password
```

If this travels across the Internet as plain text,

anyone intercepting the traffic could read it.

```
Laptop

↓

Internet

↓

Attacker 👀

↓

Bank
```

This is dangerous.

We need:

✔ Encryption

✔ Authentication

✔ Integrity

---

# HTTP Problem

HTTP provides:

✔ Communication

But it does **not** provide:

❌ Encryption

❌ Authentication

❌ Integrity Protection

Anyone who captures packets can read the contents.

---

# HTTPS Solution

HTTPS is simply:

```
HTTP

+

TLS
```

Flow:

```
Browser

↓

TLS

↓

Encrypted Channel

↓

HTTP
```

Now the HTTP messages travel securely.

---

# What is TLS?

TLS stands for:

```
Transport Layer Security
```

Its responsibilities are:

✔ Encrypt Data

✔ Verify Server Identity

✔ Detect Data Tampering

---

# SSL vs TLS

Originally:

```
SSL
```

was used.

Today:

```
TLS
```

replaced SSL.

Modern systems use TLS,

although people often still say "SSL Certificate."

---

# Three Security Goals

TLS provides:

```
Confidentiality
```

Only intended parties can read the data.

---

```
Integrity
```

Data cannot be modified without detection.

---

```
Authentication
```

The client can verify the server's identity.

---

# Encryption Basics

Without encryption:

```
Password123
```

Anyone can read it.

With encryption:

```
X8@LmP#92...
```

Only someone with the correct key can decrypt it.

---

# Two Types of Encryption

```
Symmetric
```

One shared key.

---

```
Asymmetric
```

Two keys:

```
Public

↓

Private
```

TLS uses both.

---

# Symmetric Encryption

```
Shared Secret Key
```

Both client and server know the same key.

```
Encrypt

↓

Decrypt
```

Very fast.

Used for actual data transfer.

---

# Problem with Symmetric Encryption

How do two computers securely exchange the shared key?

```
Internet

↓

Attacker Could Steal It
```

We need another method.

---

# Asymmetric Encryption

Each server has:

```
Public Key
```

and

```
Private Key
```

Public Key:

```
Shared With Everyone
```

Private Key:

```
Secret
```

Never leaves the server.

---

# Public Key

Anyone can use the Public Key to encrypt data.

```
Browser

↓

Public Key

↓

Encrypted Message
```

Only the server can decrypt it.

---

# Private Key

The server keeps:

```
Private Key
```

Only it can decrypt data encrypted with the matching Public Key.

---

# Why Not Use Asymmetric Encryption for Everything?

Asymmetric encryption is much slower than symmetric encryption.

Instead:

```
Asymmetric

↓

Exchange Secret

↓

Symmetric

↓

Transfer Data
```

This gives both security and performance.

---

# Digital Certificate

How does the browser know the Public Key really belongs to:

```
google.com
```

and not an attacker?

The server sends a:

```
Digital Certificate
```

---

# What's Inside a Certificate?

Typical contents:

- Domain Name
- Public Key
- Expiration Date
- Issuing Certificate Authority
- Digital Signature

---

# Certificate Authority (CA)

A CA is a trusted organization that verifies identities and signs certificates.

Examples include:

- Let's Encrypt
- DigiCert
- GlobalSign
- Sectigo

Browsers trust certificates signed by trusted CAs.

---

# Browser Trust Store

Browsers include a list of trusted Certificate Authorities.

```
Browser

↓

Trusted CAs
```

If the certificate is signed by one of them,

the browser continues.

Otherwise,

it shows a warning.

---

# TLS Handshake

Before sending encrypted data,

client and server establish secure communication.

---

# Step 1 — ClientHello

Browser sends:

```
Supported TLS Versions

↓

Supported Cipher Suites

↓

Random Value
```

---

# Step 2 — ServerHello

Server replies:

```
Chosen TLS Version

↓

Chosen Cipher

↓

Certificate

↓

Random Value
```

---

# Step 3 — Certificate Validation

Browser checks:

✔ Certificate Signature

✔ Domain Name

✔ Expiration

✔ Trusted CA

If validation fails,

the connection is rejected.

---

# Step 4 — Key Exchange

Client and server securely derive a shared session key.

In modern TLS versions (such as TLS 1.3),

this is commonly done using ephemeral key exchange algorithms like ECDHE.

---

# Step 5 — Session Key Created

Both sides now know:

```
Shared Session Key
```

Future communication uses symmetric encryption.

---

# TLS Handshake Flow

```
Browser

↓

ClientHello

↓

Server

↓

ServerHello

↓

Certificate

↓

Key Exchange

↓

Shared Session Key

↓

Encrypted Communication
```

---

# Session Key

The Session Key is temporary.

```
TLS Session

↓

One Key
```

When the session ends,

the key is discarded.

---

# Perfect Forward Secrecy (PFS)

Modern TLS commonly uses ephemeral key exchange.

Benefits:

Suppose the server's private key is compromised tomorrow.

Past sessions remain secure,

because each session used a unique temporary key.

This property is called:

```
Perfect Forward Secrecy
```

---

# Data Transfer

After the handshake:

```
HTTP Request

↓

Encrypt

↓

Internet

↓

Decrypt

↓

Server
```

Responses are encrypted the same way.

---

# Integrity

TLS also verifies that data wasn't modified.

If packets are altered during transit,

the receiver detects the tampering and rejects the data.

---

# Real Example

Suppose Chrome opens:

```
https://openai.com
```

Flow:

```
DNS

↓

TCP Handshake

↓

TLS Handshake

↓

Certificate Validation

↓

Shared Session Key

↓

Encrypted HTTP
```

Only after TLS completes does HTTP begin.

---

# Docker Example

Suppose Docker pulls an image.

```
Docker Client

↓

TLS

↓

Docker Registry

↓

Secure Image Download
```

The image is transferred over HTTPS.

---

# Kubernetes Example

The Kubernetes API Server communicates using HTTPS.

```
kubectl

↓

TLS

↓

API Server
```

Authentication and encryption both rely on TLS.

---

# Why TLS Matters

Everything secure on the Internet depends on TLS.

Examples:

- HTTPS Websites
- REST APIs
- Banking
- Cloud Platforms
- Kubernetes
- Docker Registry
- GitHub
- Email (SMTP over TLS, IMAPS)
- VPNs

Without TLS,

Internet communication would be vulnerable to eavesdropping and tampering.

---

# Hands-on Lab

## Lab 1 — View Certificate

```bash
openssl s_client -connect google.com:443
```

Observe:

- Certificate
- Cipher
- TLS Version

---

## Lab 2 — View Certificate Details

```bash
openssl x509 -text
```

(Use with a certificate file.)

---

## Lab 3 — Test HTTPS Headers

```bash
curl -I https://google.com
```

---

## Lab 4 — Check TLS Version

```bash
openssl s_client -connect google.com:443 -tls1_3
```

---

## Lab 5 — View Browser Certificate

Open any HTTPS website.

Click the lock icon.

Inspect the certificate information.

---

# Interview Questions

## What is TLS?

TLS (Transport Layer Security) is a protocol that provides encryption, authentication, and integrity for network communication.

---

## What is the difference between HTTP and HTTPS?

HTTPS is HTTP running over a secure TLS connection.

---

## What is the difference between SSL and TLS?

TLS is the modern successor to SSL. SSL is obsolete and should no longer be used.

---

## Why are both symmetric and asymmetric encryption used in TLS?

Asymmetric cryptography securely establishes a shared secret.

Symmetric cryptography efficiently encrypts the application data.

---

## What is a Digital Certificate?

A Digital Certificate binds a domain name to a public key and is signed by a trusted Certificate Authority.

---

## What is a Certificate Authority (CA)?

A CA verifies identities and signs digital certificates that browsers and operating systems trust.

---

## What is Perfect Forward Secrecy?

Perfect Forward Secrecy ensures that compromising a server's long-term private key does not expose previously recorded encrypted sessions.

---

## What happens before HTTPS data is exchanged?

A TLS handshake establishes trust, negotiates algorithms, and creates a shared session key.

---

# Summary

TLS secures communication between clients and servers.

```
Browser

↓

TCP Handshake

↓

TLS Handshake

↓

Certificate Validation

↓

Session Key

↓

Encrypted HTTP

↓

Server
```

Key concepts:

- HTTPS is HTTP over TLS.
- TLS provides confidentiality, integrity, and authentication.
- Symmetric encryption protects application data.
- Asymmetric cryptography enables secure key exchange.
- Digital Certificates verify server identity.
- Certificate Authorities establish trust.
- Session keys encrypt each connection.
- Perfect Forward Secrecy protects past sessions.

Understanding TLS prepares you for advanced topics such as mutual TLS (mTLS), API security, service meshes, Kubernetes security, zero-trust networking, and secure distributed systems.

---

# Next Chapter

## Chapter 39 — NAT (Network Address Translation): How One Public IP Serves Thousands of Devices

We'll explore:

- Why NAT exists
- Public vs Private IP Addresses
- Source NAT (SNAT)
- Destination NAT (DNAT)
- PAT (Port Address Translation)
- MASQUERADE
- Connection Tracking with NAT
- Home Routers
- Docker Port Mapping
- Kubernetes Service NAT
- Complete NAT Packet Flow