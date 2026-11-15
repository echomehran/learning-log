# Parameter Combinations: A Complete Catalog

Every function signature is a combination of the same building blocks:

- **Required positional** parameters — must be supplied, can be passed positionally or by name.
- **Optional positional** parameters — have defaults, can be omitted.
- **`*args`** — captures any remaining positional arguments as a tuple.
- **Keyword-only** parameters — must be supplied by name (required or defaulted).
- **`**kwargs`** — captures any remaining keyword arguments as a dict.

The combinations below are ordered roughly by increasing flexibility. Each row in this catalog is a concrete signature you can copy and adapt.

---

## 1. Positional Only — No Defaults, No Extras

The simplest signature: every argument is required, no defaults, no catch-alls.

```python
def func(a, b):
    print(a, b)

func('hello', 'world')       # hello world
func(b='world', a='hello')   # hello world — keyword passing still works
```

The parameters are positional *in definition* (they fill left-to-right when passed positionally) but the caller can still use keyword arguments. "Positional-only" in this context means "no `*` — the parameters can be passed positionally"; it does **not** mean keyword passing is forbidden.

---

## 2. Positional Only — With Defaults

Some parameters optional; the required ones must still come first.

```python
def func(a, b='world', c=10):
    print(a, b, c)

func('hello')          # hello world 10
func('hello', c='!')   # hello world !
```

The rule from the earlier lesson applies: **once a parameter has a default, every parameter after it must also have a default.** `def func(a='x', b): ...` is a `SyntaxError`.

---

## 3. Positional Only — Extra Positionals

`*args` captures everything beyond the named positionals.

```python
def func(a, b, *args):
    print(a, b, args)

func(1, 2, 'x', 'y', 'z')   # 1 2 ('x', 'y', 'z')
```

`args` is an empty tuple if the caller supplies only `a` and `b`.

### The call-site rule still applies

```python
func(b=2, a=1, 'x', 'y', 'z')
# SyntaxError: positional argument follows keyword argument
```

Once you use a keyword argument, you cannot return to positionals. The `*args` parameter doesn't change that — it just captures positionals that come **before** any keyword arguments in the call.

---

## 4. Keyword Only — No Defaults

No positional parameters at all. The bare `*` opens the keyword-only section.

```python
def func(*, a, b):
    print(a, b)

func(a=1, b=2)   # 1 2
func(1, 2)       # TypeError: takes 0 positional arguments but 2 were given
```

Both `a` and `b` are required and must be named. Positional calls are rejected outright.

---

## 5. Keyword Only — With Defaults

Some keyword-only parameters optional; required ones stay required.

```python
def func(*, a=1, b):
    print(a, b)

func(a=10, b=20)   # 10 20
func(b=2)          # 1 2   (a uses default)
```

**The ordering rule is different here.** For keyword-only parameters, required and defaulted can appear in **any order**:

```python
def func(*, a=1, b):    # ✅ legal
    ...
```

Compare to positional parameters, where `def func(a=1, b): ...` is a `SyntaxError`. The reason: keyword arguments are looked up by name, so order has no binding significance. There's no ambiguity to resolve with an ordering rule.

---

## 6. Positional + Keyword-Only

The most common signature in modern Python APIs: required positionals, then required keyword-onlys.

```python
def func(a, b, *, c, d):
    print(a, b, c, d)

func(1, 2, c=3, d=4)   # 1 2 3 4
func(1, 2, d=4, c=3)   # 1 2 3 4 — keyword order doesn't matter
func(1, c=3, d=4, b=2) # 1 2 3 4 — b by name, still positional slot
```

Notice the third call: `b=2` is a keyword argument for a **positional** parameter. That's legal (positional parameters can be passed by name), and it doesn't violate the rule that keyword-only parameters must be named — the rule is one-directional.

But `func(1, 2, 3, 4)` would fail — `c` and `d` must be named.

---

## 7. Positional + Keyword-Only with Defaults

Same as #6, with some keyword-only parameters defaulted:

```python
def func(a, b=2, *, c, d=4):
    print(a, b, c, d)

func(1, c=3)                # 1 2 3 4
func(c=3, a=1)              # 1 2 3 4
func(1, 2, c=3, d=4)        # 1 2 3 4
func(c=3, a=1, b=2, d=4)    # 1 2 3 4
```

Every call supplies `a` (required) and `c` (required keyword-only). `b` and `d` default if omitted. Every combination that fills the two required slots produces the same result.

---

## 8. Positional + `*args` + Keyword-Only

The first genuinely flexible signature — extra positionals and required keyword-onlys together:

```python
def func(a, b=2, *args, c=3, d):
    print(a, b, args, c, d)

func(1, 2, 'x', 'y', 'z', c=3, d=4)
# 1 2 ('x', 'y', 'z') 3 4
```

### The trap with defaults + `*args`

Consider this call:

```python
func(1, 'x', 'y', 'z', c=3, d=4)
# 1 x ('y', 'z') 3 4
```

`b` gets `'x'`, not its default `2`. Why? Because `'x'` is a positional argument, and `b` is a positional parameter with a default. **Defaults apply only when no positional argument fills the slot.** As soon as you supply a value positionally, it goes into `b`; then the rest go into `args`.

The lesson: **a defaulted positional parameter is filled by the first positional argument if one is supplied — regardless of whether you intended to "skip" it.** There's no way to say "use the default for `b`, but put `'x'` in `args`" — positionals always fill from left to right, from the earliest unfilled slot.

---

## 9. Positional + Keyword-Only + `**kwargs`

Extra keyword arguments without extra positionals:

```python
def func(a, b, *, c, d=4, **kwargs):
    print(a, b, c, d, kwargs)

func(1, 2, c=3, x=100, y=200, z=300)
# 1 2 3 4 {'x': 100, 'y': 200, 'z': 300}

func(x=100, y=200, z=300, c=3, b=2, a=1)
# 1 2 3 4 {'x': 100, 'y': 200, 'z': 300}
```

The second call shows that keyword arguments (for parameters like `a`, `b`, `c`) can come in any order — Python matches them by name. `x`, `y`, `z` don't match any named parameter, so they land in `kwargs`. The two calls are equivalent.

### Named parameters win over `**kwargs`

If a keyword argument matches a named parameter, it binds to that parameter, not to `kwargs`:

```python
func(1, 2, c=3, d=5, x=100)
# c=3 binds to the keyword-only parameter c — not to kwargs
# kwargs={'x': 100}
```

This is the precedence rule: **named parameters first, then `**kwargs`**.

---

## 10. The Most General Signature: Everything Combined

All five kinds of parameter in one signature:

```python
def func(a, b, *args, c, d=4, **kwargs):
    print(a, b, args, c, d, kwargs)

func(1, 2, 'x', 'y', 'z', c=3, d=5, x=100, y=200, z=300)
# 1 2 ('x', 'y', 'z') 3 5 {'x': 100, 'y': 200, 'z': 300}
```

Breaking it down:

| Parameter | Kind | Bound value |
|---|---|---|
| `a` | required positional | `1` |
| `b` | required positional | `2` |
| `args` | extra positionals (tuple) | `('x', 'y', 'z')` |
| `c` | required keyword-only | `3` |
| `d` | defaulted keyword-only (default 4) | `5` |
| `kwargs` | extra keywords (dict) | `{'x': 100, 'y': 200, 'z': 300}` |

This is the maximal signature. Every caller obligation can be expressed: which arguments must be positional, which must be keyword, which are optional, and what happens to the extras.

---

## 11. Only Catch-Alls

If a function wants to forward **everything** without naming any parameters:

```python
def func(*args, **kwargs):
    print(args, kwargs)

func(1, 2, 3, x=100, y=200, z=300)
# (1, 2, 3) {'x': 100, 'y': 200, 'z': 300}
```

This is the signature used for decorators, wrappers, and pass-through functions — anything that accepts any combination of arguments and forwards them:

```python
def log_call(fn):
    def wrapper(*args, **kwargs):
        print(f"calling {fn.__name__}")
        return fn(*args, **kwargs)
    return wrapper
```

The wrapper works with **any** function, regardless of that function's signature.

---

## The Fixed Order of Parameter Kinds

All of these fit into one grammar:

```text
    ┌─────────────────────────────────────────────────────────────────┐
    │  def f(                                                         │
    │       req_pos,              ← required positional               │
    │       opt_pos=...,          ← defaulted positional              │
    │       *args,                ← extra positionals (tuple)         │
    │       kw_only,              ← required or defaulted             │
    │       kw_only=...,          ←   keyword-only (any order)        │
    │       **kwargs              ← extra keywords (dict)             │
    │  )                                                              │
    └─────────────────────────────────────────────────────────────────┘

    Groups appear in a FIXED order. Within each group:
        positional:       required before defaulted
        keyword-only:     any order
        *args / **kwargs: named arbitrarily

    Nothing can follow **kwargs.
    A bare * (no name) is allowed; it opens the keyword-only section.
```

The ordering is enforced at parse time — a signature that violates it raises `SyntaxError` before the function is even defined.

---

## A Concrete Example: `print`

The built-in `print` is a perfect illustration of a real signature that uses every feature:

```python
help(print)
# print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
```

In Python's actual signature:

```python
def print(*values, sep=' ', end='\n', file=sys.stdout, flush=False):
    ...
```

- `*values` — any number of positional values to print (including zero).
- `sep=' '` — keyword-only, defaulted.
- `end='\n'` — keyword-only, defaulted.
- `file=sys.stdout` — keyword-only, defaulted.
- `flush=False` — keyword-only, defaulted.

The signature is a **direct application** of the grammar above. Every keyword-only parameter has a sensible default, so `print(1, 2, 3)` works without any keyword arguments. But when you want to customize behavior, the parameters are right there:

```python
print(1, 2, 3)                    # 1 2 3
print(1, 2, 3, sep='--')          # 1--2--3
print(1, 2, 3, end='***\n')       # 1 2 3***
print(1, 2, 3, sep='\t', end='\t***\t')
# 1	2	3	***	
```

### Why this signature is well-designed

- **`*values` is required** — print without values is legal (prints an empty line).
- **Keyword-only parameters are optional** — the defaults produce the expected output, so simple calls stay simple.
- **No possibility of accidental misordering** — you can't accidentally pass `'--'` as the second value and have it treated as `sep`; you must write `sep='--'` explicitly.

This is the recommended pattern for any function with "options" parameters: **put the data as `*args` (or as required positionals), and put the options as keyword-only with defaults.**

---

## A Real-World Use Case

A function that computes stats and optionally logs them:

```python
def calc_hi_lo_avg(*args, log_to_console=False):
    hi = int(bool(args)) and max(args)
    lo = int(bool(args)) and min(args)
    avg = (hi + lo) / 2
    if log_to_console:
        print("high={0}, low={1}, avg={2}".format(hi, lo, avg))
    return avg
```

```python
avg = calc_hi_lo_avg(1, 2, 3, 4, 5)
print(avg)   # 3.0

avg = calc_hi_lo_avg(1, 2, 3, 4, 5, log_to_console=True)
# high=5, low=1, avg=3.0
# 3.0
```

The pattern:

- **`*args`** — any number of numeric values, no fixed count.
- **`log_to_console=False`** — keyword-only flag, off by default.

Callers who want logging opt in with `log_to_console=True`. Callers who don't simply omit it. The "options" parameter is impossible to pass positionally — `calc_hi_lo_avg(1, 2, 3, 4, 5, True)` would put `True` in `args`, where it would corrupt the `max`/`min` computation (if it didn't raise `TypeError` first). The keyword-only marker prevents the bug entirely.

### A small aside: `int(bool(args)) and max(args)`

The line `hi = int(bool(args)) and max(args)` handles the empty-`args` case:

- If `args` is empty, `bool(args)` is `False`, `int(False)` is `0`, and `0 and ...` short-circuits to `0`.
- If `args` is non-empty, `bool(args)` is `True`, `int(True)` is `1`, and `1 and max(args)` evaluates to `max(args)`.

This idiom relies on two earlier lessons: the short-circuit behavior of `and`, and the int-truthiness of `bool` values. It's clever but a bit opaque — for clarity, `max(args) if args else 0` would be more readable. The point of the example is the *signature*, not the implementation.

---

## Summary

| Signature | Positionals | Keyword-only | `*args` | `**kwargs` | Notes |
|---|---|---|---|---|---|
| `def f(a, b)` | required | — | — | — | Simplest |
| `def f(a, b=2, c=3)` | required, defaulted | — | — | — | Defaults |
| `def f(a, b, *args)` | required + extras | — | ✅ | — | Extra positionals |
| `def f(*, a, b)` | — | required | — | — | Keyword-only |
| `def f(*, a=1, b)` | — | required + defaulted | — | — | Any order |
| `def f(a, b, *, c, d)` | required | required | — | — | Modern API style |
| `def f(a, b=2, *, c, d=4)` | required + defaulted | required + defaulted | — | — | Flexible |
| `def f(a, b=2, *args, c=3, d)` | required + defaulted + extras | required + defaulted | ✅ | — | Full positional |
| `def f(a, b, *, c, d=4, **kw)` | required | required + defaulted | — | ✅ | Keywords catch-all |
| `def f(a, b, *args, c, d=4, **kw)` | everything | everything | ✅ | ✅ | Maximal |
| `def f(*args, **kwargs)` | — | — | ✅ | ✅ | Forwarding signature |

The rule, one more time:

```text
    def f(req_pos, opt_pos=…, *args, kw_only, kw_only=…, **kwargs):
           └── positional ──┘     └──── keyword-only ────┘

    Ordering is fixed.
    Positional: required before defaulted.
    Keyword-only: any order.
    *args and **kwargs are optional catch-alls.
    Nothing follows **kwargs.
```

Every real-world signature is one combination of these building blocks. The `print` function, decorators, and the `calc_hi_lo_avg` example all reduce to the same grammar applied to different requirements. Once you've seen the full range, designing a new function's signature becomes a matter of asking: *which arguments are required, which are optional, which must be named, and which should forward to something else?* — and mapping the answers onto the grammar.