# Booleans: Precedence and Short-Circuiting

## Operator Precedence

Consider:

```python
True or True and False
# → True
```

This evaluates as:

```python
True or (True and False)
# → True
```

Because `and` has **higher precedence** than `or`, it binds tighter and is grouped first.

It is **not** equivalent to:

```python
(True or True) and False
# → False
```

The difference is dramatic — same operands, opposite result. Precedence is not cosmetic; it changes semantics.

### Precedence Ladder (relevant subset)

From **highest** to **lowest**:

```text
not
and
or
```

So `not` binds tightest, then `and`, then `or`. Concretely:

```text
not A or B and C   ≡   (not A) or (B and C)
A or B and C       ≡   A or (B and C)
A and B or C       ≡   (A and B) or C
```

> **Rule of thumb:** When in doubt, **parenthesize**. Explicit grouping is free; debugging a mis-grouped boolean expression is not.

---

## Short-Circuit Evaluation

Short-circuiting is the runtime behavior that complements the precedence rules: the right operand is **only evaluated if needed**.

### Example 1: Avoiding Division by Zero

Without a guard:

```python
a = 10
b = 2

if a/b > 2:
    print('a is at least double b')
# → a is at least double b
```

Now change `b`:

```python
a = 10
b = 0

if a/b > 2:
    print('a is at least double b')
# ZeroDivisionError: division by zero
```

The expression `a/b > 2` is evaluated *before* the `if` decides anything — so the division happens first, and it explodes.

Add a guard using `and`:

```python
a = 10
b = 0

if b and a/b > 2:
    print('a is at least double b')
# (nothing printed — no error)
```

Here's what happens step by step:

```text
b and a/b > 2
  ↓
b = 0  → falsy
  ↓
short-circuit: return 0, never evaluate a/b
  ↓
if 0: → False → skip the block
```

The `b and ...` guard is essentially saying: *"only proceed to the dangerous check if `b` is truthy."* This is the single most common idiomatic use of short-circuiting in real Python code.

---

### Example 2: Guarding Against Empty or Null Strings

Short-circuiting is also very useful when dealing with nullable or empty string values — a common situation when reading from a database or user input.

The `string` module provides handy constants:

```python
import string

string.digits          # '0123456789'
string.ascii_letters   # 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
```

Consider a check like *"the name must not start with a digit"*:

```python
name = ''
if name[0] in string.digits:
    print('Name cannot start with a digit!')
# IndexError: string index out of range
```

The check is unsafe for an empty string because `name[0]` doesn't exist.

Now add a guard:

```python
name = ''
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
# (nothing printed — no error)
```

Why it works:

```text
name and name[0] in string.digits
  ↓
name = ''  → falsy
  ↓
short-circuit: return '', never evaluate name[0]
  ↓
if '': → False → skip the block
```

The same guard handles `None` cleanly:

```python
name = None
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
# (nothing printed — None is falsy)
```

And it still detects the real problem case:

```python
name = 'Bob'
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
# (nothing printed — 'B' is not a digit)

name = '1Bob'
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
# → Name cannot start with a digit!
```

The `name and ...` pattern is a lightweight, dependency-free **guard clause**. It makes the code robust against empty strings and `None` in a single expression, without an explicit `if name:` block or a `try/except IndexError`.

---

## Why Short-Circuiting Matters

Short-circuiting is not just a micro-optimization. It has **three real consequences**:

### 1. Safety — Preventing Exceptions

As shown above, `b and a/b` avoids `ZeroDivisionError`, and `name and name[0]` avoids `IndexError`. The right operand is never reached when it would be unsafe.

### 2. Performance — Skipping Expensive Work

```python
if cache and cache.is_valid():
    ...
```

If `cache` is `None`, there's no point in calling `is_valid()`. Short-circuit skips the potentially expensive method call.

### 3. Side Effects — Controlling What Runs

Because Python evaluates left-to-right and stops early, you can **rely** on the right operand not running when the left short-circuits. This is a semantic guarantee, not an implementation detail.

---

## Short-Circuiting in `and` vs `or`

Both operators short-circuit, but in **opposite directions**:

| Operator | When it short-circuits | Which side is skipped |
|---|---|---|
| `X and Y` | When `X` is **falsy** | `Y` is skipped (not evaluated) |
| `X or Y` | When `X` is **truthy** | `Y` is skipped (not evaluated) |

So:

- `and` short-circuits on the **falsy** path.
- `or` short-circuits on the **truthy** path.

This is why:

- `b and a/b` works — if `b` is falsy, we skip the division.
- `name or 'n/a'` works — if `name` is truthy, we skip the default.

---

## Putting It Together: A Mental Model

```text
Precedence:   not  >  and  >  or

Evaluation:   left  →  right

Short-circuit:
    and: stop when left is falsy   (right side is unsafe/unneeded)
    or:  stop when left is truthy  (we already have a usable value)

Idioms:
    guard:      if x and x.attr: ...
    default:    value = x or default
    safe access: (obj and obj.attr) or fallback
```

Once you internalize both **precedence** (how the expression groups) and **short-circuiting** (which parts actually run), boolean expressions stop looking like cryptic one-liners and start reading as clear, defensible control flow.

---

## Summary

- **Precedence:** `not` binds tightest, then `and`, then `or`. Use parentheses when in doubt.
- **Short-circuiting:** the right operand of `and`/`or` is evaluated **only if needed**.
- `X and Y` → returns `X` if falsy, else `Y`.
- `X or Y` → returns `X` if truthy, else `Y`.
- The `x and <dangerous expression>` pattern is the idiomatic way to **guard** against `None`, empty values, and zero divisors.
- The `x or <default>` pattern is the idiomatic way to supply a **fallback** value.