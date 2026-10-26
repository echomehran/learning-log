# Python Optimizations: Interning

## Shared References Are Automatic

Earlier we saw that Python sometimes creates **shared references** on its own:

```python
a = 10
b = 10
print(id(a))   # 1968827120
print(id(b))   # 1968827120
```

`a` and `b` reference the **same object** — identical `id()`.

But consider this:

```python
a = 500
b = 500
print(id(a))   # 1935322088624
print(id(b))   # 1935322089008
```

Different `id()`s! `a` and `b` reference **different objects**, even though they hold the same value.

Why the difference? Because Python **pre-creates and caches** integer objects in the range **[-5, 256]**.

---

## The Interned Range

Within [-5, 256], integers are **singletons** — one object per value, reused everywhere:

```python
a = 256
b = 256
print(id(a))   # 1968834992
print(id(b))   # 1968834992   ← same

a = -5
b = -5
print(id(a))   # 1968826640
print(id(b))   # 1968826640   ← same
```

But cross either boundary:

```python
a = 257
b = 257
print(a is b)   # False  (two separate objects)

a = -6
b = -6
print(a is b)   # False  (two separate objects)
```

This caching is called **interning**: Python **interns** integers in [-5, 256], making them effectively singletons.

### Why that particular range?

The range [-5, 256] is a **CPython implementation detail** — not part of the language spec. It reflects empirical observation: these values are overwhelmingly common in real code (small counters, indices, booleans-as-ints, byte values, ASCII codes). Caching them saves allocations and memory without wasting much.

> ⚠️ **Don't rely on this in your own code.** `a is b` for large integers may or may not be `True` depending on the implementation, version, or even how the values were produced (literal vs. computed). Use `==` for value comparison, `is` only for identity of **singletons** like `None`, `True`, `False`.

---

## Interning Is About Identity, Not Equality

The interned integers behave as **singletons**. So any way of producing the *value* 10 yields the *same object*:

```python
a = 10
b = int(10)
c = int('10')
d = int('1010', 2)   # binary literal → 10

print(a, b, c, d)
# 10 10 10 10
```

Now compare identities:

```python
a is b   # True
a is c   # True
a is d   # True
```

All four were produced by **different mechanisms** — a literal, a constructor call, a string parse, and a base-2 parse — yet they all reference the **same interned object** in memory.

### What's happening

```text
    a = 10                  ──►  ┌──────────┐
    b = int(10)             ──►  │ int(10)  │  (interned singleton)
    c = int('10')           ──►  │  addr X  │
    d = int('1010', 2)      ──►  └──────────┘
    
    a is b is c is d  →  True   (all point to addr X)
```

The integer `10` is not "created four times" — the constructor and parsers detect the interned value and return the existing object.

---

## Where Interning Applies Beyond Integers

Integers in [-5, 256] are the canonical example, but similar caching appears in other places:

| Type | What's interned | Notes |
|---|---|---|
| **`int`** | -5 to 256 | CPython-specific |
| **`bool`** | `True`, `False` | Only two instances ever exist — `bool` cannot be subclassed |
| **`None`** | The singleton | `type(None)` is `NoneType`; one instance per process |
| **Short `str`** (literal-like) | Sometimes | Implementation-dependent; identifiers and simple literals may be interned |
| **Empty tuple** `()` | Yes | `() is ()` is `True` |
| **Small bytes** | Implementation-dependent | |

The point isn't to memorize every case — it's to understand the **model**: *some values are singletons, some are not, and `is` tells you which.*

---

## `is` vs. `==` — The Essential Distinction

This is where interning pays off in understanding:

```python
a = 500
b = 500

a == b   # True   (equal values)
a is b   # False  (different objects)

a = 10
b = 10

a == b   # True
a is b   # True   (same interned object — but that's incidental)
```

**Rule of thumb:**

- Use `==` to compare **values** — always.
- Use `is` to compare **identity**, and only for true singletons: `None`, `True`, `False`, `Ellipsis`, `NotImplemented`, sentinel objects you create.

Never write `if x is 256:` — you're depending on an implementation detail that may change tomorrow, in another interpreter (PyPy, Jython, MicroPython), or under different compilation paths.

---

## Why Interning Exists

Interning isn't just a curiosity — it's a real optimization with two benefits:

### 1. Memory

Consider how often small integers appear: loop counters, array indices, ASCII codes, status codes, boolean-as-int uses. Interning [-5, 256] means **262 objects total** for those values, instead of millions of tiny allocations across the program's lifetime.

### 2. Speed

- **Allocation**: no `int.__new__` call needed for cached values — return a pointer to the singleton.
- **Comparison**: `a is b` is a single pointer comparison, not a value comparison.
- **Hashing / dict lookup**: interned values are stable, simplifying some caching paths.

The tradeoff is trivial: 262 integer objects per process is negligible memory in exchange for eliminating a huge volume of small allocations.

---

## Implementation Detail vs. Language Guarantee

It's crucial to separate what **the language promises** from what **CPython happens to do**:

| Guaranteed by the language | CPython implementation detail |
|---|---|
| `None` is a singleton (`x is None` works) | Integers in [-5, 256] are interned |
| `True` and `False` are the only `bool` instances | Small strings may be interned |
| `type(x) is ...` for many built-in types | Exact interning thresholds vary by version |
| Immutable objects *may* be reused freely | Which ones, and when, is up to the interpreter |

Your code should **never** depend on the right column. If you find yourself writing `x is 300`, treat it as a bug — even if it happens to work in your current interpreter.

---

## Summary

```text
Interning:
    Python pre-creates and caches integer objects in [-5, 256].
    These values are singletons — one object per value, reused everywhere.

Consequences:
    a = 10; b = int('10')
    a is b   → True    (same interned object)
    
    a = 500; b = 500
    a is b   → False   (not interned — separate objects)

Beyond integers:
    None, True, False         → always singletons
    Empty tuple ()            → singleton
    Short strings (sometimes) → implementation-dependent

Use the right operator:
    ==   for value equality        (always safe)
    is   for identity / singletons (None, True, False, your sentinels)

Don't rely on interning:
    [-5, 256] is CPython-specific, not guaranteed by the language.
    Code that compares large integers with `is` is fragile.
```

Once you internalize that **some objects are singletons and some aren't**, and that **`is` tests identity while `==` tests value**, the behavior of `10` vs. `500` stops being surprising. It becomes a concrete consequence of a deliberate, narrow optimization — one you should understand but never depend on.