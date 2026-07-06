# 👑 C Mastery Roadmap

# Module 8 — Bit Manipulation

# Chapter 64 — Toggling Bits

---

# Most beginners think:

> "To change a bit, I must first check whether it is 0 or 1."

For example:

```c
if(bit == 0)
{
    bit = 1;
}
else
{
    bit = 0;
}
```

This works...

but it's unnecessary.

There is a much smarter way.

The **Bitwise XOR (`^`)** operator can flip a bit with **one operation**.

This technique is used everywhere:

- Linux Kernel
- Embedded Systems
- Device Drivers
- Network Protocols
- Microcontrollers
- LED Controllers
- Operating Systems

---

# Learning Objectives

After this chapter you will understand:

- What toggling means
- Why XOR (`^`) is used
- How to toggle one bit
- How to toggle multiple bits
- Real-world examples
- Common mistakes

---

# What Does "Toggling" Mean?

Toggling means:

```
0

↓

1
```

or

```
1

↓

0
```

Every time you toggle,

the bit flips to the opposite value.

---

# Real-Life Example

Think about a room light.

Current state:

```
OFF
```

Press the switch.

```
ON
```

Press again.

```
OFF
```

Press again.

```
ON
```

The switch **toggles** the light.

Bit toggling works exactly the same way.

---

# Why XOR?

Look at the XOR truth table.

```
0 ^ 0 = 0

1 ^ 0 = 1

0 ^ 1 = 1

1 ^ 1 = 0
```

Notice something.

Whenever the mask bit is:

```
1
```

the original bit flips.

```
0

↓

1
```

```
1

↓

0
```

Exactly what we need.

---

# Example

Current:

```
00000000
```

Need:

```
Toggle Bit 2
```

Mask:

```
00000100
```

Operation:

```
00000000

^

00000100

------------

00000100
```

Toggle again:

```
00000100

^

00000100

------------

00000000
```

The bit flips every time.

---

# Creating the Mask

Professional programmers write:

```c
1 << 2
```

instead of:

```
00000100
```

Much cleaner.

---

# Basic Example

```c
#include <stdio.h>

int main()
{
    unsigned char status = 0;

    status ^= (1 << 2);

    printf("%u\n", status);

    status ^= (1 << 2);

    printf("%u\n", status);

    return 0;
}
```

Output:

```
4

0
```

---

# Shorter Syntax

Instead of:

```c
status = status ^ (1 << 2);
```

Use:

```c
status ^= (1 << 2);
```

Professional C code almost always uses:

```c
^=
```

---

# Toggling Multiple Bits

Need to toggle:

```
Bit 1

Bit 4

Bit 6
```

Create one mask.

```c
status ^= (1 << 1)
       |  (1 << 4)
       |  (1 << 6);
```

All three bits flip simultaneously.

---

# Visual Representation

Current:

```
10100110
```

Toggle:

```
Bit 4
```

Mask:

```
00010000
```

Operation:

```
10100110

^

00010000

------------

10110110
```

Only Bit 4 changed.

---

# Toggle Again

Current:

```
10110110
```

Same mask:

```
00010000
```

Operation:

```
10110110

^

00010000

------------

10100110
```

Original value restored.

---

# Why XOR is Special

Remember:

```
A ^ A

=

0
```

and

```
A ^ B ^ B

=

A
```

Example:

```
10110110

↓

XOR Mask

↓

XOR Same Mask

↓

Original Value
```

This property makes XOR useful in:

- Encryption
- Checksums
- Error detection
- Cryptography

---

# Real-World Example — LED Controller

Suppose:

```
Bit 0

↓

LED 1

Bit 1

↓

LED 2

Bit 2

↓

LED 3
```

Press a button.

```
LED

↓

Toggle
```

Every press flips the LED state.

---

# Real-World Example — Linux Debug Flags

Suppose debugging mode is stored in one bit.

Pressing:

```
Debug Toggle
```

uses XOR.

```
OFF

↓

ON

↓

OFF

↓

ON
```

---

# Real-World Example — Device Register

Suppose:

```
Power

↓

Bit 0

WiFi

↓

Bit 1

Bluetooth

↓

Bit 2
```

Need to invert Bluetooth.

One XOR instruction.

No condition needed.

---

# Common Mistakes

---

## Using OR Instead of XOR

Wrong:

```c
status |= (1 << 3);
```

OR always sets the bit to `1`.

It does **not** toggle.

---

## Using AND

Wrong:

```c
status &= ~(1 << 3);
```

AND always clears the bit.

It does **not** toggle.

---

## Forgetting XOR Flips Every Time

Calling:

```c
status ^= (1 << 3);
```

twice returns the bit to its original value.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    unsigned char status = 0;

    status ^= (1 << 0);

    status ^= (1 << 2);

    status ^= (1 << 5);

    printf("%u\n", status);

    return 0;
}
```

Binary:

```
00100101
```

Run it again with the same toggles,

and those bits return to their original state.

---

# Hands-on Labs

## Lab 1

Toggle:

```
Bit 0
```

twice.

Observe the result.

---

## Lab 2

Toggle:

```
Bit 2

Bit 5

Bit 7
```

---

## Lab 3

Build an LED simulator.

Each key press toggles one LED.

---

## Lab 4

Represent Linux permissions.

Toggle:

```
Execute
```

permission.

---

## Lab 5

Create a hardware register.

Toggle:

```
Bluetooth

GPS
```

---

# Interview Questions

### What does toggling mean?

Changing:

```
0

↓

1
```

or

```
1

↓

0
```

---

### Which operator is used?

Bitwise XOR:

```c
^
```

---

### Why does XOR work?

Because:

```
Bit ^ 1

↓

Opposite Bit
```

---

### What happens if XOR is applied twice with the same mask?

The original value is restored.

---

### Where is bit toggling used?

- LED controllers
- Linux kernel
- Device drivers
- Embedded systems
- Cryptography
- Network protocols

---

# Summary

```
Current Value

↓

Bit Mask

↓

XOR

↓

Selected Bit

↓

Flipped
```

Example:

```
00100110

^

00010000

↓

00110110
```

Toggle again:

```
00110110

^

00010000

↓

00100110
```

## Key Takeaways

- Toggling flips a bit from `0` to `1` or from `1` to `0`.
- XOR (`^`) is the standard operator for toggling bits.
- Masks are usually created with:

```c
1 << bitPosition
```

- Applying the same XOR mask twice restores the original value.
- Bit toggling is widely used in operating systems, embedded systems, networking, hardware control, and cryptography.

---

# Next Chapter

## Chapter 65 — Checking Bits

You'll learn:

- How to test whether a bit is ON or OFF
- Why AND (`&`) is used for testing
- Writing helper functions like `isBitSet()`
- Real-world examples from Linux permissions, TCP flags, CPU status registers, and device drivers