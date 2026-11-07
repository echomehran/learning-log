# Variables Are References to Memory

## Finding the Address

We can find the memory address a variable **references** using the `id()` function.

`id()` returns the memory address of its argument as a **base-10 integer**.

We can use `hex()` to convert that base-10 integer to **base-16 (hexadecimal)** — a more readable form for addresses.

```python
my_var = 10
print('my_var = {0}'.format(my_var))
print('memory address of my_var (decimal): {0}'.format(id(my_var)))
print('memory address of my_var (hex): {0}'.format(hex(id(my_var))))
```

```text
my_var = 10
memory address of my_var (decimal): 1968827120
memory address of my_var (hex): 0x7559eaf0
```

```python
greeting = 'Hello'
print('greeting = {0}'.format(greeting))
print('memory address of greeting (decimal): {0}'.format(id(greeting)))
print('memory address of greeting (hex): {0}'.format(hex(id(greeting))))
```

```text
greeting = Hello
memory address of greeting (decimal): 1688681719264
memory address of greeting (hex): 0x1892d4625e0
```

Note that `my_var` and `greeting` have **different** memory addresses.

---

## The Critical Distinction

Here's the most important thing to understand from this example:

> **`my_var` is not *equal to* 10. `my_var` is a *reference to* an object (an integer) containing the value 10, located at the memory address `id(my_var)`.**

The same applies to `greeting`.

This isn't a pedantic phrasing — it's the correct mental model. The variable `my_var` doesn't *hold* the number 10. It's a **name** pointing to an **object** that happens to have the value 10. The distinction becomes visible the moment you inspect the address.

---

## What the Model Actually Looks Like

### In the simplified (common but imprecise) view

```text
    my_var = 10        →    my_var holds 10
    greeting = 'Hello' →    greeting holds 'Hello'
```

This reads naturally but obscures what's really happening.

### In the accurate view

```text
    my_var   ────►   int object    (value: 10,      address: 0x7559eaf0)
    greeting ────►   str object    (value: 'Hello', address: 0x1892d4625e0)
```

Each variable is a **reference** — a pointer to an object. The object is what has:
- A **value** (what it represents — `10`, `'Hello'`)
- A **type** (its class — `int`, `str`)
- An **identity** (its address in memory — `id()`)

The variable itself has none of these. It's just a name that resolves to the object's address.

### What `id()` and `hex()` are telling you

```text
    id(obj)      →  the object's address, as a base-10 integer
    hex(id(obj)) →  the same address, as a base-16 string ('0x7559eaf0')
```

The address is the object's **identity**. Two references to the same address are references to the same object. Two references to different addresses are references to different objects.

---

## Why This Matters: Three Concrete Consequences

### 1. `my_var` and `10` are not the same thing

```python
my_var = 10
```

The `10` in this line is a **literal** — a syntactic expression that **creates or reuses an object with the value 10**. The `my_var` in this line is a **name** that gets **bound** to that object.

Two different things:
- **`10`** — a *literal* in the source code (an *expression*)
- **`my_var`** — a *name* in the source code (a *binding*)

`my_var` **references** an object whose value is 10. The value 10 is on the object; the reference is in the name. This is why:

```python
my_var = 10
my_var = 20
```

The first line doesn't "overwrite 10" — it binds `my_var` to an object with value 10. The second line doesn't "change 10 into 20" — it **rebinds** `my_var` to a *different* object with value 20. The integer `10` object still exists (it's interned); it's just no longer referenced by `my_var`.

### 2. Two names can reference the same object

```python
a = 10
b = 10
print(id(a))   # 1968827120
print(id(b))   # 1968827120   ← same address
```

`a` and `b` reference the **same object**. There aren't two separate `10`s in memory — there's one, and both names point to it. This is **shared referencing**, and it's safe here only because the integer object is immutable.

### 3. Names can move; objects stay put

```python
x = 10
print(id(x))   # 1968827120

x = 15
print(id(x))   # different address
```

The name `x` moved. The object it was pointing to (the interned `10`) didn't go anywhere — `x` just no longer references it.

---

## The Full Picture

```text
    Source code            Names                 Objects
    ───────────            ─────                 ───────

    my_var = 10            my_var ─────► int(value=10, addr=A)

    greeting = 'Hello'     greeting ───► str(value='Hello', addr=B)

    print(id(my_var))      asks: what object does `my_var` reference?
                           answer: the int at addr A
                           prints: 1968827120

    print(hex(id(my_var))) same object, address in hex:
                           prints: '0x7559eaf0'
```

### In one sentence

> **A variable is a name bound to an object. The object has a value, a type, and an address. `id(x)` tells you the address of the object that `x` currently references.**

---

## Why the "Variable = Value" Mental Model Fails

Many tutorials say "a variable is like a box that holds a value." That metaphor breaks down in Python because:

| Box metaphor says | Reality |
|---|---|
| `a = 10` puts 10 into box `a` | `a` is bound to an existing object (the interned `10`) |
| `b = a` copies the value from `a` into `b` | `b` is bound to the **same object** `a` references |
| `a = 20` overwrites the box | `a` is **rebound** to a different object; the original still exists |
| Two boxes with the same value are independent | Two names can reference the same object (shared) |

The box metaphor suggests **storage** — value goes in, value comes out. Python is about **references** — names point at objects. Once you replace the box metaphor with the reference model, the surprises (shared lists, immutable sharing, rebinding) become logical consequences.

### The correct metaphor: labels, not boxes

```text
    Names are LABELS stuck onto objects.

    a = 10           →  label "a" stuck on the interned int object 10
    b = a            →  label "b" stuck on the same object
    a = 20           →  label "a" removed from 10, stuck on 20
                        (label "b" still on 10)
```

The objects are what's real; names are just labels that make objects reachable. `id()` tells you which object a label is stuck on.

---

## Summary

```text
The model:

    Variables are references (bindings) to objects.
    Objects have: value, type, identity (address).
    Names have: nothing — they just point to objects.

    id(obj)      →  the object's address, as a base-10 integer
    hex(id(obj)) →  the same address, in hex

Consequences:

    my_var = 10            name ──► int(value=10) at addr A
    my_var == 10           compares values, not addresses
    my_var is 10           compares addresses (rarely the right check)

    a = 10; b = 10         both names ──► same object (interned 10)
                           → same id, shared reference

    a = 10; a = 15         a moves from int(10) to int(15)
                           → different id, old object unchanged

Metaphor shift:

    ❌ "variable = box that holds a value"
    ✅ "variable = label stuck onto an object"
```

The point isn't merely that `id()` and `hex()` are useful — it's that they **demonstrate the model directly**. Every behavior Python has demonstrated in this course — interning, shared references, mutability, aliasing — is a logical consequence of one fact: **variables are references to objects, not containers for values**. Once you internalize this, `id()` is no longer a curiosity — it's the tool that makes the invisible relationship between names and objects visible and testable.