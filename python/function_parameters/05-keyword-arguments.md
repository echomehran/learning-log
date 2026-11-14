# Keyword Arguments

## Recap: Positional Parameters as Named Arguments

Any positional parameter can be passed by name, regardless of its position in the function's parameter list:

```python
def func1(a, b, c):
    print(a, b, c)

func1(10, 20, 30)          # 10 20 30
func1(b=20, c=30, a=10)    # 10 20 30   ← order doesn't matter
func1(10, c=30, b=20)      # 10 20 30   ← positional + keyword mix
```

Using a named argument is **optional** and up to the caller. The function definition doesn't change — it defines positional parameters, and keyword passing is just a more explicit way to bind them.

### Why bother with keyword arguments?

| Reason | Example |
|---|---|
| **Clarity** | `func1(a=10, b=20, c=30)` reveals what each value means |
| **Order independence** | `func1(c=30, a=10, b=20)` — same result |
| **Selective overriding** | Skip parameters you don't care about (when they have defaults) |
| **Robustness to refactoring** | If the parameter order changes, keyword calls still work |

## Forcing Keyword Arguments: The `*` Marker

What if you want to **require** callers to use keyword arguments? For instance, because the parameters are easy to confuse when passed positionally:

```python
def draw_rect(10, 20, 100, 50)    # ← which is width vs height?
```

Python gives you a way to forbid positional passing for specific parameters — by **exhausting all positional slots** and then declaring the parameters that should be keyword-only.

### The pattern

```python
def func1(a, b, *args, d):
    print(a, b, args, d)
```

Here:
- `a`, `b` — required positional parameters.
- `*args` — captures **any** remaining positional arguments (possibly zero).
- `d` — **keyword-only** parameter (must be passed by name).

The call requires at least `a` and `b` positionally, then `*args` absorbs extra positionals, and `d` must be named.

```python
func1(10, 20, 'a', 'b', 100)
# TypeError: func1() missing 1 required keyword-only argument: 'd'
```

The `100` went into `args` (as one more positional), leaving `d` unfilled. Python doesn't "guess" that `100` was meant for `d` — once a parameter is after `*args`, it's keyword-only, period.

```python
func1(10, 20, 'a', 'b', d=100)    # ✅
# a=10, b=20, args=('a', 'b'), d=100
```

Here `d=100` is passed by name. The remaining positionals `'a'`, `'b'` fill `args`.

### Why this works

`*args` acts as a **barrier**. Everything before it is positional; everything after it is keyword-only. The `*` doesn't have to be given a name that gets used — it just has to be there to mark the boundary.

This is why the pattern is often written as:

```python
def func1(a, b, *, d):
    ...
```

The bare `*` (with no name) means "nothing more positional from here." It's the same rule as `*args`, just without collecting the extra arguments.

## Optional Positional Arguments + Required Keyword Arguments

You can mix freely. A function with optional positionals and mandatory keyword parameters:

```python
def func1(*args, d):
    print(args)
    print(d)
```

- `*args` — any number of positional arguments (including zero).
- `d` — required keyword argument.

```python
func1(1, 2, 3, d='hello')    # args=(1, 2, 3), d='hello'
func1(d='hello')             # args=(), d='hello'
```

**Zero positionals is allowed** — `*args` accepts the empty case. But `d` is still required:

```python
func1()
# TypeError: func1() missing 1 required keyword-only argument: 'd'
```

### Making `d` optional

Add a default value:

```python
def func1(*args, d='n/a'):
    print(args)
    print(d)

func1(1, 2, 3)   # args=(1, 2, 3), d='n/a'
func1()          # args=(), d='n/a'
```

Now the function is callable with any number of positionals and optionally `d`.

## Keyword-Only Parameters (No Positionals at All)

What if you want a function with **only** keyword arguments — no positional parameters at all?

You still need the `*` marker to tell Python "no positionals past this point," but you don't need `*args` to capture any. The syntax:

```python
def func1(*, d='hello'):
    print(d)
```

The bare `*` says "everything after this is keyword-only." No `*args` variable is created — extra positional arguments simply cause an error:

```python
func1(10, d='bye')
# TypeError: func1() takes 0 positional arguments but 1 was given

func1(d='bye')    # ✅ prints 'bye'
func1()           # ✅ prints 'hello'  (default)
```

### Forcing the keyword argument

Omit the default, and the keyword argument becomes required:

```python
def func1(*, a, b):
    print(a)
    print(b)

func1(a=10, b=20)    # ✅
func1(10, 20)        # TypeError: func1() takes 0 positional arguments but 2 were given
```

No way to call this function without using keyword arguments for both `a` and `b`. The `*` ensures that positionals are rejected outright.

This is the pattern used throughout the Python standard library and third-party packages for functions that would otherwise have cryptic, easy-to-confuse positional calls. It's especially valuable when the parameters are boolean flags or have similar types.

## The Ordering Rules for Keyword-Only Parameters

Here's where keyword-only parameters **differ** from positional ones:

> **For positional parameters:** required must come before defaulted. `def f(a, b=2, c): ...` is a `SyntaxError`.

> **For keyword-only parameters:** required and defaulted can be in **any order**.

```python
def func1(a, *, b='hello', c):
    print(a, b, c)

func1(5, c='bye')    # a=5, b='hello', c='bye'
```

`b` (defaulted) comes **before** `c` (required) — and it's legal. Why?

Because keyword arguments are looked up **by name**, not by position. There's no "first required, then optional" rule to enforce, since order has no meaning for named arguments. The default for `b` applies when `b` is omitted; the required `c` must always be provided, regardless of where it's declared.

### Compare

```python
# Positional: required must come first
def f(a, b=2, c): ...        # ❌ SyntaxError: non-default argument follows default argument

# Keyword-only: order doesn't matter
def f(a, *, b=2, c): ...     # ✅ legal — c is required keyword-only, b has default
```

The syntactic rule that forces "required before defaulted" applies only to **positional** parameters. Once you're past the `*`, the constraint disappears — because it was never about keyword parameters in the first place; it was about disambiguating positional binding.

## The Full Parameter Order

The complete grammar for a function signature, in order:

```text
    def f(positional_required, positional_defaulted, *args, keyword_only, **kwargs):
           └──────────── positional ────────┘     └── keyword-only ──┘
```

Putting it all together:

```python
def func1(a, b=20, *args, d=0, e='n/a'):
    print(a, b, args, d, e)
```

- `a` — required positional
- `b` — optional positional (default `20`)
- `*args` — collects remaining positionals
- `d` — keyword-only with default (`0`)
- `e` — keyword-only with default (`'n/a'`)

### Calls

```python
func1(5, 4, 3, 2, 1, d=0, e='all engines running')
# a=5, b=4, args=(3, 2, 1), d=0, e='all engines running'

func1(0, 600, d='goooood morning', e='python!')
# a=0, b=600, args=(), d='goooood morning', e='python!'

func1(11, 'm/s', 24, 'mph', d='unladen', e='swallow')
# a=11, b='m/s', args=(24, 'mph'), d='unladen', e='swallow'
```

In the third call, notice that `'m/s'` — a string — is bound to `b`, which had no type constraint (Python is dynamically typed). The values fill the positional slots in order, `args` captures the rest, and `d`/`e` are passed by keyword. This is **maximum flexibility** in one signature.

### The full grammar

```text
    ┌───────────────────────────────────────────────────────────────┐
    │  def f(                                                       │
    │       req_positional,        ← required, no default           │
    │       opt_positional=...,    ← defaulted, must come after     │
    │       *args,                 ← captures remaining positionals │
    │       kw_only_required,      ← required, must be passed by    │
    │       kw_only_default=...,   ← name — any order here          │
    │       **kwargs               ← captures extra named arguments │
    │  )                                                            │
    └───────────────────────────────────────────────────────────────┘
```

The ordering of these groups is fixed. Within each group:
- **Positional group**: required first, then defaulted.
- **Keyword-only group**: any order.
- **`**kwargs`** (if present): last — and always after `*`/`*args`.

## Why Keyword-Only Parameters Matter

Several practical benefits:

### 1. Avoiding the boolean trap

```python
def process(data, cache, verbose):
    ...

process(my_data, True, True)    # ← what do those booleans mean?
process(my_data, cache=True, verbose=True)    # ✅ clear
```

Force keyword-only to eliminate ambiguity:

```python
def process(data, *, cache=False, verbose=False):
    ...

process(my_data, True, True)    # TypeError — positionals forbidden
process(my_data, cache=True, verbose=True)    # ✅ intended call
```

### 2. Safe refactoring

If a function has many optional parameters, callers who use positional passing will break when you reorder them. Keyword-only signatures are robust to reordering — the call site binds by name.

### 3. API stability

Public APIs can add new keyword-only parameters without breaking existing callers, because optional keyword arguments are never ambiguous with positional ones. This is why keyword-only parameters are the dominant idiom in modern Python APIs.

## Summary

| Feature | Syntax | Notes |
|---|---|---|
| Positional params | `def f(a, b): ...` | Bound by position or name |
| Keyword-only params | `def f(a, *, d): ...` | Must be passed by name |
| Keyword-only (defaulted) | `def f(a, *, d=0): ...` | May be omitted |
| Extra positionals | `def f(a, *args): ...` | `args` is a tuple |
| Keyword-only after `*args` | `def f(a, *args, d): ...` | Same rule — anything after `*` is keyword-only |
| Full signature | `def f(a, b=2, *args, d=0, **kw): ...` | All groups in fixed order |

### The rules, distilled

```text
Before *:
    - positional parameters
    - required must come before defaulted (SyntaxError otherwise)

After * (or after *args):
    - keyword-only parameters
    - required and defaulted can be in ANY order
    - must be passed by name at the call site

Positions:
    - bare *  → no positionals accepted beyond this point (no name)
    - *args   → extra positionals captured in a tuple

Call site:
    - positionals fill left-to-right from the start of the signature
    - keywords fill by name
    - positional cannot follow keyword (SyntaxError)
```

Keyword arguments turn a rigid positional interface into a flexible named one. Requiring them (via the `*` marker) lets you design APIs where the caller **cannot** accidentally pass arguments in the wrong order — the compiler enforces intent. Combined with `*args` (accept arbitrary positionals) and defaults (make parameters optional), Python's parameter system provides fine-grained control over what callers can and cannot do, without any static typing required.