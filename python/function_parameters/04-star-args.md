# `*args`: Arbitrary Positional Arguments

## The Connection to Unpacking

Recall from iterable unpacking that the `*` operator captures "the rest":

```python
a, b, *c = 10, 20, 'a', 'b'

print(a, b)   # 10 20
print(c)      # ['a', 'b']   ← a LIST
```

On the assignment side, `*c` collects all remaining elements into a **list**. The same `*` symbol carries over to function definitions — but with a subtle difference in what it produces.

## Defining `*args`

In a function definition, a parameter prefixed with `*` collects all **extra positional arguments** into a **tuple**:

```python
def func1(a, b, *args):
    print(a)      # 1
    print(b)      # 2
    print(args)   # ('a', 'b')

func1(1, 2, 'a', 'b')
```

The first two positional arguments bind to `a` and `b`. Everything after them lands in `args`.

### Three things to notice

1. **`args` is a tuple, not a list.** This is the mirror-image difference from iterable unpacking on the assignment side, which produces a list. The parameter side uses a tuple because the arguments are fixed at call time — immutability is the natural choice.
2. **The name is arbitrary.** `args` is just convention. `*my_vars`, `*numbers`, `*extras` all work identically:

   ```python
   def func1(a, b, *my_vars):
       print(a)          # 10
       print(b)          # 20
       print(my_vars)    # ('a', 'b', 'c')

   func1(10, 20, 'a', 'b', 'c')
   ```
3. **`*args` can be empty.** If the call passes exactly the required arguments, `args` is an empty tuple:

   ```python
   func1(1, 2)   # args = ()
   ```

   No error — the function still receives a valid (empty) tuple.

## What Happens After `*args`?

You cannot write a normal positional parameter after `*args`, but you **can** write a parameter there — it just becomes **keyword-only**:

```python
def func1(a, b, *c, d):
    print(a, b, c, d)

func1(10, 20, 'a', 'b', 100)
# TypeError: func1() missing 1 required keyword-only argument: 'd'
```

The call fails because `d` is after `*c`, so `d` must be passed **by keyword**:

```python
func1(10, 20, 'a', 'b', d=100)   # ✅ works
```

The parameter `d` is not unreachable — it's just required to be named. This is the beginning of the **keyword-only parameters** topic (covered in the next lesson). For now, the practical rule:

> **Nothing positional can follow `*args`.** Any parameter after it is keyword-only.

The reason: `*args` absorbs every positional argument the caller provides. There's no way to distinguish "this positional goes to `*args`" from "this positional goes to the parameter after `*args`" without giving one of them a name. So Python draws a clean line: positionals stop at `*args`; anything after must be named.

## A Practical Example: `avg`

A function that averages any number of arguments:

```python
def avg(*args):
    count = len(args)
    total = sum(args)
    return total / count

avg(2, 2, 4, 4)   # 3.0
```

But watch what happens with zero arguments:

```python
avg()
# ZeroDivisionError: division by zero
```

`count` is `0`, and `total / 0` raises. The function is happy to accept zero arguments — `*args` has no minimum — but the arithmetic can't handle it.

### Fix 1: handle the empty case

```python
def avg(*args):
    count = len(args)
    total = sum(args)
    if count == 0:
        return 0
    return total / count

avg(2, 2, 4, 4)   # 3.0
avg()             # 0
```

This is a **runtime** fix — the signature still accepts zero arguments, and the function decides what to do.

### Fix 2: require at least one argument

```python
def avg(a, *args):
    count = len(args) + 1
    total = a + sum(args)
    return total / count

avg(2, 2, 4, 4)   # 3.0
avg()             # TypeError: avg() missing 1 required positional argument: 'a'
```

By splitting the signature into `a` (required) plus `*args` (the rest), the "at least one argument" contract is enforced **by the signature itself**. Callers who pass zero arguments get a clear, immediate `TypeError` — not a silent `0` return that might mask a bug upstream.

### Which is better?

**Fix 2** is generally preferred when zero arguments is genuinely an error. It shifts the check from runtime to the call boundary — the failure is reported where the mistake was made, with a precise error message, rather than propagating a questionable return value into downstream code.

Fix 1 is appropriate when "average of nothing" has a meaningful definition (e.g., `0` as the neutral element of a sum).

## Unpacking an Iterable *into* Positional Arguments

The `*` also works on the **call side**: it unpacks an iterable into separate positional arguments.

```python
def func1(a, b, c):
    print(a, b, c)

l = [10, 20, 30]
```

Passing the list directly fails:

```python
func1(l)
# TypeError: func1() missing 2 required positional arguments: 'b' and 'c'
```

The list is treated as a **single** argument, bound to `a`, leaving `b` and `c` unfilled. The function has no way to know that the list's contents were meant to be separate arguments.

Unpack it explicitly:

```python
func1(*l)   # ✅ 10 20 30
```

`func1(*l)` is equivalent to `func1(10, 20, 30)` — the `*` expands the list into three separate positional arguments.

### The two directions of `*`

This is the mirror image of `*args` in the parameter list:

| Where `*` appears | Effect |
|---|---|
| **Parameter list** — `def f(*args)` | **Collects** extra positional arguments into a tuple |
| **Call site** — `f(*iterable)` | **Expands** an iterable into separate positional arguments |

The same symbol, used at opposite ends of the function-call relationship. One is "receive many as one"; the other is "send one as many."

The bare `*l,` expression (with a trailing comma) is a **tuple display**, so it captures the intermediate result before unpacking:

```python
*l,   # (10, 20, 30)   — a tuple
```

This is a nice illustration of what `*l` in a call does: it produces a sequence of elements that get spread into the argument list.

## Combining Positional and Keyword Arguments

What happens when you try to mix keyword arguments with `*args`?

```python
def func1(a, b, c, *d):
    print(a, b, c, d)

func1(10, c=20, b=10, 'a', 'b')
# SyntaxError: positional argument follows keyword argument
```

This is a **syntax error** — rejected by the parser before the call even runs. The rule from the previous lesson (positional arguments must come before keyword arguments in a call) still applies, and `*args` doesn't change it.

Once you've used a keyword argument, you cannot return to positional arguments. The `*` expansion on the call side doesn't rescue you from this — the syntax rule is unconditional.

## The `*` in Context

Pulling everything together:

```text
    Position in code          *x means…
    ─────────────────         ─────────────
    Assignment LHS            "collect the rest into a list"
    Function definition       "collect extra positionals into a tuple"
    Function call             "unpack this iterable into separate arguments"
    List/set literal          "unpack this iterable into the new collection"
```

All four are applications of the same underlying idea: **the `*` handles the variable-length remainder**, whether you're receiving many values at once, or splitting a single iterable into many.

### Quick reference

| Syntax | Context | Result |
|---|---|---|
| `a, *b = it` | assignment | `b` is a **list** |
| `def f(*args): ...` | definition | `args` is a **tuple** |
| `f(*iterable)` | call | each element becomes a separate argument |
| `[*a, *b]` | list literal | merge into a **list** |
| `{*a, *b}` | set literal | merge into a **set** |
| `{**a, **b}` | dict literal | merge into a **dict** (later keys win) |

The asymmetry between list (assignment) and tuple (definition) isn't an arbitrary quirk — it reflects different intents. On the left-hand side, `*b` is collecting elements into a new mutable container (a list). In a parameter list, `*args` is receiving a fixed set of arguments for the duration of the call (a tuple). Each choice matches the needs of the context.

## Summary

```text
*args in a function definition:
    - Collects all extra positional arguments into a tuple.
    - The name (args) is arbitrary — use whatever reads best.
    - Can be empty: f(1, 2) with def f(a, b, *args) gives args = ().

What comes after *args:
    - Nothing positional.
    - Any parameter after *args is keyword-only.
    - f(10, 20, 'a', 'b', 100) fails if d is after *c.
    - f(10, 20, 'a', 'b', d=100) succeeds.

*args in practice:
    def avg(*args): ...       accepts any number, including zero
    def avg(a, *args): ...    requires at least one (better error)

Unpacking into positional args (call side):
    f(*lst) == f(lst[0], lst[1], lst[2])
    The * expands the iterable into separate arguments.

Mixing with keywords:
    Once you use a keyword, you cannot return to positionals.
    SyntaxError: positional argument follows keyword argument.
```

The `*` operator is one of Python's most economical features: a single character that unifies four separate use cases — capturing extra arguments, unpacking iterables into arguments, merging collections, and splitting sequences into head and tail. Once you see them all as "handle the variable-length remainder," the syntax stops being four unrelated tricks and becomes one flexible mechanism applied in four contexts.