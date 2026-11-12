# Positional Arguments

## The Baseline: Positional Calling

A function defined with plain parameters expects its arguments to be passed **positionally** — the first argument goes to the first parameter, the second to the second, and so on:

```python
def my_func(a, b, c):
    print("a={0}, b={1}, c={2}".format(a, b, c))

my_func(1, 2, 3)
# a=1, b=2, c=3
```

This works because Python matches arguments to parameters by **order**. `1` binds to `a`, `2` to `b`, `3` to `c`.

Positional matching is the **default** and the simplest calling convention — but it has obvious limitations once the parameter list grows or certain parameters are almost always given the same value. That's what defaults address.

---

## Default Values

You can give parameters **default values**, making them optional at call time:

```python
def my_func(a, b=2, c=3):
    print("a={0}, b={1}, c={2}".format(a, b, c))
```

Now callers can omit `b` and/or `c`:

```python
my_func(10, 20, 30)   # a=10, b=20, c=30   (all provided)
my_func(10, 20)       # a=10, b=20, c=3    (c uses default)
my_func(10)           # a=10, b=2,  c=3    (b, c use defaults)
```

If a parameter without a default is missing, the call fails at runtime:

```python
my_func()
# TypeError: my_func() missing 1 required positional argument: 'a'
```

`a` has no default, so it is **required** — no way to call the function without providing it.

### The ordering rule

> **Once a parameter has a default value, every parameter after it must also have a default value.**

So this is a **syntax error** — rejected before the program even runs:

```python
def fn(a, b=2, c):
    print(a, b, c)
# SyntaxError: non-default argument follows default argument
```

### Why the rule exists

Consider what would happen if this were legal:

```python
def fn(a, b=2, c):      # hypothetical — illegal in Python
    ...

fn(1, 3)                # what does c get? what does b get?
```

When you call `fn(1, 3)`, Python must decide: is `3` the value for `b`, or for `c`? Positional matching gives no way to skip `b` and land on `c`. The rule exists to **eliminate ambiguity at call time** — a parameter with a default is *skippable*, and once you allow skipping, all later parameters must be skippable too. Otherwise you'd never know which parameter a positional argument was meant for.

This is also why the error message is precise: *"non-default argument follows default argument"*. It's telling you that the non-default `c` cannot follow the default `b`, because `c`'s position would become unpredictable in calls that omit `b`.

### The correct ordering, always

```text
    Required parameters   →   Optional parameters
    (no default)               (with default)

    ✅  def f(a, b, c=3, d=4): ...
    ✅  def f(a, b=2, c=3): ...
    ❌  def f(a, b=2, c): ...        SyntaxError
    ❌  def f(a=1, b, c=3): ...      SyntaxError
```

Required-first is the rule, and it holds everywhere — not just for positional defaults, but also for `*args`, `**kwargs`, and keyword-only parameters (which come later in the course).

---

## Keyword Arguments

Any positional parameter can also be provided **by name**, regardless of its position in the signature:

```python
def my_func(a, b=2, c=3):
    print("a={0}, b={1}, c={2}".format(a, b, c))

my_func(c=30, b=20, a=10)    # a=10, b=20, c=30
```

The order in which you write the keyword arguments **doesn't matter** — Python matches them by name, not by order. This is the whole point of keyword arguments: they decouple the caller's argument order from the function's parameter order.

### Mixing positional and keyword arguments

You can mix them, as long as the positional arguments come first:

```python
my_func(10, c=30, b=20)      # a=10, b=20, c=30
```

Here, `10` is bound positionally to `a`, and `c`/`b` are bound by name.

### The ordering rule for keyword arguments

> **Once a keyword argument has been used, every argument after it must also be a keyword argument.**

So this is a **syntax error**:

```python
my_func(10, b=20, 30)
# SyntaxError: positional argument follows keyword argument
```

### Why this rule exists

By the time the parser sees `b=20`, it has committed to keyword-based binding. A later positional argument like `30` would be ambiguous: does it bind to the next free positional slot (`c`)? Or is it intended for something else? Rather than allow this ambiguity, Python rejects it outright. The rule is *"positional before keyword"* — in the call as well as in the definition.

### The parallel with the definition rule

Notice the symmetry:

| Context | Rule |
|---|---|
| **Definition** (parameter list) | Non-default parameters must come before defaulted ones |
| **Call** (argument list) | Positional arguments must come before keyword ones |

Both rules are about the same thing: **avoiding ambiguity when some things are specified by position and others by name.** Once you switch to "named" mode, you can't switch back within the same call (or definition).

---

## Combining Defaults and Keyword Arguments

The real power emerges when defaults and keyword arguments combine:

```python
def my_func(a, b=2, c=3):
    print("a={0}, b={1}, c={2}".format(a, b, c))

my_func(10, c=30)         # a=10, b=2,  c=30   (skip b, set c)
my_func(a=30, c=10)       # a=30, b=2,  c=10   (skip b, name a and c)
my_func(c=10, a=30)       # a=30, b=2,  c=10   (order doesn't matter)
```

Notice `my_func(10, c=30)`: `a` is filled positionally with `10`, `b` is skipped (using its default), `c` is filled by name with `30`. This is only possible because `b` has a default — otherwise skipping it would be illegal (you'd have no way to express "skip `b`, fill `c`").

This is the practical payoff of keyword arguments:

- **Clarity**: `my_func(a=30, c=10)` reads unambiguously — you can see what each value means.
- **Selective overriding**: You can override just the parameters you care about, in any order, without caring about the others.
- **Order independence**: For a function with 6 parameters, `f(x, z=10, b=20)` and `f(x, b=20, z=10)` are identical — you don't have to remember the order.

---

## The Complete Ruleset

```text
    Parameter list (definition):
        required, required, ..., defaulted, defaulted, ...
        ↑                        ↑
        must come first          can follow only after required

        def f(a, b, c=3, d=4):    ✅
        def f(a, b=2, c):         ❌ SyntaxError

    Argument list (call):
        positional, positional, ..., keyword, keyword, ...
        ↑                       ↑
        must come first          can follow only after positional

        f(1, 2, c=30):            ✅
        f(1, b=20, 30):           ❌ SyntaxError

    Binding:
        positional → binds to the next unbound parameter, in order
        keyword    → binds to the named parameter (regardless of position)
        missing    → uses default if available, else TypeError
        too many   → TypeError
        duplicate  → TypeError (same parameter bound twice)
```

### A mental model

Think of the parameter list as a **left-to-right sequence of slots**, each with a name and an optional default:

```text
    def my_func(a, b=2, c=3):
                │   │    │
                │   │    └── slot "c" (default: 3)
                │   └─────── slot "b" (default: 2)
                └─────────── slot "a" (required)

    Call: my_func(10, c=30)

        ┌────────────┐
        │ slot a ← 10│   positional: fill first unfilled slot
        │ slot b ← 2 │   default: not provided, use default
        │ slot c ← 30│   keyword: fill by name
        └────────────┘
```

Positional arguments fill **slots from left to right**. Keyword arguments fill **slots by name**. Defaults fill **slots not otherwise filled**. The two ordering rules (in the definition and in the call) exist precisely to keep this slot-filling algorithm unambiguous — you can never "return" to positional filling after switching to keywords, and you can never have a required slot after an optional one.

---

## Summary

| Concept | Rule | Why |
|---|---|---|
| **Positional** | Arguments bind in order | Simplest; matches parameter positions |
| **Defaults** | Optional parameters | Omit when the default suffices |
| **Definition order** | Required before defaulted | Otherwise positions become ambiguous |
| **Keyword args** | Named parameters can be passed in any order | Decouples argument order from call clarity |
| **Call order** | Positional before keyword | Once named, can't return to positional |
| **Combining** | `my_func(10, c=30)` skips `b` and sets `c` | Defaults make skipping legal |
| **Missing required** | `TypeError` at call time | Function cannot run without required args |

The two ordering rules — *required before defaulted* in the definition, and *positional before keyword* in the call — look like arbitrary constraints, but they're both manifestations of one underlying principle: **once you introduce a way to specify something non-positionally, you can't mix in positional specification after it, because that reintroduces the ambiguity you were trying to escape**. Understanding this makes the rules obvious rather than memorizable, and it generalizes directly to the more advanced parameter kinds (keyword-only, `*args`, `**kwargs`) that come next.