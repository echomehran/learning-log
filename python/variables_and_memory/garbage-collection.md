# Garbage Collection

## Setup

We'll build on the same `ref_count` helper used in the reference counting lesson. Using the object's *address* (rather than a reference to the object itself) avoids adding an extra reference that would distort the count.

```python
import ctypes
import gc

def ref_count(address):
    """Return the reference count of the object at `address`."""
    return ctypes.c_long.from_address(address).value
```

We also need a way to check whether an object still exists in the garbage collector's tracked set:

```python
def object_by_id(object_id):
    """Return 'Object exists' if any GC-tracked object has this id, else 'Not found'."""
    for obj in gc.get_objects():
        if id(obj) == object_id:
            return "Object exists"
    return "Not found"
```

---

## Building a Circular Reference

Define two classes that reference each other:

```python
class A:
    def __init__(self):
        self.b = B(self)
        print('A: self: {0}, b: {1}'.format(hex(id(self)), hex(id(self.b))))


class B:
    def __init__(self, a):
        self.a = a
        print('B: self: {0}, a: {1}'.format(hex(id(self)), hex(id(self.a))))
```

When `A()` is constructed:

1. `A.__init__` creates `B(self)` — passing the `A` instance to `B`.
2. `B.__init__` stores that reference as `self.a`.
3. Back in `A`, the new `B` instance is stored as `self.b`.

The result is a **cycle**: `A` → `B` → `A`, with neither object referencing anything else that would keep it alive.

---

## Disabling the GC

To see how reference counting alone behaves, disable the cyclic garbage collector:

```python
gc.disable()
```

Reference counting still runs — every object's refcount updates on assignment — but the **cycle detector** is off. This isolates the two mechanisms from each other.

---

## Creating the Cycle

```python
my_var = A()
```

```text
B: self: 0x1fc1eae44e0, a: 0x1fc1eae4908
A: self: 0x1fc1eae4908, b: 0x1fc1eae44e0
```

The addresses reveal the cycle immediately:

```text
A instance (0x…4908) ──.b──► B instance (0x…44e0)
B instance (0x…44e0) ──.a──► A instance (0x…4908)
```

And `my_var` is a third reference — an external one — to the `A` instance:

```python
hex(id(my_var))
# '0x1fc1eae4908'
```

We can confirm the full topology explicitly:

```python
print('a:   \t{0}'.format(hex(id(my_var))))
print('a.b: \t{0}'.format(hex(id(my_var.b))))
print('b.a: \t{0}'.format(hex(id(my_var.b.a))))
```

```text
a:   	0x1fc1eae4908
a.b: 	0x1fc1eae44e0
b.a: 	0x1fc1eae4908      ← same as `a` — the cycle is closed
```

Save the ids for later inspection:

```python
a_id = id(my_var)
b_id = id(my_var.b)
```

---

## Inspecting the Reference Counts

```python
print('refcount(a) = {0}'.format(ref_count(a_id)))
print('refcount(b) = {0}'.format(ref_count(b_id)))
print('a: {0}'.format(object_by_id(a_id)))
print('b: {0}'.format(object_by_id(b_id)))
```

```text
refcount(a) = 2
refcount(b) = 1
a: Object exists
b: Object exists
```

Interpreting the counts:

| Object | Refcount | References come from |
|---|---|---|
| `A` instance | **2** | `my_var` (external) + `B.a` (cycle) |
| `B` instance | **1** | `A.b` (cycle) |

Both objects exist, tracked by the GC.

---

## Dropping the External Reference

Now remove the only outside reference to the cycle:

```python
my_var = None
```

```python
print('refcount(a) = {0}'.format(ref_count(a_id)))
print('refcount(b) = {0}'.format(ref_count(b_id)))
print('a: {0}'.format(object_by_id(a_id)))
print('b: {0}'.format(object_by_id(b_id)))
```

```text
refcount(a) = 1
refcount(b) = 1
a: Object exists
b: Object exists
```

### What changed

```text
Before:                          After:
                                 
my_var ──► A ◄──.a── B           A ◄──.a── B
           │          ▲              │          ▲
           └──.b──────┘              └──.b──────┘
           
refcount(A) = 2                  refcount(A) = 1
refcount(B) = 1                  refcount(B) = 1
```

The external reference from `my_var` disappeared. What remains is a **self-contained cycle**:

- `A`'s only refcount comes from `B.a`.
- `B`'s only refcount comes from `A.b`.

### Why reference counting alone cannot clean this up

Reference counting frees an object when its count drops to **zero**. Here, both counts are stuck at `1` — but each `1` is held *by the other object in the cycle*.

```text
To free A, its refcount must reach 0.
But A's refcount is held by B.
To free B, its refcount must reach 0.
But B's refcount is held by A.

Neither can be freed first. The cycle is stuck.
```

This is not a bug in reference counting — it's a fundamental limitation. Reference counting is a **local** mechanism: each object only knows its own count. It cannot see that an entire *group* of objects is unreachable from any external root.

Without intervention, this is a **memory leak** — the objects persist indefinitely even though nothing outside the cycle can ever reach them.

---

## Breaking the Cycle with `gc.collect()`

Manually trigger the cyclic garbage collector:

```python
gc.collect()
```

Now re-inspect:

```python
print('refcount(a) = {0}'.format(ref_count(a_id)))
print('refcount(b) = {0}'.format(ref_count(b_id)))
print('a: {0}'.format(object_by_id(a_id)))
print('b: {0}'.format(object_by_id(b_id)))
```

```text
refcount(a) = 0
refcount(b) = 0
a: Not found
b: Not found
```

Both objects are gone. The cycle detector:

1. **Traced** reachable objects starting from known roots.
2. **Identified** `A` and `B` as unreachable (nothing external points to them).
3. **Broke** the cycle by decrementing the references between them.
4. Allowed reference counting to free each object (refcounts fell to 0).

---

## The Two-Mechanism Model

Python uses **two** cooperating garbage collection strategies:

| Mechanism | Detects | Handles | When it runs |
|---|---|---|---|
| **Reference counting** | Objects whose refcount hits `0` | Acyclic garbage | Immediately, on every assignment |
| **Cyclic GC** | Groups of objects forming unreachable cycles | Cyclic garbage | Periodically (generational) or via `gc.collect()` |

Reference counting handles the **vast majority** of deallocation. The cyclic GC exists *specifically* to catch the case reference counting cannot — reference cycles among container-type objects.

### Why both?

- **Reference counting** is cheap, deterministic, and immediate. Most objects are freed the instant their last reference goes away.
- **Cyclic GC** is more expensive (requires graph traversal), so it runs less often — typically triggered by allocation thresholds, in a generational scheme (young → middle → old generations).

### Generational collection (brief)

CPython's cyclic GC is **generational**:

- **Generation 0** — newly created objects. Scanned most often.
- **Generation 1** — survivors of gen-0 collections. Scanned less often.
- **Generation 2** — long-lived objects. Scanned rarely.

The hypothesis: most objects die young, so scanning new objects frequently catches most garbage while keeping long-lived objects out of the way.

---

## Practical Implications

### 1. Cycles happen naturally — and not just via `__init__`

Any mutual reference produces a cycle. Common real-world sources:

- **Parent ↔ child**: `parent.children` and `child.parent` both store references.
- **Observer/event systems**: emitter holds listeners, listeners hold the emitter.
- **Exception tracebacks**: an exception's `__traceback__` references frames that reference the exception.
- **Closures over `self`**: a lambda or nested function capturing an object that stores the function.

### 2. Cycles cost more than acyclic garbage

Reference counts alone cannot free them, so they accumulate until a GC pass runs. If your program produces many cycles rapidly, the cyclic GC becomes a factor.

### 3. You can break cycles explicitly with `weakref`

For parent/child relationships, this is the idiomatic fix:

```python
import weakref

class Parent:
    def __init__(self):
        self.children = []

class Child:
    def __init__(self, parent):
        self.parent = weakref.ref(parent)   # doesn't add to refcount
```

A `weakref` does **not** increment the reference count of the target. When the parent is freed, the `weakref` becomes dead (`child.parent()` returns `None`). The cycle never forms in the first place.

### 4. `gc.collect()` is available, but usually not necessary

Explicit `gc.collect()` is useful for:

- **Measuring** memory in tests
- **Flushing** memory before a memory-sensitive operation
- **Debugging** leaks

In production code, the automatic collector is almost always sufficient.

### 5. Objects with `__del__` in a cycle

Before PEP 442 (Python 3.4), cycles containing objects with `__del__` methods were **uncollectable** — the GC skipped them to avoid double-finalization. Since Python 3.4, `__del__` no longer blocks cycle collection. Still, `__del__` is fragile in general; prefer `weakref.finalize` or context managers for cleanup.

---

## Summary

```text
Two mechanisms, working together:

1. Reference counting
    - Tracks refcount per object
    - Frees immediately at refcount == 0
    - Fails on cycles (counts held mutually, never reach 0)

2. Cyclic garbage collector
    - Traces reachability from roots
    - Identifies unreachable cycles
    - Breaks cycles and lets refcounting finish the job
    - Runs periodically (generational), or via gc.collect()

Key insight:
    A cycle of N objects all pointing at each other
    can have all N with refcount ≥ 1
    yet be completely unreachable.
    Only the cyclic GC can detect this.

Practical rules:
    - Most garbage is freed by refcounting immediately.
    - Cycles require the cyclic GC — don't disable it in production.
    - Use weakref to avoid cycles in parent/child patterns.
    - gc.collect() is for diagnostics or specific memory-sensitive moments.
    - Since Python 3.4, __del__ in cycles is no longer fatal, but still avoid it.
```

The essential lesson: **reference counting alone is not enough for a cyclic reference model.** Python's GC is a hybrid — the fast path handles the common case, and the slower collector handles the structurally impossible case for the fast path. Understanding which mechanism does what clarifies why cycles leak, why `gc.collect()` fixes them, and when you need to intervene.