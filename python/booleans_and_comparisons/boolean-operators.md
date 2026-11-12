# Booleans: Boolean Operators in Python

Python's `and` and `or` operators behave differently from what you might expect if you're coming from languages where logical operators always return `True` or `False`. In Python, they **return one of their operands**, not a boolean. This is called **operand-returning behavior**, and it's tightly coupled with **short-circuit evaluation**.

Understanding this deeply is important because it's the basis for many idiomatic Python patterns (default values, safe attribute access, guard clauses).

---

## The `or` Operator

### Rule

> `X or Y`:
> - If `X` is **truthy** → return `X` (and `Y` is **not evaluated**)
> - If `X` is **falsy** → evaluate and return `Y`

### Examples

```python
'' or 'abc'          # → 'abc'
0 or 100             # → 100
[] or [1, 2, 3]      # → [1, 2, 3]
[1, 2] or [1, 2, 3]  # → [1, 2]
```

Notice that the result is **never converted to a boolean**. `0 or 100` returns the integer `100`, not `True`.

### Key Insights

1. **The truthiness of `Y` is never considered** when computing the result of `or`. Only the left operand decides *which* operand is returned.
2. `Y` is only **evaluated** if it needs to be returned. If `X` is truthy, `Y` is skipped entirely — this is **short-circuiting**.
3. The returned value's truthiness doesn't matter — the operator simply hands back whichever operand "wins."

### Almost Reimplementing `or`

We could write a function that behaves almost identically:

```python
def _or(x, y):
    if x:
        return x
    else:
        return y
```

```python
print(_or(0, 100) == (0 or 100))          # True
print(_or(None, 'n/a') == (None or 'n/a')) # True
print(_or('abc', 'n/a') == ('abc' or 'n/a')) # True
```

### Why "Almost"?

The crucial difference is **evaluation timing**:

- The built-in `or` operator is **short-circuiting** — the right operand is only evaluated when needed.
- Our `_or` function is **not short-circuiting** — both arguments are evaluated *before* the function is even called.

This distinction matters when the right operand has **side effects** or can **raise exceptions**:

```python
1 or 1/0
# → 1  (1/0 is never evaluated — no error)
```

```python
_or(1, 1/0)
# ZeroDivisionError: division by zero
# (1/0 is evaluated eagerly, before the call)
```

> **Takeaway:** Short-circuiting isn't just an optimization — it's a semantic guarantee. It lets you write expressions like `x is not None and x.attr` safely.

---

## The `and` Operator

### Rule

> `X and Y`:
> - If `X` is **falsy** → return `X` (and `Y` is **not evaluated**)
> - If `X` is **truthy** → evaluate and return `Y`

### Examples

```python
s1 = None
s2 = ''
s3 = 'abc'

print(s1 and s1[0])   # None   (s1[0] never evaluated — would raise TypeError)
print(s2 and s2[0])   # ''     (s2[0] never evaluated — would raise IndexError)
print(s3 and s3[0])   # 'a'    (s3[0] is safe here)
```

Notice how `and` acts as a **guard**: if the left side is falsy, the right side — which might be unsafe — is never touched.

### Combining `and` + `or` for Defaults

A very common idiom is `(X and Y) or default`:

```python
print((s1 and s1[0]) or '')  # ''
print((s2 and s2[0]) or '')  # ''
print((s3 and s3[0]) or '')  # 'a'
```

Or with a meaningful default:

```python
print((s1 and s1[0]) or 'n/a')  # 'n/a'
print((s2 and s2[0]) or 'n/a')  # 'n/a'
print((s3 and s3[0]) or 'n/a')  # 'a'
```

This pattern — **"safe access with fallback"** — works because:
- `s and s[0]` → safely gets the first character (or returns `s` if falsy)
- `... or 'n/a'` → supplies a fallback if the previous step produced a falsy result

⚠️ **Caveat:** This breaks down if `s[0]` itself can be a "legitimately falsy" value (e.g., the string `"0"` is truthy, but the integer `0` isn't). In modern Python, prefer:

```python
s[0] if s else 'n/a'
```

or the `or`-with-`None`-check pattern:

```python
(s[0] if s else None) or 'n/a'
```

---

## The `not` Operator

Unlike `and`/`or`, `not` **always returns a genuine `bool`**:

```python
not 'abc'   # False
not []      # True
not None    # True
```

It applies `bool()` to its operand and negates the result:

```python
bool(None)  # False
not None    # True
```

### Quick Reference: Truthiness

| Value | Truthy? |
|---|---|
| `0`, `0.0`, `0j` | ❌ Falsy |
| `''`, `[]`, `{}`, `set()`, `()` | ❌ Falsy |
| `None`, `False` | ❌ Falsy |
| Non-zero numbers | ✅ Truthy |
| Non-empty containers/strings | ✅ Truthy |
| Any object (by default) | ✅ Truthy |

---

## Summary Table

| Expression | Returns | Right side evaluated? |
|---|---|---|
| `X or Y` | `X` if `X` truthy, else `Y` | Only if `X` is falsy |
| `X and Y` | `X` if `X` falsy, else `Y` | Only if `X` is truthy |
| `not X` | `bool` (negated truthiness) | Always |

---

## Mental Model

Think of `and`/`or` as **"conditional expression selectors"** rather than boolean combiners:

```text
X or Y   →   "X, unless X is falsy, in which case Y"
X and Y  →   "X, unless X is truthy, in which case Y"
```

Both operators:
- **Return an operand** (never coerce to `bool`)
- **Short-circuit** (skip the right side when it can't affect the result)

Once you internalize this, patterns like `name = user_input or "Anonymous"`, `result = obj and obj.method()`, and `value = a and b or c` stop feeling like tricks and start reading like plain English.