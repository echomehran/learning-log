# The Mutable Default Argument Trap

## The Problem

Consider a logging function that should stamp each message with the current time:

```python
from datetime import datetime

def log(msg, *, dt=datetime.utcnow()):
    print(f"{dt}: {msg}")
```

At first glance, this looks correct. Every call without an explicit `dt` should use "the current time," right?

```python
log("message 1")
# 2017-08-22 04:04:18.406943: message 1

log("message 2")
# 2017-08-22 04:04:18.406943: message 2   ← same timestamp!
```

Every call produces the **same** timestamp — the exact moment the function was **defined**, not the moment it was called. Even minutes or hours later, `log("message 3")` still prints the function's definition time.

### Why this happens

Default values are evaluated **once**, when the `def` statement executes. They are **not** re-evaluated on each call.

```text
    def log(msg, *, dt=datetime.utcnow()):
                          ↑
                          │
                          └── evaluated ONCE, here, when
                              the `def` statement runs
```

The result of `datetime.utcnow()` is stored as the default value and reused on every subsequent call. It's not a recipe for "call `utcnow()` when needed" — it's the **snapshot** of what `utcnow()` returned at definition time.

This isn't a bug in the language — it's the intended behavior. Defaults are part of the function object itself, and the function object exists from the moment the `def` statement executes.

---

## The Correct Pattern: `None` as a Sentinel

Use `None` (or any sentinel) as the default, and compute the actual default **inside** the function, at call time:

```python
def log(msg, *, dt=None):
    dt = dt or datetime.utcnow()
    print(f"{dt}: {msg}")
```

Now each call without an explicit `dt` computes a fresh timestamp:

```python
log("message 1")
# 2017-08-22 04:15:11.797640: message 1

log("message 2")
# 2017-08-22 04:15:14.529496: message 2   ← different timestamp
```

And an explicit `dt` still works:

```python
log("message 3", dt="2001-01-01 00:00:00")
# 2001-01-01 00:00:00: message 3
```

### How `dt = dt or datetime.utcnow()` works

The expression `dt or datetime.utcnow()` returns:

- `dt` if `dt` is **truthy** (caller supplied a value)
- `datetime.utcnow()` if `dt` is **falsy** (`None` — caller omitted it)

This is the short-circuit `or` operator from the boolean lesson, applied to supply a default. The equivalent explicit form is clearer:

```python
def log(msg, *, dt=None):
    if dt is None:
        dt = datetime.utcnow()
    print(f"{dt}: {msg}")
```

### Why `is None` rather than `not dt`

The `dt or datetime.utcnow()` form is idiomatic and works **for this case** because a valid `datetime` is always truthy. But it has a subtle flaw: if a caller passes a **falsy but legitimate** value — `0`, `""`, `False`, an empty list — the `or` would silently replace it with the current time.

`is None` is more precise: it only replaces the value when it's actually `None`, preserving any other falsy values the caller might pass.

```python
# Safer: preserves any non-None value
if dt is None:
    dt = datetime.utcnow()
```

For `dt` (a `datetime` object, always truthy), the `or` form is fine. But for numeric, string, or collection defaults, `is None` is the correct idiom.

---

## The Same Trap with Mutable Defaults

The `datetime.utcnow()` example is one instance of a broader problem. The trap also manifests — **more dangerously** — with mutable defaults like `[]`, `{}`, and `set()`. These are shared across all calls, so mutations accumulate:

```python
def append_item(item, bucket=[]):
    bucket.append(item)
    return bucket
```

```python
append_item(1)   # [1]
append_item(2)   # [1, 2]      ← not [2]!
append_item(3)   # [1, 2, 3]   ← not [3]!
```

The list `[]` is created **once**, at definition time. Every call uses the same list object. This is the *same* mechanism as `datetime.utcnow()` — a default evaluated once — but the consequences are worse: instead of a stale timestamp, the state **accumulates silently**.

### The fix: same pattern

```python
def append_item(item, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket
```

Now each call without a `bucket` argument gets a fresh list.

### The general rule

**Never use a mutable value as a default argument.** Use `None` as a sentinel, then create the mutable object inside the function. The only exception is when you *want* the shared-state behavior (e.g., a cache), and you should make that intention explicit with a comment.

### Common offenders

| Default | Risk |
|---|---|
| `[]` | Appends accumulate across calls |
| `{}` | Keys accumulate across calls |
| `set()` | Elements accumulate across calls |
| `datetime.utcnow()` | Timestamp frozen at definition time |
| Any function call | Its return value is the default, computed once |

---

## Why Python Behaves This Way

Default values are part of the function's **definition**, which is evaluated when the `def` statement runs. They live on the function object:

```python
def f(x, y=[]):
    pass

print(f.__defaults__)
# ([],)   ← the default list is stored here, as an attribute of f
```

Every call with no explicit `y` receives that exact same list object, because `f.__defaults__[0]` is the default that gets bound. The function doesn't "remember to call `[]`" — the `[]` was already called, once, and its result is what's bound.

This is a direct consequence of the evaluation model: **`def` is an executable statement**. When it runs, it evaluates the default expressions and constructs the function object with those values. There's no notion of "deferred evaluation" for defaults — that would require a different language design.

### The conceptual mistake

The trap comes from reading the default as an **expression** — "if `y` isn't supplied, use `[]`" — when actually it's a **value**: "if `y` isn't supplied, use *this particular list object*, which was created once."

Once you see defaults as **pre-computed values baked into the function object**, the behavior stops being surprising. It's the only behavior consistent with eager evaluation.

---

## A Comparison with Other Languages

Different languages handle this differently:

| Language | Mutable default behavior |
|---|---|
| **Python** | Evaluated once at definition; shared across calls |
| **JavaScript** | No equivalent feature (each call re-evaluates a fresh default expression) |
| **Ruby** | Defaults evaluated per call |
| **Kotlin/Swift** | Default expressions evaluated per call |

Python is unusual here. The pattern has been debated since the language's early days, but changing it would break code that relies on the shared-state behavior (rare, but real). The `None` sentinel idiom has become the accepted solution — it's so standard that linters flag mutable defaults automatically.

---

## Summary

```text
The trap:
    def f(x, dt=datetime.utcnow()):       # dt frozen at definition
    def f(x, items=[]):                   # items shared across calls

Why:
    Default values are evaluated ONCE, when `def` executes.
    They are stored on the function object and reused on every call.
    "Default expression" is misleading — it's a pre-computed value.

The fix:
    Use None as a sentinel; compute the real default inside.

        def f(x, dt=None):
            if dt is None:
                dt = datetime.utcnow()

        def f(x, items=None):
            if items is None:
                items = []

Rules of thumb:
    - NEVER use a mutable value as a default argument.
    - NEVER use a function call as a default (it evaluates once).
    - Use None as the sentinel when the true default depends on call time.
    - Use "is None", not "not x", to avoid clobbering legitimate falsy values.
    - If shared state IS the intent (caches), document it explicitly.

Common offenders:
    []   {}   set()   datetime.utcnow()   any_function_call()
```

This is one of Python's most important "gotchas" — subtle, silent, and capable of producing bugs that are hard to trace because the state mutates invisibly across calls. The `None` sentinel pattern is the standard fix, and the underlying mental model (defaults are pre-computed values, not deferred expressions) is the key to predicting and preventing the issue.