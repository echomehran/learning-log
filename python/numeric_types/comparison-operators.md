# Comparison Operators

Python's comparison operators fall into four families, each answering a different question about operands:

| Family | Operators | Question |
|---|---|---|
| **Identity** | `is`, `is not` | Are these the *same object*? |
| **Membership** | `in`, `not in` | Is this value *contained in* that iterable/container? |
| **Equality** | `==`, `!=` | Do these have the *same value*? |
| **Ordering** | `<`, `<=`, `>`, `>=` | How do these values *rank* relative to each other? |

Each family has its own rules — about which types it accepts, what it compares, and what it can raise. Understanding the distinctions is the key to reading comparisons correctly.

---

## Identity and Membership Operators

### Identity: `is` and `is not`

`is` and `is not` work with **any** data type because they compare **memory addresses** — the identity of the objects, not their contents.

```python
0.1 is (3+4j)        # False   (different objects)
'a' is [1, 2, 3]     # False   (different objects)
```

These are *always* well-defined — no type compatibility required. Two objects either occupy the same memory or they don't. Identity is a universal, type-independent property.

> **Recall from prior lessons:** `is` is only reliable as an equality check for **guaranteed singletons** (`None`, `True`, `False`, and user-defined sentinels). For everything else, `is` tests *identity*, not *value*.

### Membership: `in` and `not in`

The `in` and `not in` operators work with **iterables** and **containers** to test whether a value is present:

```python
1 in [1, 2, 3]              # True   (list — linear scan)
[1, 2] in [1, 2, 3]         # False  (element-wise: is [1,2] an *element*?)
[1, 2] in [[1,2], [2,3], 'abc']   # True   (the list [1,2] is an element)

'key1' in {'key1': 1, 'key2': 2}  # True   (dict — key lookup)
1 in {'key1': 1, 'key2': 2}       # False  (1 is not a *key*)
```

Two subtleties worth noting:

**1. Membership is not "subset".** `[1, 2] in [1, 2, 3]` asks whether the *list object* `[1, 2]` is one of the *elements* of `[1, 2, 3]` — not whether all of `[1, 2]`'s contents appear in `[1, 2, 3]`. Since the elements of `[1, 2, 3]` are the integers `1`, `2`, `3`, the answer is `False`.

To test subset-style membership:

```python
all(x in [1, 2, 3] for x in [1, 2])    # True   (all elements present)
set([1, 2]).issubset([1, 2, 3])        # True   (set-friendly)
```

**2. For `dict`, membership checks *keys*, not values.**

```python
'key1' in {'key1': 1, 'key2': 2}   # True   (a key)
1 in {'key1': 1, 'key2': 2}        # False  (a value, not a key)
```

If you need to check values:

```python
1 in {'key1': 1, 'key2': 2}.values()   # True
```

**3. Complexity depends on the container:**

| Container | Average `in` cost | Mechanism |
|---|---|---|
| `list`, `tuple` | O(n) | Linear scan |
| `set`, `frozenset` | O(1) amortized | Hash lookup |
| `dict` | O(1) amortized | Hash lookup |
| `str` | O(n·m) | Substring search |

This is the same complexity lesson as the "membership rewriting" optimization — sets and dicts use hash tables, which makes membership checks essentially constant-time.

---

## Equality Operators

`==` and `!=` are **value comparison** operators. They compare contents, not identities.

### Mixed-type equality works when meaningful

Python's `==` is flexible: it compares values across types when the comparison is well-defined:

```python
1 == '1'   # False   (an int is never equal to a str)
```

But numeric types are designed to compare with each other:

```python
from decimal import Decimal
from fractions import Fraction

Decimal('0.1') == Fraction(1, 10)   # True
1 == 1 + 0j                         # True    (int vs. complex)
True == Fraction(2, 2)              # True    (bool vs. Fraction — 1 == 1)
False == 0j                         # True    (0 == 0+0j)
```

### Why does this work? The numeric tower

Python's numeric types follow a **coercion hierarchy**:

```text
    bool  ⊂  int  ⊂  float  ⊂  complex
              ↑
          Fraction, Decimal (participate in cross-type comparison)
```

When comparing two numeric values of different types, Python's `__eq__` implementations are designed to **coerce to a common representation** and compare numerically. The result is mathematically correct — `Decimal('0.1')` and `Fraction(1, 10)` both represent exactly one tenth, so they compare equal.

### But it does *not* work across unrelated types

```python
1 == '1'   # False   (no sensible coercion between int and str)
```

A string and an integer simply aren't comparable in a value sense — Python returns `False` rather than raising an error. This is one of the "strong typing" properties we saw earlier: incompatible types don't silently coerce.

### What `==` actually does

Behind the scenes, `a == b` roughly does:

1. Try `a.__eq__(b)`.
2. If it returns `NotImplemented`, try `b.__eq__(a)` (reflected comparison).
3. If both return `NotImplemented`, fall back to `a is b` (identity).
4. Otherwise, use `bool(...)` on the result.

For most types, this means **value comparison**. For some custom types, `__eq__` may be overridden (or may not exist — in which case identity is used).

---

## Ordering Comparisons

`<`, `<=`, `>`, `>=` compare **ordering** — but not every type supports ordering.

### Complex numbers cannot be ordered

```python
1 + 1j < 2 + 2j
# TypeError: '<' not supported between instances of 'complex' and 'complex'
```

There is no natural total order on complex numbers — the complex plane is two-dimensional. Python's designers chose to **refuse** rather than invent an arbitrary order.

### Mixed-type ordering requires meaningfulness

```python
1 < 'a'
# TypeError: '<' not supported between instances of 'int' and 'str'
```

An integer and a string have no sensible ordering. Python raises `TypeError` instead of guessing.

But numeric types remain comparable:

```python
Decimal('0.1') < Fraction(1, 2)
# → True    (0.1 < 0.5)
```

The same coercion rules as equality apply — *if* the types participate in the numeric tower.

### What about `str` and `bytes`?

Strings **do** support ordering — by Unicode code point:

```python
'a' < 'b'    # True
'A' < 'a'    # True    (uppercase letters have lower code points)
'ab' < 'abc'  # True   (prefix is smaller)
```

This is why comparisons like `'Apple' < 'banana'` are legal — and why casing matters. (For case-insensitive or locale-aware comparison, use `str.casefold()` or `locale.strcoll()`.)

### Summary: ordering rules

| Comparison | Result |
|---|---|
| Same numeric type | ✅ Ordered |
| Different numeric types (int, float, Fraction, Decimal) | ✅ Ordered (with coercion) |
| Complex vs anything | ❌ `TypeError` |
| `str` vs `str` | ✅ Ordered (by code point) |
| `int` vs `str` | ❌ `TypeError` |
| `str` vs `bytes` | ❌ `TypeError` |
| Custom class with `__lt__` defined | ✅ Ordered according to `__lt__` |
| Custom class without `__lt__` | ❌ `TypeError` |

---

## Chained Comparisons

This is where Python shows one of its most elegant — and most misread — features.

### The rule

> **`a < b < c`** is evaluated as **`a < b and b < c`**.

Not as `(a < b) < c`. Not as `a < (b < c)`. The comparison **chains** — each middle operand is compared to both its neighbors.

```python
1 < 2 < 3     # True   (1 < 2 and 2 < 3)
```

### A more complex example

```python
1 < 2 > -5 < 50 > 4
# → True
```

Desugared:

```text
    1 < 2  and  2 > -5  and  -5 < 50  and  50 > 4
    True   and  True    and  True     and  True
    → True
```

Each adjacent pair is joined by `and` — the direction of each operator is independent.

### Mixed types across the chain

```python
1 < 2 == Decimal('2.0')
# → True
```

Desugars to:

```text
    1 < 2           and   2 == Decimal('2.0')
    True            and   True
    → True
```

The `2` on the right of `<` is the *same* `2` compared to `Decimal('2.0')` via `==`. Both comparisons succeed.

### Mixed operator chain

```python
import string
'A' < 'a' < 'z' > 'Z' in string.ascii_letters
# → True
```

Desugars to:

```text
    'A' < 'a'                    and
    'a' < 'z'                    and
    'z' > 'Z'                    and
    'Z' in string.ascii_letters
```

Each of these is `True`:
- `'A' < 'a'` → `True` (uppercase has lower code point)
- `'a' < 'z'` → `True` (alphabetical)
- `'z' > 'Z'` → `True` (lowercase has higher code point)
- `'Z' in string.ascii_letters` → `True` (Z is in the ASCII letters set)

`and`-ing them all yields `True`.

### Why chaining is a feature

Chained comparisons eliminate the clumsy repetition of the middle operand:

```python
# Without chaining:
if 0 <= x and x < 10:
    ...

# With chaining:
if 0 <= x < 10:
    ...
```

The chained form reads like mathematical notation and evaluates each middle operand only once — a genuine win over `0 <= x and x < 10` (which would evaluate `x` twice).

### The cost: it's easy to misread

Because `a == b == c` desugars to `a == b and b == c`, it does **not** mean "does `a == b` equal `c`?" — a common trap for newcomers and even experienced developers:

```python
1 == 2 == False        # False   (chained comparison)
(1 == 2) == False      # True    (explicit comparison)
```

Different expressions — different results. **Parenthesize when you intend the second meaning**, and remember that chaining is **always `and`-joined pairwise comparisons**, never nested comparisons.

---

## A Complete Mental Model

```text
    Comparison operator families:

    ┌──────────────────────────────────────────────────────────────┐
    │ Identity (is, is not)                                        │
    │   Compares: memory addresses                                 │
    │   Works with: any type                                       │
    │   Reliable for: None, True, False, custom sentinels          │
    │                                                              │
    │ Membership (in, not in)                                      │
    │   Compares: value presence in iterable/container             │
    │   Cost: O(1) for set/dict;  O(n) for list/tuple;  O(n·m) str │
    │   For dict: keys only (not values)                           │
    │                                                              │
    │ Equality (==, !=)                                            │
    │   Compares: values                                           │
    │   Coerces: numeric tower (bool/int/float/complex/Fraction/…) │
    │   Never coerces: unrelated types (int vs str → False)        │
    │                                                              │
    │ Ordering (<, <=, >, >=)                                      │
    │   Compares: rank                                             │
    │   Supported: numerics (except complex), strings, bytes,      │
    │              any type that defines __lt__/__le__/…           │
    │   Not supported: complex, mixed unrelated types              │
    │                                                              │
    │ Chaining (a op b op c …)                                     │
    │   Desugars to pairwise AND:                                  │
    │       a op1 b  and  b op2 c  and  …                          │
    │   Each middle operand compared to both neighbors             │
    │   Parenthesize to avoid accidental chaining                  │
    └──────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Key fact |
|---|---|
| `is` / `is not` | Identity comparison; works for any type; reliable for singletons |
| `in` / `not in` | Membership test; O(1) for set/dict, O(n) for list/tuple; dict checks keys |
| `==` / `!=` | Value comparison; coerces across numeric types; cross-type non-numeric → `False` |
| `<` / `>` / `<=` / `>=` | Ordering; unsupported for complex and mixed unrelated types |
| Chained comparisons | `a < b < c` means `a < b and b < c` — not `(a < b) < c` |
| Parenthesize | When you mean "the result of a comparison, compared to another value" |

The four families each encode a fundamentally different question — identity, membership, value, ordering. Knowing which operator answers which question is the foundation for writing comparisons that are both correct and meaningful. The chaining rule ties back to the same lesson from the boolean section: **`and`-joining is the language's default way of combining comparisons** — and treating `a op b op c` as anything other than `a op b and b op c` is the single most common source of confusion.