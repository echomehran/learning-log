# Booleans

## What `bool` Actually Is

The **`bool`** class represents boolean values. Two facts define it:

1. **`bool` is a subclass of `int`.**
2. **`True` and `False` are its only instances** — singletons with underlying integer values `1` and `0`.

```python
issubclass(bool, int)
# → True
```

This is the foundation for everything that follows: booleans in Python are *not* a separate primitive type. They're integers with a specialized class that restricts construction to two instances.

---

## The Two Singletons

```python
type(True), id(True), int(True)
# → (bool, 1658060976, 1)

type(False), id(False), int(False)
# → (bool, 1658061008, 0)
```

- **Type** is `bool`.
- **Identity** is fixed — the same address every time.
- **Integer value** is `1` or `0`.

Because of inheritance, `True` and `False` are instances of *both* `bool` and `int`:

```python
isinstance(True, bool)   # True
isinstance(True, int)    # True
```

This dual identity is the source of nearly every surprising behavior in this lesson.

---

## Singletons Enable Identity Comparison

Since `True` and `False` are the **only** instances of `bool`, any boolean expression that produces `True` must return the *same* `True` object — not a copy:

```python
id(True), id(1 < 2)
# → (1658060976, 1658060976)   ← same address

id(False), id(1 == 3)
# → (1658061008, 1658061008)   ← same address
```

Therefore, `is` and `==` **both** work for comparing to `True` and `False`:

```python
(1 < 2) is True, (1 < 2) == True     # (True, True)
(1 == 2) is False, (1 == 2) == False # (True, True)
```

This is one of the rare cases where `is` is a safe equality check — because the value is guaranteed to be a singleton. (Contrast with integers and strings, where interning is an implementation detail and `is` can fail.)

> **Style note:** `if x is True:` is considered redundant in Python — `if x:` is idiomatic, and `if x is True:` is discouraged unless you genuinely need to distinguish `True` from other truthy values. Use `is` only for `None`, `True`, `False` when *identity* is genuinely what you mean — e.g., when a parameter must be exactly `True`, not `1` or `"hello"`.

---

## The Chained Comparison Trap

Here's a subtle bug that catches nearly everyone the first time:

```python
1 == 2 == False
# → False
```

But:

```python
(1 == 2) == False
# → True
```

**Same characters. Different meaning.** The parentheses change everything.

### Why?

Python supports **chained comparisons**. An expression like `a == b == c` is evaluated as:

```text
a == b == c    ≡    a == b and b == c
```

Not as `(a == b) == c`. This is a deliberate design choice — it lets you write `0 < x < 10` naturally, where you'd otherwise need `0 < x and x < 10`.

So:

```python
1 == 2 == False
# is actually:  1 == 2 and 2 == False
# evaluates as: False and False
# → False
```

Breaking it down:

```python
1 == 2,            # False
2 == False,        # False   (2 != 0)
1 == 2 and 2 == False   # False
```

All three components are `False` — hence the final `False`.

### But with parentheses

```python
(1 == 2)  # → False
```

So `(1 == 2) == False` becomes `False == False`, which is `True`.

### Visual comparison

```text
    1 == 2 == False             (chained comparison)
        ↓  desugars to
    1 == 2 and 2 == False
        ↓
    False and False
        ↓
    False

    (1 == 2) == False           (parenthesized)
        ↓  evaluates the inner expression first
    False == False
        ↓
    True
```

The difference isn't a quirk — it's the *same* chaining rule that makes `0 < x < 10` work. The lesson: **chained comparisons and comparisons-as-values look similar but mean different things.** When in doubt, parenthesize.

---

## `True` and `False` Are Also Their Integer Values

Since `False` has underlying value `0`, and `bool` inherits from `int`, comparing a boolean expression to `0` is equivalent to comparing it to `False`:

```python
(1 == 2) == 0
# → True    ← False == 0

(1 == 2) == False
# → True    ← also True, but clearer
```

The underlying integer values:

```python
int(True), int(False)
# → (1, 0)
```

So as equality comparisons:

```python
1 == True, 0 == False
# → (True, True)
```

But — and this is the crucial distinction — **equality is not identity**:

```python
1 == True,   1 is True
# → (True, False)   ← equal values, different objects

0 == False,  0 is False
# → (True, False)   ← equal values, different objects
```

`True` and `1` have the same *value* but are different *objects* (`bool` vs `int`). `False` and `0` likewise. The distinction between `==` and `is` — established in previous lessons — holds firmly here.

---

## Converting Integers to Booleans

Any integer can be cast to `bool`, following a simple rule:

> **`bool(x)` is `True` for any non-zero `x`, and `False` for `0`.**

```python
bool(0)              # False
bool(1), bool(100), bool(-1)   # (True, True, True)
```

Note that **negative numbers are truthy** — the rule is "non-zero," not "positive."

This is why `if n:` is the idiomatic way to check "is `n` non-zero?" — it works for any numeric type (`int`, `float`, `complex`, `Fraction`, `Decimal`) uniformly.

---

## Booleans Behave Like Integers

Because `bool` is a subclass of `int`, and Python uses **polymorphism** via inheritance, all standard integer operators, properties, and methods are available on booleans:

```python
True > False          # True     (1 > 0)
True + 2              # 3        (1 + 2)
False // 2            # 0        (0 // 2)
True + True + True    # 3        (1 + 1 + 1)
(True + True + True) % 2   # 1
-True                 # -1       (-1)
100 * False           # 0        (100 * 0)
```

Each operation treats `True` as `1` and `False` as `0`. The result is a plain `int` — not a `bool`. So `True + 2` returns `3` (an `int`), not `True`.

This is polymorphism in action: because `bool` inherits from `int`, all of `int`'s dunder methods (`__add__`, `__mul__`, `__floordiv__`, `__neg__`, …) are inherited and work using the underlying values `1` and `0`.

### The double-edged sword

This inheritance is genuinely useful — for example, it makes `sum([True, False, True])` return `2` (a count of `True` values), which is an elegant way to count matches:

```python
sum(x > 3 for x in [1, 5, 2, 7, 4])
# → 2    (two values greater than 3)
```

But it can also produce code that is *technically correct* yet hard to read:

```python
True + True + True       # 3
100 * False              # 0
-True                    # -1
```

> I certainly **do not recommend** writing code like this — but be aware that it works, and understand *why* it works. Code whose behavior depends on `bool` being `int` is fragile under refactoring and confusing to readers who don't know the inheritance.

The idiomatic patterns — `if x:`, `sum(condition for ...)`, `not x` — are clearer *and* idiomatic. Arithmetic on booleans should be reserved for cases like the `sum` trick, where the intent (counting) is obvious.

---

## A Complete Mental Model

```text
    bool
      │
      └── inherits from int
      │
      ├── has exactly two instances: True, False
      │
      ├── True  ≡ int value 1  (singleton)
      └── False ≡ int value 0  (singleton)

    Consequences:

        1. is / == both work for True and False
           (because they're singletons — one object each)

        2. True == 1  but  True is not 1
           (equal values, different objects)

        3. Chained comparisons desugar:
                a == b == c   ≡   a == b and b == c
           (NOT (a == b) == c)

        4. bool(x) is True for all non-zero x, False for x == 0

        5. Booleans support integer arithmetic:
                True + True = 2,  -True = -1,  100 * False = 0
           (because all int operators are inherited)

        6. isinstance(True, int) is True
           (subclass relationship — the root of both
            the flexibility and the pitfalls)
```

---

## Summary

| Concept | Fact |
|---|---|
| `bool` inherits from `int` | `issubclass(bool, int) is True` |
| Only instances | `True`, `False` |
| Underlying values | `True = 1`, `False = 0` |
| Singletons | `id(True)` and `id(False)` are stable |
| Comparison | `is` and `==` both work for `True`/`False` |
| `True == 1` | ✅ `True` |
| `True is 1` | ❌ `False` |
| `bool(x)` | `False` iff `x == 0` |
| Chained comparisons | `a == b == c` means `a == b and b == c` |
| Parenthesize | `(a == b) == c` is different from `a == b == c` |
| Arithmetic | Booleans work as integers `1` and `0` — but don't rely on it in production code |

Booleans are a study in how inheritance, singletons, and operator semantics combine. `bool` inheriting from `int` is what makes booleans arithmetic-compatible, singletons are what make `is` safe for them, and the chaining rule that surprises newcomers is the same rule that makes `0 < x < 10` elegant. Understanding all three — and knowing which of them to *use* (identity checks, truthiness) versus which to *avoid* (arithmetic, `is True` in conditionals) — is the difference between code that exploits the language's design and code that merely happens to work.