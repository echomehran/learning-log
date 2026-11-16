# Shared State via Mutable Defaults: A Practical Walkthrough

## The Setup

You're building a tool that tracks tasks. Each task list should be independent — adding to one shouldn't affect another. You start with a helper that appends a formatted entry to a given list:

```python
def add_task(title, priority, task_list):
    entry = f"{title} (priority {priority})"
    task_list.append(entry)
    return task_list
```

Two projects, two lists:

```python
project_a = []
project_b = []

add_task("write docs", 1, project_a)
add_task("fix bug", 2, project_a)
add_task("review PR", 1, project_b)

print(project_a)   # ['write docs (priority 1)', 'fix bug (priority 2)']
print(project_b)   # ['review PR (priority 1)']
```

That works. Each list is independent.

## The "Improvement" That Breaks Things

The signature is a bit clunky — callers always have to create and pass a list. What if you make the list optional?

```python
def add_task(title, priority, task_list=[]):
    entry = f"{title} (priority {priority})"
    task_list.append(entry)
    return task_list
```

Now you can start a new project without creating a list first:

```python
project_a = add_task("write docs", 1)
add_task("fix bug", 2, project_a)
print(project_a)   # ['write docs (priority 1)', 'fix bug (priority 2)']
```

Looks fine. Now start a **second** project the same way:

```python
project_b = add_task("review PR", 1)
print(project_b)
# ['write docs (priority 1)', 'fix bug (priority 2)', 'review PR (priority 1)']
```

`project_b` contains **project_a's items** plus its own. The lists are not independent — they're the **same object**.

## Why This Happens

The default `[]` is evaluated **once**, when the `def` statement executes. It becomes a fixed attribute of the function:

```python
add_task.__defaults__
# ([...],)   ← this exact list is reused on every call
```

Every call that omits `task_list` receives that same list — the one created at definition time. Appending to it in one call is visible in the next call, because it's the same list.

```text
    def add_task(title, priority, task_list=[]):
                                            ↑
                                            │
                                            └── ONE list, created here,
                                                shared by all calls

    add_task("a", 1)  ────►  appends "a" to shared list  → ['a']
    add_task("b", 1)  ──┐    appends "b" to shared list  → ['a', 'b']
                        └─ same list!
```

The function isn't "creating a new list when called" — the list was already created, and every call gets a reference to it.

## The Fix: `None` as a Sentinel

Use `None` as the default. It's immutable, so it can't be accidentally shared. Inside the function, create a fresh list when the caller didn't supply one:

```python
def add_task(title, priority, task_list=None):
    if task_list is None:
        task_list = []
    entry = f"{title} (priority {priority})"
    task_list.append(entry)
    return task_list
```

Now each call without a `task_list` gets a **fresh** list:

```python
project_a = add_task("write docs", 1)
add_task("fix bug", 2, project_a)

project_b = add_task("review PR", 1)

print(project_a)   # ['write docs (priority 1)', 'fix bug (priority 2)']
print(project_b)   # ['review PR (priority 1)']   ← independent
```

### Why `is None` and not `not task_list`

```python
if not task_list:          # ❌ wrong here
    task_list = []
```

`not task_list` is `True` for **any** falsy value — including an empty list the caller explicitly passed:

```python
empty = []
add_task("x", 1, empty)
# The function sees `not empty` → True → creates a NEW list
# The caller's `empty` is never appended to — silently ignored!
```

`is None` only triggers when the caller **omitted** the argument. An explicitly-passed empty list is respected:

```python
if task_list is None:      # ✅ correct
    task_list = []
```

This is the same lesson from the `datetime.utcnow()` case: **use `is None` for sentinel checks**, never `not x` when `x` can be a legitimate falsy value.

---

## When Shared State Is Actually Intentional

The `task_list=[]` pattern isn't universally wrong — it's wrong **when you don't want sharing**. Sometimes you *do* want a single piece of state to persist across calls. The classic case is a **cache**.

### The problem: redundant computation

A recursive factorial with no caching:

```python
def factorial(n):
    if n < 1:
        return 1
    print(f"computing {n}!")
    return n * factorial(n - 1)
```

```python
factorial(3)
# computing 3!
# computing 2!
# computing 1!
# 6

factorial(3)
# computing 3!      ← recomputes everything
# computing 2!
# computing 1!
# 6
```

Every call redoes the full recursion. For `factorial(1000)`, that's 1000 redundant multiplications even if `factorial(999)` was already computed.

### The fix: a shared cache dict

```python
def factorial(n, cache={}):
    if n < 1:
        return 1
    if n in cache:
        return cache[n]
    print(f"computing {n}!")
    result = n * factorial(n - 1)
    cache[n] = result
    return result
```

Here, the **same** `cache` dict is used across all calls — deliberately. Each call checks the shared cache before computing:

```python
factorial(3)
# computing 3!
# computing 2!
# computing 1!
# 6

factorial(3)
# 6              ← no computation; all hits

factorial(5)
# computing 5!
# computing 4!
# 120            ← 3!, 2!, 1! served from cache
```

The caching version computes `5!` in two new steps (`5!` and `4!`) instead of five, because `3!` was already in the shared dict. That's the whole point.

### This is memoization

The technique is called **memoization**: storing computed results so future calls can return them without recomputation. The shared-mutable-default is a crude mechanism for it — the dict is created once, and all calls share it.

### But this is not the best way

The default-dict trick has a real downside: **the cache lives as long as the function**. There's no way to clear it, inspect it, or use a separate cache per call site. The modern approach is:

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def factorial(n):
    if n < 1:
        return 1
    print(f"computing {n}!")
    return n * factorial(n - 1)
```

`lru_cache` handles the memoization with a proper, inspectable cache, and provides `factorial.cache_clear()` and `factorial.cache_info()`. This is what you should reach for in real code.

### Still, the pattern teaches something

The `cache={}` pattern is worth understanding even though you won't use it in production:

1. It shows that **shared state via default is a real thing**, not always a bug.
2. It introduces the idea that a function can **carry state** between calls.
3. It sets up the more general concept of **closures** (functions that capture state) and **decorators** (functions that wrap other functions with stateful behavior).

Both `lru_cache` and the default-dict trick are crude forms of "the function remembers something." The idiomatic approach is a decorator that manages state explicitly; the default-dict version is a low-tech preview.

---

## The Decision Rule

For any default value, ask: **should all calls share this object, or should each call get a fresh one?**

| Intent | Default | Example |
|---|---|---|
| **Independent per call** | Use `None`, create inside | `task_list`, `datetime.utcnow()`, any mutable container |
| **Shared across calls** | Mutable default is OK (but prefer a decorator) | `cache={}` for memoization |
| **Immutable, no state** | Any literal is safe | `n=0`, `name=""`, `enabled=True` |

The **safe default** is: **never use a mutable object as a default**. If you need per-call state, use `None`. If you need shared state, use `functools.lru_cache` or a class — not a mutable default.

### The complete safe pattern

```python
def function(arg, optional=None):
    if optional is None:
        optional = <fresh value>
    ...
```

This is safe for lists, dicts, sets, timestamps, and any other case where the natural default is "compute this fresh each call."

---

## Summary

```text
The trap:
    Mutable defaults (e.g., []) are created once at definition time
    and shared by every call. Mutations accumulate across calls.

The symptom:
    A second "fresh" list contains data from the first call.
    A timestamp never updates.
    A cache of unrelated calls grows without bound.

The fix:
    Use None as a sentinel; create the real default inside the function.

        def f(arg, items=None):
            if items is None:
                items = []

The detection:
    Use "is None", not "not items".
    "not items" also triggers on [] and other legitimate falsy values.

When sharing is intentional:
    A cache dict is a legitimate shared default, but prefer
    functools.lru_cache or a class for clarity and control.

Rules of thumb:
    - Never use a mutable default unless you want sharing.
    - Never use a function call as a default (it evaluates once).
    - Use None sentinels for "fresh per call."
    - Use "is None" for the sentinel check.
```

The trap is subtle because it's silent: the bug doesn't raise an exception — it produces a **wrong result** that looks correct at a glance. A second list with extra items is easy to overlook until a user reports "my new project already has tasks in it." The `None` sentinel pattern eliminates the possibility entirely, and it should be your reflex whenever a parameter's natural default is a mutable value.