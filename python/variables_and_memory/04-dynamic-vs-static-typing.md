# Dynamic Typing

Python is **dynamically typed**.

This means that the type of a variable is simply the type of the object that the variable name references. The variable itself has **no type associated with it** — only the object it points to does.

---

## What "Dynamic" Actually Means

In a statically typed language, a variable name is *declared* with a type, and that type is enforced for the lifetime of the variable:

```text
# Hypothetical static language
int a = 10        # a IS an int, forever
a = "hello"       # compile error: a is not a str
```

In Python, the name is just a **label** that points to an object. The name doesn't *carry* a type — the object does. Reassignment simply re-points the label to a different object.

---

## Examples

### A variable holding a string

```python
a = "hello"
type(a)
# → str
```

### The same name, now holding an int

```python
a = 10
type(a)
# → int
```

### The same name, now holding a function

```python
a = lambda x: x**2
a(2)
# → 4

type(a)
# → function
```

Each reassignment didn't "change `a`'s type" — there was never a type on `a` to change. Instead, `a` was re-pointed to a *different object*, and `type(a)` reports the type of whatever object `a` currently references.

---

## The Name/Object Distinction

This is the core mental model:

```text
    a ──────────────►  "hello"      (str object)
    
    a ──────────────►  10           (int object)
    
    a ──────────────►  <function>   (function object)
```

The name `a` is a **binding** — a reference from a name to an object. Rebinding `a` doesn't mutate the previous object; it changes *which object* the name points to. The old object (`"hello"`, `10`, etc.) still exists independently (and may be garbage-collected if nothing else references it).

This is why you can think of it like:

```text
    type(a)   ≡   type(<the object a currently points to>)
```

Never:

```text
    type(a)   ≡   <a fixed type that a was declared with>
```

---

## Implications

### 1. Names are labels, not containers

```python
x = [1, 2, 3]
y = x            # y points to the SAME list object
y.append(4)
print(x)         # [1, 2, 3, 4] — x sees the change
```

`x` and `y` are two names pointing to one object. There's no "copy" implicit in assignment — only a new binding.

### 2. Reassignment ≠ mutation

```python
x = [1, 2, 3]
x = [1, 2, 3, 4]   # rebinding x to a NEW list object
```

The first list is unchanged; `x` just no longer points to it.

### 3. Functions don't have a "variable type"

Because the name doesn't have a type, there's nothing to enforce:

```python
def process(a):
    return a + 1

process(10)        # works with int
process(2.5)       # works with float
process(True)      # works with bool (True + 1 == 2)
```

Whether this is a feature or a bug depends on design — but it's a direct consequence of dynamic typing.

### 4. Types live on objects, not names

```python
a = 10
type(a)      # → int       — the type of the int object
del a
type(a)      # NameError   — "a" doesn't exist at all
```

The type doesn't "live" in `a`; it lives in the object `a` referenced. Delete the name, and there's nothing left to have a type.

---

## Related Concepts

Dynamic typing is not the same as *weak* typing or *untyped*:

| Concept | Meaning |
|---|---|
| **Dynamic typing** | Types checked at **runtime**, attached to **objects**, not names |
| **Static typing** | Types checked at **compile time**, attached to **names/declarations** |
| **Strong typing** | Operations between incompatible types raise errors (e.g., `"a" + 1` fails) |
| **Weak typing** | Implicit coercion between types (e.g., `"a" + 1` becomes `"a1"`) |

Python is **dynamically typed** *and* **strongly typed**. You don't get a type of `a` fixed at declaration, but you *do* get errors when you mix incompatible types:

```python
"hello" + 10
# TypeError: can only concatenate str (not "int") to str
```

So "dynamic" refers to **when** and **where** types are tracked — not to whether they're enforced at all.

---

## Summary

```text
Dynamic typing:
    - Variables have NO type.
    - Objects HAVE a type.
    - A variable is a name bound to an object.
    - Reassignment rebinds the name to a different object.
    - type(x) returns the type of the object x currently references.
    - Type checking happens at runtime.

Mental model:
    name  →  object  →  type(object)
    (no type)          (has type)

Not to be confused with:
    - Weak typing   (implicit coercion)  — Python is NOT weakly typed
    - Untyped       (no types at all)    — Python is NOT untyped
```

Once you internalize that **names are labels, not boxes**, and that **types belong to objects**, the behavior of reassignment, aliasing, and `type()` all become consequences of one simple rule rather than a collection of quirks.