# Reference Counting

## Why Count References?

CPython manages memory primarily through **reference counting**: every object tracks how many references point to it. When that count drops to zero, the object is deallocated. Understanding this mechanism clarifies *why* certain behaviors (like aliasing or argument passing) work the way they do.

We can inspect the reference count ourselves using `ctypes`:

```python
import ctypes

def ref_count(address):
    return ctypes.c_long.from_address(address).value
```

This reads the reference-count field directly out of the object's memory at the given address — the same field CPython itself maintains.

Let's create a variable and check:

```python
my_var = [1, 2, 3, 4]
ref_count(id(my_var))
# → 1
```

One reference: `my_var` itself. Simple.

---

## The `sys.getrefcount` Surprise

There's a built-in way to get the same information:

```python
import sys
sys.getrefcount(my_var)
# → 2
```

**Why 2 instead of 1?** We just established `my_var` is the only name pointing to the list.

The answer is subtle but essential:

> `sys.getrefcount()` takes `my_var` **as an argument** — and *passing* it means **storing** a new reference for the duration of the call.

The function receives a reference to the object, so the count momentarily increases by one. Inside the function, there are now **two** references: your `my_var`, plus the parameter binding inside `getrefcount`.

When you see 2 from `sys.getrefcount(x)` and 1 from `ref_count(id(x))`, the difference is exactly that temporary parameter binding.

### Visualizing it

```text
Before the call:
    my_var ──► [1,2,3,4]   (refcount = 1)

Inside sys.getrefcount(my_var):
    my_var    ──► [1,2,3,4]   ┐
                               ├── refcount = 2
    (param)   ──► [1,2,3,4]   ┘

After the call returns:
    my_var ──► [1,2,3,4]   (refcount = 1, parameter released)
```

This is **not** a bug — it's the honest consequence of passing an argument. `sys.getrefcount` documents this ("the count returned is generally one higher than you might expect") for exactly this reason.

That's why we'll use `ref_count()` (via `ctypes`) for measurements that need to be **exactly** accurate: it reads the count without adding a reference.

---

## Aliasing: Two Names, One Object

Create another reference to the **same** object:

```python
other_var = my_var
```

Now inspect both:

```python
print(hex(id(my_var)), hex(id(other_var)))
# 0x1e43f368388 0x1e43f368388   ← identical addresses

print(ref_count(id(my_var)))
# → 2
```

The addresses are the same — `my_var` and `other_var` reference **the same object**, not two copies. The count is 2 because **two names** point to it.

### Diagram

```text
    my_var ────┐
               ├──► [1, 2, 3, 4]   (refcount = 2)
    other_var ─┘
```

This is the essence of **aliasing**: multiple names, one object. Assignment never copies; it only binds a new name to the existing object.

---

## Removing a Reference

Now force `other_var` to point elsewhere:

```python
other_var = None
```

`other_var` is **rebound** — it now points to `None`. The list's reference count drops accordingly:

```python
print(ref_count(id(my_var)))
# → 1
```

Back to 1. The list still exists because `my_var` still references it. If we also did `my_var = None`, the count would drop to 0 — and CPython would immediately deallocate the list.

### Diagram

```text
    my_var ────┐
               ├──► [1, 2, 3, 4]   (refcount = 1)
    other_var ─┘                          │
                                          │
    (after rebinding)                     │
                                          ▼
    other_var ──► None          [1,2,3,4] now has refcount 1
                                (still alive, still reachable via my_var)
```

Crucially, **the object itself is unchanged** — same address, same contents. Only the **binding** of `other_var` moved on.

---

## The Reference-Counting Model

Reference counting follows a few simple rules:

| Action | Effect on refcount |
|---|---|
| `x = obj` | **+1** (name `x` now references `obj`) |
| `x = <other>` | **-1** for whatever `x` *was* pointing to |
| `del x` | **-1** for the object `x` referenced |
| `f(obj)` | **+1** temporarily (parameter binding) |
| `return obj` | **+1** temporarily (until the caller stores it) |
| `lst.append(obj)` | **+1** (the container now holds a reference) |
| `lst.remove(obj)` / `lst.pop()` | **-1** |
| `sys.getrefcount(obj)` | **+1** for the duration of the call |

When the count reaches **0**:

```text
refcount == 0
    ↓
object is deallocated immediately
    ↓
its memory is reclaimed
    ↓
any objects it referenced also get -1
    ↓
(which may trigger further deallocations — cascade)
```

This cascading is how entire data structures are freed: dropping the last reference to a list also drops each of its elements' counts, which may free those, and so on.

---

## Reference Cycles — the Limitation

Pure reference counting has one blind spot: **cycles**.

```python
a = []
b = [a]
a.append(b)     # a → b → a

del a
del b
# Both names gone — but each object still has refcount 1,
# because they reference each other!
```

Neither object's count reaches zero, even though no name points to them. They leak — unless the **cyclic garbage collector** (a separate mechanism in CPython) detects and breaks the cycle.

This is why CPython has **two** memory-management layers:

1. **Reference counting** — the primary mechanism, immediate and predictable.
2. **Cyclic garbage collection** — a periodic sweep to find and free cycles that reference counting can't handle.

Languages that rely purely on tracing GC (like Java or Go) don't have this duality — but they also don't free unreferenced objects immediately. CPython trades the complexity of two systems for deterministic, prompt deallocation in the common case.

---

## A Note on the `ctypes` Approach

`ctypes.c_long.from_address(address).value` reads the refcount field directly. This is:

- **Fast** — no extra reference is created.
- **Exact** — the number you see is the real count.
- **CPython-specific** — the memory layout of `PyObject` (which starts with `ob_refcnt` and `ob_type`) is an implementation detail. On PyPy, Jython, or MicroPython, this code won't work.

The address of an object is `id(obj)` — a CPython implementation detail too, but stable enough during a program's lifetime to make this kind of introspection reliable.

---

## Practical Reality

> "You will probably never need to do this in Python."

This is the most important point: **reference counting is fully transparent**. You don't manage it. You don't think about it. Python handles allocation and deallocation for you.

So why bother looking at refcounts? Three reasons:

### 1. It explains behaviors you'll encounter

- Why does `sys.getrefcount` return one more than you'd expect? (Parameter binding.)
- Why don't assignments in a function affect the caller's object? (The parameter is a **separate binding** — a new name, not a new reference to the *name*, but a new reference to the *object*.)
- Why do mutable objects change in place while immutable ones appear unchanged? (Mutation changes the *object*; rebinding changes the *name*.)

### 2. It builds intuition about identity and mutability

Every lesson so far — dynamic typing, names-as-references, function arguments, interning — rests on one model: **names point to objects, and objects track their own refcounts**. Reference counting is what makes that model *observable*.

### 3. It's foundational for what comes next

Upcoming topics — garbage collection, weak references, memory leaks, `__del__`, the `gc` module — all presuppose that you understand:

- What a reference is.
- What increasing/decreasing it means.
- Why some objects get freed immediately and others don't.

---

## Summary

```text
Reference counting:
    Every object tracks how many references point to it.
    When refcount reaches 0, the object is immediately deallocated.

Reading the count:
    ctypes.c_long.from_address(id(obj)).value   → exact count
    sys.getrefcount(obj)                        → count + 1
                                                    (parameter binding)

Aliasing:
    x = obj;  y = obj    → refcount = 2   (two names, one object)
    y = None             → refcount = 1   (y rebound; object unchanged)

What raises / lowers the count:
    +1   assignment, append, return, parameter binding
    -1   rebinding, del, pop/remove, end of function scope

Limitation:
    Cycles (a → b → a) never reach 0 by reference counting alone.
    The cyclic garbage collector (gc module) handles these separately.

Why you rarely touch this:
    Memory management is fully transparent.
    You only inspect it to understand what Python does on your behalf.
```

Reference counting is invisible in everyday Python — but it's the mechanism that makes "everything is an object, names are references" **concrete and testable**. Seeing the counts change as you rebind names, pass arguments, and remove references is the clearest demonstration that **assignment never copies and names never own their objects** — the program is a web of references, and CPython keeps precise track of every thread.