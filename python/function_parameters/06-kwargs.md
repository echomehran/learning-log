# `**kwargs`: Arbitrary Keyword Arguments

## The Basic Mechanism

Where `*args` collects arbitrary **positional** arguments into a tuple, `**kwargs` collects arbitrary **keyword** arguments into a **dict**:

```python
def func(**kwargs):
    print(kwargs)

func(x=100, y=200)
# {'x': 100, 'y': 200}
```

Every keyword argument passed to the function that doesn't match a named parameter lands in `kwargs`, as a key-value pair.

- **`kwargs` is a dict**, not a list or tuple — unlike `args`, which is a tuple.
- **The name `kwargs` is arbitrary** — `**options`, `**extras`, `**config` all work identically.
- **`kwargs` can be empty** — a call with no extra keyword arguments gives `{}`.

## `*args` and `**kwargs` Together

The most general signature uses both:

```python
def func(*args, **kwargs):
    print(args)
    print(kwargs)

func(1, 2, a=100, b=200)
# (1, 2)
# {'a': 100, 'b': 200}
```

Every positional argument (beyond the named ones) is captured in `args` as a tuple; every keyword argument (beyond the named ones) is captured in `kwargs` as a dict. Together, `*args, **kwargs` gives you a function that accepts **any** combination of positional and keyword arguments.

This is the signature used throughout Python's libraries for wrappers, decorators, and pass-through functions:

```python
def log_call(fn):
    def wrapper(*args, **kwargs):
        print(f"calling {fn.__name__}")
        return fn(*args, **kwargs)
    return wrapper
```

The wrapper doesn't know what `fn` expects, so it captures everything and forwards it — preserving the original function's interface exactly.

## What You **Cannot** Do

Several syntaxes that seem natural are actually illegal:

### `def func(*, **kwargs)` — SyntaxError

```python
def func(*, **kwargs):
    print(kwargs)
# SyntaxError: named arguments must follow bare *
```

A bare `*` means "everything after this is keyword-only" — but it expects at least one named keyword-only parameter to follow. `**kwargs` alone doesn't satisfy that expectation because it's a **catch-all**, not a specific named parameter. Python's parser requires at least one explicit parameter after `*` if `*` is present without a name.

### It's unnecessary anyway

As the notebook points out, `**kwargs` **already implies** no more positional parameters — any keyword argument that isn't bound to a named parameter goes into `kwargs`, and positional arguments can never bind to `**kwargs`. So a bare `*` before `**kwargs` adds nothing:

```python
def func(**kwargs):        # ✅ this is fine
    print(kwargs)

def func(*, **kwargs):     # ❌ SyntaxError — the * is redundant AND illegal here
    print(kwargs)
```

The bare `*` is only needed when there are **explicit keyword-only parameters** after it. With just `**kwargs`, it has no purpose.

### `def func(a, b, **kwargs, c)` — SyntaxError

```python
def func(a, b, **kwargs, c):
    pass
# SyntaxError: invalid syntax
```

You **cannot** declare parameters after `**kwargs`. This makes intuitive sense: `**kwargs` is a catch-all that absorbs *every* keyword argument not otherwise named. Any parameter after it could never be bound — the parser forbids it because it would be unreachable.

The same is true for `*args` — but with a subtle difference. `*args` can be followed by **keyword-only** parameters (they'd be passed by name, not position), whereas `**kwargs` can be followed by **nothing**.

### The ordering rule, precisely

```text
    def f(positional, ..., *args, kw_only, ..., **kwargs):
                       ↑            ↑               ↑
                       |            |               └── must be last
                       |            └── after *args, keyword-only
                       └── captures extra positionals

    Everything before *args: positional (required then defaulted)
    Everything between *args and **kwargs: keyword-only
    **kwargs: always last
    Nothing after **kwargs.
```

## Mixing Named Parameters with `**kwargs`

You can combine named positional parameters with `**kwargs` — everything the named parameters don't claim goes into `kwargs`:

```python
def func(a, b, **kwargs):
    print(a)         # 1
    print(b)         # 2
    print(kwargs)    # {'x': 100, 'y': 200}

func(1, 2, x=100, y=200)
```

- `1` and `2` bind to `a` and `b` positionally.
- `x=100` and `y=200` aren't named parameters, so they go into `kwargs`.

If you accidentally pass a name that **is** a positional parameter, it binds to that parameter — not to `kwargs`:

```python
func(1, b=2, x=100)
# a=1, b=2, kwargs={'x': 100}
#              ↑ b=2 binds to the named parameter b, not to kwargs
```

This is an important subtlety: **named parameters take precedence over `**kwargs`**. A keyword argument tries to bind to a named parameter first; only if no matching name exists does it fall into `kwargs`.

## Keyword-Only Parameters + `**kwargs`

If you want both **specific** keyword-only parameters **and** a catch-all for the rest, you must first establish the keyword-only section with `*` (or `*args`), then declare the specific parameters, then the catch-all:

```python
def func(*, d, **kwargs):
    print(d)         # 1
    print(kwargs)    # {'x': 100, 'y': 200}

func(d=1, x=100, y=200)
```

Here:
- The bare `*` opens the keyword-only section.
- `d` is a required keyword-only parameter — it must be passed by name.
- `**kwargs` collects the rest.

This is the standard way to define a function that has known keyword parameters plus a catch-all for extras — useful for APIs that want to accept "any additional keyword arguments" while requiring certain ones.

### Why the `*` is necessary here

Without the `*`, you'd write:

```python
def func(d, **kwargs):    # ← d is positional-or-keyword, not keyword-only
    ...
```

Now `d` can be passed positionally (`func(1, x=100)`) or by name (`func(d=1, x=100)`). The bare `*` makes it keyword-only:

```python
def func(*, d, **kwargs):
    ...

func(1, x=100)       # TypeError: takes 0 positional arguments but 1 given
func(d=1, x=100)     # ✅
```

If you want to force `d` to be named — for clarity in the call, or to prevent accidental misordering — the `*` is required.

## The Full Signature Grammar

Bringing together everything from the parameter lessons:

```text
    ┌─────────────────────────────────────────────────────────────────┐
    │  def f(                                                         │
    │       req_pos,              ← required positional               │
    │       opt_pos=...,          ← defaulted positional              │
    │       *args,                ← extra positionals (tuple)         │
    │       kw_only_req,          ← required keyword-only             │
    │       kw_only_opt=...,      ← defaulted keyword-only            │
    │       **kwargs              ← extra keywords (dict)             │
    │  )                                                              │
    └─────────────────────────────────────────────────────────────────┘

    Ordering is fixed. Position determines binding rules:
        - before  *: positional-or-keyword
        - after   *: keyword-only
        - *args:     tuple of extra positionals
        - **kwargs:  dict of extra keywords; always last
```

### Rules

| Rule | Consequence |
|---|---|
| Positional params come before `*` | Order: required, then defaulted |
| Keyword-only params come after `*` | Any order — required or defaulted |
| `*args` captures extra positionals | Name optional (bare `*` is legal) |
| `**kwargs` captures extra keywords | Always last; nothing after it |
| Named params take precedence over `**kwargs` | A keyword matching a param binds to it |
| Bare `*` requires at least one kw-only param | `def f(*, **kwargs)` is invalid |

### Quick reference

```python
# Simplest — arbitrary keywords only
def f(**kwargs): ...
f(a=1, b=2)              # kwargs={'a':1, 'b':2}

# Arbitrary positionals and keywords
def f(*args, **kwargs): ...
f(1, 2, a=3, b=4)        # args=(1,2), kwargs={'a':3, 'b':4}

# Named parameters + catch-all keywords
def f(a, b, **kwargs): ...
f(1, 2, x=3)             # a=1, b=2, kwargs={'x':3}

# Specific keyword-only + catch-all
def f(*, d, **kwargs): ...
f(d=1, x=2)              # d=1, kwargs={'x':2}

# Invalid — nothing after **kwargs
def f(**kwargs, c): ...  # SyntaxError

# Invalid — bare * needs at least one kw-only param
def f(*, **kwargs): ...  # SyntaxError
```

## Why `**kwargs` Matters

### 1. Pass-through wrappers

Decorators and wrapper functions use `*args, **kwargs` to forward arguments without knowing the wrapped function's signature:

```python
def deprecated(fn):
    def wrapper(*args, **kwargs):
        print(f"WARNING: {fn.__name__} is deprecated")
        return fn(*args, **kwargs)
    return wrapper
```

The wrapper works with any function — 0 parameters, 10 parameters, positional-only, keyword-only — because it doesn't inspect the signature; it just forwards everything.

### 2. Configuration dicts

Functions that accept arbitrary configuration:

```python
def connect(host, port=5432, **options):
    conn = Connection(host, port)
    for key, value in options.items():
        conn.set_option(key, value)
    return conn

connect('db.example.com', timeout=30, ssl=True, pool_size=10)
```

The `**options` pattern separates "known" parameters (host, port) from "configurable extras."

### 3. Forwarding to parent constructors

Subclasses often forward `**kwargs` to a parent's `__init__`:

```python
class MyView(View):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```

This keeps the subclass signature open — it inherits whatever parameters the parent accepts, without having to list them all.

### 4. Building function calls dynamically

The call-side form `f(**mapping)` unpacks a dict into keyword arguments:

```python
options = {'host': 'localhost', 'port': 8000}
connect(**options)    # equivalent to: connect(host='localhost', port=8000)
```

This is how frameworks build calls from configuration dicts, request parameters, or other data sources.

## Call-Side `**` Unpacking

Mirroring the `*` at the call site, `**` unpacks a mapping into keyword arguments:

```python
def func(a, b, **kwargs):
    print(a, b, kwargs)

mapping = {'a': 10, 'b': 20, 'x': 100, 'y': 200}
func(**mapping)
# a=10, b=20, kwargs={'x': 100, 'y': 200}
```

Each key becomes a keyword argument name; each value becomes the argument value. Keys must be strings.

This is the counterpart to `f(*lst)` for positional arguments. Together:

```python
def f(*args, **kwargs): ...

positional = [1, 2, 3]
keywords = {'a': 10, 'b': 20}

f(*positional, **keywords)
# args=(1, 2, 3), kwargs={'a': 10, 'b': 20}
```

The two unpacking forms combine to build arbitrary calls from iterables and mappings — the foundation of Python's generic forwarding machinery.

## Summary

| Feature | Syntax | Result type | Notes |
|---|---|---|---|
| Keyword catch-all | `def f(**kwargs): ...` | `dict` | Captures all unnamed keyword arguments |
| Both catch-alls | `def f(*args, **kwargs): ...` | tuple + dict | Fully general signature |
| Named params + catch-all | `def f(a, b, **kwargs): ...` | dict | Named params take precedence |
| Keyword-only + catch-all | `def f(*, d, **kwargs): ...` | dict | Requires `*` to open keyword-only section |
| Call-side unpacking | `f(**mapping)` | — | Unpacks dict into keyword args |

The rule for **`**kwargs`**:

- It's always **last** — nothing can follow it.
- It can never capture positional arguments.
- Named parameters (before it) take precedence over it for keyword binding.
- Bare `*` before it is illegal — `**kwargs` alone already means "no more positionals."

`**kwargs` completes Python's parameter system. `*args` handles variable positional; `**kwargs` handles variable keyword; both together handle variable everything. This flexibility is what makes decorators, wrappers, and configuration-driven APIs possible without static typing — the function can accept whatever the caller provides, and forward it appropriately, without any compile-time knowledge of the interface.