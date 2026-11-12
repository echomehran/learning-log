# Unpacking Iterables

## Side Note: What Actually Defines a Tuple

Before we can talk about unpacking, we need to be precise about tuples, because their syntax is subtler than it looks.

A tuple can be written with parentheses:

```python
a = (1, 2, 3)
type(a)   # tuple
```

But it can also be written **without** them:

```python
a = 1, 2, 3
type(a)   # tuple
```

Both produce the same object. The parentheses are optional — they're just grouping.

### The comma is what makes the tuple

> **What defines a tuple is the comma (`,`), not the parentheses.**

This becomes obvious when you try to create a **single-element tuple**. The intuitive attempt fails:

```python
a = (1)
type(a)   # int   ← NOT a tuple!
```

`(1)` is just the integer `1` wrapped in parentheses — grouping, not a tuple. To make a single-element tuple, you need a trailing comma:

```python
a = (1,)
type(a)   # tuple

a = 1,
type(a)   # tuple   ← parentheses optional even here
```

### The two exceptions

There are exactly two cases where parentheses are required:

1. **Empty tuple**: `a = ()` — there's no value, so there's nothing to put a comma next to.
2. **Disambiguation in expressions**: `f((1, 2))` needs the inner parentheses to distinguish "a tuple as one argument" from "two arguments."

You can also construct tuples via the constructor:

```python
a = tuple()          # empty tuple
a = tuple([1, 2, 3]) # from an iterable
```

### Summary of tuple syntax

| Expression | Type | Notes |
|---|---|---|
| `(1, 2, 3)` | `tuple` | Parentheses optional |
| `1, 2, 3` | `tuple` | Same object |
| `(1)` | `int` | Just grouping — **not** a tuple |
| `(1,)` | `tuple` | Trailing comma is the key |
| `1,` | `tuple` | Same, parentheses omitted |
| `()` | `tuple` | Empty tuple — parentheses required |
| `tuple()` | `tuple` | Constructor form |

This matters for unpacking because unpacking operates on **iterables**, and tuples are the most common iterable you'll be unpacking. Knowing that `1, 2` and `(1, 2)` are the same object removes a surprising amount of confusion.

---

## Unpacking: The Core Idea

**Unpacking** splits an iterable into individual variables in a single assignment:

```python
l = [1, 2, 3, 4]
a, b, c, d = l
print(a, b, c, d)   # 1 2 3 4
```

The left-hand side is a **pattern** (a comma-separated list of names), and the right-hand side is an **iterable**. Python iterates the right-hand side and assigns each element to the corresponding name.

### What can be unpacked?

Any iterable:

```python
a, b, c = 'XYZ'         # strings are iterables of characters
print(a, b, c)          # X Y Z

a, b, c = [10, 20, 30]  # lists
a, b, c = (10, 20, 30)  # tuples
a, b, c = range(3)      # ranges
```

The right-hand side doesn't have to be a literal — any expression that produces an iterable works:

```python
def coordinates():
    return (3, 5)

x, y = coordinates()    # unpack the function's return value
```

### The length must match

Unpacking requires the number of elements to match the number of names exactly:

```python
a, b = [1, 2, 3]
# ValueError: too many values to unpack (expected 2)

a, b, c = [1, 2]
# ValueError: not enough values to unpack (expected 3, got 2)
```

This is the fundamental constraint — no partial unpacking without the extended syntax (`*rest`, covered separately). If you don't know the length, don't use plain unpacking.

---

## Swapping Variables

The most famous application of unpacking — and the one that most clearly shows its power — is variable swapping.

### The traditional approach (from other languages)

```python
a = 10
b = 20

tmp = a
a = b
b = tmp

print(a, b)   # 20 10
```

You need a temporary variable because assigning `a = b` destroys `a` before you can assign `b = a`. This is how it works in Java, C, C++, and most other languages.

### The Pythonic approach

```python
a = 10
b = 20

a, b = b, a

print(a, b)   # 20 10
```

No temporary variable. One line. This works because of how Python evaluates assignments:

1. **Evaluate the right-hand side first.** `b, a` creates a tuple `(20, 10)` — both values are captured *before* any assignment happens.
2. **Unpack into the left-hand side.** The tuple `(20, 10)` is unpacked into `a, b`, so `a = 20` and `b = 10`.

The key insight: **the entire right-hand side is evaluated before any left-hand binding occurs**. So there's no risk of the intermediate state clobbering a value.

### Visualizing the swap

```text
    Before:
        a ────► 10
        b ────► 20
    
    Evaluate RHS:  b, a   creates tuple (20, 10)
        ┌───────────────┐
        │ tuple(20, 10) │
        └───────────────┘
    
    Unpack LHS:
        a ────► 20   (first element of the tuple)
        b ────► 10   (second element)
    
    After:
        a ────► 20
        b ────► 10
```

### Initialization also unpacks

The same mechanism lets you initialize multiple variables in one statement:

```python
a, b = 10, 20
```

Here `10, 20` is a tuple, unpacked into `a, b`. This is not special syntax — it's the same "tuple then unpack" mechanism as the swap.

### Chaining: the swap and the initialization combine

```python
a, b = 10, 20
a, b = b, a
print(a, b)   # 20 10
```

Two lines. No temp variable. `a, b = 10, 20` is an unpack of a tuple; `a, b = b, a` is an unpack of a (different) tuple. Both lines use the exact same mechanism — one binds names to initial values, the other rebinds them to each other's values.

---

## Unpacking Unordered Objects

Dictionaries and sets are also iterable — but with important caveats.

### Dicts: iterating keys

A dict iterates over its **keys**:

```python
dict1 = {'p': 1, 'y': 2, 't': 3, 'h': 4, 'o': 5, 'n': 6}

for c in dict1:
    print(c)
# p
# y
# t
# h
# o
# n
```

Unpacking follows the same rule — it unpacks the **keys**, not the values:

```python
a, b, c, d, e, f = dict1
print(a)   # p
print(b)   # y
# ... and so on
```

If you want values or key-value pairs:

```python
a, b, c, d, e, f = dict1.values()   # unpacks values
a, b, c, d, e, f = dict1.keys()     # unpacks keys (same as default)
(a1, 1), (b1, 2), ... = dict1.items()   # unpacks (key, value) tuples — one level of nesting
```

### Sets: iterate in arbitrary order

```python
s = {'p', 'y', 't', 'h', 'o', 'n'}

for c in s:
    print(c)
# p
# t
# y
# n
# o
# h
```

Unpacking works — but the order is arbitrary:

```python
a, b, c, d, e, f = s
```

This **succeeds** (the set has exactly 6 elements, and 6 names can be bound), but the mapping of which name gets which element is **not under your control** and is **not guaranteed to be stable across runs or Python versions**.

### The order guarantee (and lack thereof)

Here's the crucial detail — and it's changed over Python versions:

| Version | Dict iteration order | Set iteration order |
|---|---|---|
| Python < 3.7 | Arbitrary (hash-based) | Arbitrary |
| Python 3.7+ | **Insertion order** (guaranteed) | Arbitrary (still hash-based) |

As of Python 3.7, **dictionaries preserve insertion order** — that's a language guarantee, not an implementation detail. So `{'p': 1, 'y': 2, ...}` will iterate as `'p', 'y', ...` consistently, in every conforming Python 3.7+ implementation.

But **sets are still unordered** in the sense that their iteration order isn't specified — it's determined by the hash table layout, and while it happens to be stable for a given set within a running process, it can differ between runs (due to hash randomization) or between Python versions.

### What this means in practice

- **Dicts**: Unpacking is deterministic on Python 3.7+, but for anything that **requires** a specific order that isn't insertion order, use `OrderedDict` or convert explicitly (e.g., `sorted(dict1)`).
- **Sets**: Never rely on the unpacking order — it can change without notice. If you need deterministic unpacking, convert first: `a, b, c = sorted(s)`.

### The recurring lesson

Unpacking operates on **iteration order**, whatever that order happens to be for the given container. For ordered iterables (lists, tuples, strings, ranges, dicts in 3.7+), the order is well-defined. For unordered containers (sets), it isn't — and expecting stability is a bug waiting to surface.

---

## Summary

```text
Tuple syntax:
    (1, 2)       tuple
    1, 2         tuple    (parentheses optional)
    (1)          int      (NOT a tuple — just grouping)
    (1,)         tuple    (trailing comma matters)
    1,           tuple    (same)
    ()           tuple    (empty — parentheses required)
    tuple()      tuple    (constructor)

Unpacking:
    a, b, c = iterable         requires len(iterable) == 3
    a, b    = [1, 2]           ✅
    a, b, c = [1, 2]           ❌ ValueError

Swap (and initialization):
    a, b = 10, 20              unpack tuple into a, b
    a, b = b, a                RHS evaluated first,
                               then unpacked — no temp needed

Dicts (Python 3.7+):
    Iterate keys (default).
    Unpack keys, not values.
    Order: insertion order (guaranteed).

Sets:
    Unpack in arbitrary order.
    Order: hash-based, not guaranteed.
    Never rely on it — convert/sort first if you must.

Core mental model:
    RHS is fully evaluated before any assignment.
    LHS is a pattern; each name gets one element.
    Lengths must match.
```

Unpacking is one of Python's most quietly powerful features — it unifies multiple assignment, variable swapping, and tuple construction under a single syntactic form. The underlying rule is simple: **the right-hand side is evaluated completely, then its elements are distributed to the names on the left, in iteration order**. Once you internalize that, tuple syntax stops being a syntactic curiosity (commas vs. parentheses), swaps stop needing a temp variable, and the subtle difference between ordered and unordered containers stops being a source of silent bugs.