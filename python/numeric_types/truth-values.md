# Booleans: Truthiness

Every object in Python has an associated **truth value** — a notion of whether the object is considered "true" or "false" in a boolean context.

In the previous lesson, we saw that integers have an intrinsic truth value:

```python
bool(0)          # False
bool(1), bool(-1), bool(100)   # (True, True, True)
```

This truthiness has **nothing to do with the fact that `bool` is a subclass of `int`**. It's determined by whether the object's class implements `__bool__()` (or, failing that, `__len__()`).

---

## Where Truthiness Comes From

### `__bool__()`

Let's look at `int`:

```python
help(bool)
```

Among the inherited methods from `int`, we find:

```text
|  __bool__(self, /)
|      self != 0
```

So when we write:

```python
bool(100)   # True
```

Python is effectively calling:

```python
(100).__bool__()   # True
(0).__bool__()     # False
```

The `__bool__()` method is the **primary hook** for truthiness.

### `__len__()` as a Fallback

If a class does **not** implement `__bool__()`, Python falls back to `__len__()`:

- **Empty** → falsy (`len(obj) == 0`)
- **Non-empty** → truthy (`len(obj) > 0`)

This is why containers like `list`, `tuple`, `str`, `dict`, and `set` are falsy when empty and truthy when not — they implement `__len__`, not `__bool__`.

### Default: Always Truthy

If a class implements **neither** `__bool__()` nor `__len__()`, its instances are **always truthy**:

```python
class Thing:
    pass

bool(Thing())   # True
```

This is an important default: *"if you haven't said otherwise, you're truthy."*

---

## Truthiness by Category

### Numeric Types

Every non-zero numeric value is truthy; every zero value is falsy.

```python
from fractions import Fraction
from decimal import Decimal

bool(10), bool(1.5), bool(Fraction(3, 4)), bool(Decimal('10.5'))
# (True, True, True, True)

bool(0), bool(0.0), bool(Fraction(0, 1)), bool(Decimal('0')), bool(0j)
# (False, False, False, False, False)
```

Note: `0j` (zero complex) is falsy, but `1j` (non-zero complex) is truthy — the same rule applies.

### Sequence Types

An **empty** sequence is falsy; a **non-empty** sequence is truthy.

```python
bool([1, 2, 3]), bool((1, 2, 3)), bool('abc'), bool(1j)
# (True, True, True, True)

bool([]), bool(()), bool('')
# (False, False, False)
```

### Mapping Types

Same rule: empty mapping is falsy, non-empty mapping is truthy.

```python
bool({'a': 1}), bool({1, 2, 3})   # (True, True)
bool({}), bool(set())              # (False, False)
```

### `None`

The singleton `None` is **always falsy**:

```python
bool(None)   # False
```

---

## A Real Use Case: Truthiness as a Guard

Any conditional expression involving non-`bool` objects will use their **truth value** as the result of the test.

```python
a = [1, 2, 3]
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
# → 1

a = []
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
# → a is None, or a is empty

a = 'abc'
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
# → a

a = ''
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
# → a is None, or a is empty
```

This single `if a:` handles **both** `None` and empty values — which is why it's idiomatic.

---

## Why the "Explicit" Alternatives Are Broken

We *could* try to express the same check with a longer, more "explicit" expression:

```python
a = 'abc'
if a is not None and len(a) > 0:
    print(a[0])
else:
    print('a is None, or a is empty')
# → a
```

This works — but only because it's *both* correct *and* carefully ordered. Removing either half breaks it in a different way:

### Fragile variant 1 — `is not None` only

```python
a = 'abc'
if a is not None:
    print(a[0])
# → a   (works, because 'abc'[0] exists)

a = ''
if a is not None:
    print(a[0])
# IndexError: string index out of range
```

`''` is not `None`, so the guard lets it through — and indexing explodes.

### Fragile variant 2 — `len(a) > 0` only

```python
a = None
if len(a) > 0:
    print(a[0])
# TypeError: object of type 'NoneType' has no len()
```

`len(None)` itself raises — we never even reach the comparison.

### The fully correct long form

You'd need **both** checks, in the **right order**:

```python
a = None
if a is not None and len(a) > 0:
    print(a[0])
```

---

## Order Matters — Short-Circuiting in Guard Clauses

The order of the two checks is not stylistic; it's **load-bearing**. Consider the reversed order:

```python
a = None
if len(a) > 0 and a is not None:
    print(a[0])
# TypeError: object of type 'NoneType' has no len()
```

Why? Because `and` evaluates **left to right**, and the left side `len(a) > 0` runs *first*. Since `a` is `None`, `len(a)` raises before `a is not None` is ever considered.

The correct order puts the **cheap, safe, exception-free check** first:

```text
a is not None and len(a) > 0
└──── safe ────┘   └─ safe only if the first passed ─┘
```

If `a is None`, the second operand is short-circuited — never evaluated — and no exception is raised.

---

## The Idiomatic Form: `if a:`

Compare the two implementations:

**Long, fragile form:**

```python
if a is not None and len(a) > 0:
    print(a[0])
```

**Idiomatic form:**

```python
if a:
    print(a[0])
```

The idiomatic form:

- Is **shorter** and reads more naturally.
- Correctly handles `None`, empty strings, empty lists, `0`, and any other falsy value in one test.
- Doesn't require you to remember the correct ordering of sub-conditions.
- Delegates the "what counts as empty?" question to the object's own `__bool__` / `__len__` — which is exactly where it belongs.

---

## Truthiness Summary

| Category | Falsy when | Truthy when |
|---|---|---|
| Numbers (`int`, `float`, `complex`, `Fraction`, `Decimal`, …) | `== 0` | `!= 0` |
| Sequences (`str`, `list`, `tuple`, `bytes`, …) | empty (`len() == 0`) | non-empty |
| Mappings (`dict`, `set`, `frozenset`, …) | empty | non-empty |
| `None` | always | never |
| Objects with `__bool__` | `__bool__()` returns `False` | `__bool__()` returns `True` |
| Objects with only `__len__` | `len() == 0` | `len() > 0` |
| Objects with neither | never | always |

---

## Mental Model

```text
Truthiness is a protocol, not a coercion.

bool(x) does, in order:
    1. If x.__bool__ exists         → use its return value
    2. Else if x.__len__  exists    → use (len(x) != 0)
    3. Else                          → True

Guarding:
    "if a:"               → handles None, empty, zero — the idiomatic form
    "a is not None and len(a) > 0" → equivalent, but fragile and verbose

Ordering in "and":
    Put safe, cheap checks first (is not None)
    Put checks that require the first to pass second (len(a), a[0])
```

Once you internalize truthiness as a **protocol** (`__bool__`, then `__len__`, then default `True`), the idiomatic `if a:` stops feeling like a shortcut and starts reading as the *most correct* expression of intent — one that automatically adapts to whatever "empty" means for the type at hand.