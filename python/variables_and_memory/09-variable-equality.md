# Equality of Variables

## Identity vs. Equality: Two Different Questions

From the previous lesson, we know that `a` and `b` may share a **reference**:

```python
a = 10
b = 10

print(hex(id(a)))   # 0x7559eaf0
print(hex(id(b)))   # 0x7559eaf0   ← same address
```

Two operators ask two fundamentally different questions about this relationship:

| Operator | Question it asks | Compares |
|---|---|---|
| `is` | *"Are these the same object?"* | **Identity** (memory address) |
| `==` | *"Do these have the same value?"* | **Equality** (contents, via `__eq__`) |

```python
print("a is b: ", a is b)   # True  — same object
print("a == b: ", a == b)   # True  — same value
```

When the same object is referenced, both answers agree. But the operators are **independent**, and they can disagree.

---

## When `is` and `==` Disagree

### Distinct objects with equal values

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(hex(id(a)))   # 0x27006f17288
print(hex(id(b)))   # 0x27006e968c8   ← different addresses
```

Two different list objects — but their contents are equal:

```python
print("a is b: ", a is b)   # False  — different objects
print("a == b: ", a == b)   # True   — same value
```

```text
    a ────► [1, 2, 3]   (addr A)
    
    b ────► [1, 2, 3]   (addr B)
    
    a is b   → False   (different objects)
    a == b   → True    (same contents)
```

This is why the memory manager **refused to auto-share** these lists: they're mutable, so sharing would be unsafe (as we saw in the previous lesson). Each literal creates a fresh object; equality is preserved, identity is not.

### Different types, equal values

Python's `==` is **cross-type friendly** — it compares values across numeric types when they're comparable:

```python
a = 10       # int
b = 10.0     # float
```

These are **not** the same object — one is `int`, the other is `float`:

```python
print(type(a))   # <class 'int'>
print(type(b))   # <class 'float'>

print(hex(id(a)))   # 0x7559eaf0
print(hex(id(b)))   # 0x270064b1870   ← different
```

```python
print('a is b:', a is b)   # False  — different objects, different types
print('a == b:', a == b)   # True   — numerically equal
```

Even `int` vs `complex` works the same way:

```python
c = 10 + 0j
print(type(c))    # <class 'complex'>

print('a is c:', a is c)   # False  — different types
print('a == c:', a == c)   # True   — numerically equal (10 == 10+0j)
```

### How can `10 == 10.0 == 10+0j`?

This isn't magic — it's the **numeric tower** in Python's data model. When comparing two numeric values of different types, Python **coerces one to a common type** (following the numeric tower: `int → float → complex`) and compares numerically. The `==` operator delegates to each type's `__eq__`, which is designed to handle cross-type numeric comparison.

The key insight: **equality is about values, not about types or identities**. Two objects can be `==` without being `is`, and even without being the same type.

### The full picture

```text
    a = 10          a ────► int(10)      addr A
    b = 10.0        b ────► float(10.0)  addr B
    c = 10+0j       c ────► complex(10)  addr C
    
    a is b   → False (different objects)
    a is c   → False
    a == b   → True  (10 == 10.0)
    a == c   → True  (10 == 10+0j)
    b == c   → True  (10.0 == 10+0j)
```

Different objects, different types, equal values.

---

## `None`: The Canonical Singleton

### What `None` actually is

**Note:** `None` is a built-in "variable" of type `NoneType`.

The keyword `None` is really a reference to a single instance of `NoneType` — the **only** instance that ever exists in a Python process.

```python
print(None)              # None
hex(id(None))            # '0x75576bc0'
type(None)               # NoneType
```

Since `NoneType` is immutable (there's nothing to mutate — it carries no state), the memory manager **shares it freely**. Every reference to `None` in your program points to the same object.

### References to `None` always share identity

```python
a = None
print(type(a))          # <class 'NoneType'>
print(hex(id(a)))       # 0x75576bc0   ← same as id(None)

b = None
print(hex(id(b)))       # 0x75576bc0   ← same again

print(a is None)        # True
print(a is b)           # True   — the same singleton
print(a == b)           # True   — trivially (same object ⇒ same value)
```

```text
    None ──────────────► NoneType instance  (addr X, one per process)
    a ─────────────────► (addr X)
    b ─────────────────► (addr X)
```

All names that reference `None` reference the **same object**, because there's only one.

### Comparing other objects to `None`

```python
l = []
type(l)          # list

l is None        # False
l == None        # False
```

- `l is None` → `False`: the list is not the `None` singleton.
- `l == None` → `False`: the list's value isn't equal to `None`'s value. (More precisely: `list.__eq__` returns `NotImplemented` when compared to `None`, and Python falls back to identity, which also fails.)

---

## Why `is None` Is Preferred over `== None`

Python style guides (PEP 8, and every linter) recommend:

```python
if x is None:       # ✅ preferred
    ...

if x == None:       # ❌ discouraged
    ...
```

### Why?

1. **`is` is faster.** It's a single pointer comparison — no `__eq__` method dispatch, no type coercion.
2. **`is` is semantically correct.** You're asking "is this *the* `None` object?" — an identity question. There's only one, so identity is exactly the right check.
3. **`==` can be overridden.** A custom class could define `__eq__` to return `True` when compared to `None` — which would be surprising and probably a bug. `is` cannot be overridden; it always tests identity.

```python
class Sneaky:
    def __eq__(self, other):
        return True   # always "equal" to anything

s = Sneaky()
s == None     # True    ← surprising!
s is None     # False   ← correct
```

`is None` is immune to this. It's the canonical way to test for `None` in Python.

### When would `x == None` be correct?

Essentially never, in well-written Python. The only situation where it matters is if you have a legitimate reason to invoke a custom `__eq__` that specifically handles `None` — which is extremely rare and usually a design smell.

---

## The Three-Layer Model

Now we can state the full model precisely:

| Layer | Question | Operator | Example |
|---|---|---|---|
| **Identity** | Same object? | `is` | `a is b` |
| **Equality** | Same value? | `==` | `a == b` |
| **Type** | Same class? | `type()` / `isinstance()` | `type(a) is int` |

These are **independent** — any combination is possible:

| `a is b` | `a == b` | `type(a) is type(b)` | Example |
|---|---|---|---|
| ✅ True | ✅ True | ✅ True | `a = b = 10` (same interned int) |
| ✅ True | ✅ True | ✅ True | `a = b = [1,2,3]` (same list) |
| ❌ False | ✅ True | ✅ True | `a = [1,2,3]; b = [1,2,3]` (two equal lists) |
| ❌ False | ✅ True | ❌ False | `a = 10; b = 10.0` (int vs. float) |
| ❌ False | ❌ False | ✅ True | `a = [1,2]; b = [3,4]` (two different lists) |
| ✅ True | ✅ True | ✅ True | `a = b = None` (the singleton) |

Note the first and last rows: shared identity **implies** equal values (for immutable and even mutable objects — if they're the same object, they trivially have the same state). Identity is a *sufficient* condition for equality, but not a *necessary* one.

### The logical structure

```text
    Identity (is)  ⟹  Equality (==)        (same object ⇒ same value)
    Equality (==)  ⇏  Identity (is)        (same value ⇏ same object)

    Type equality  is independent of both:
        - Same type doesn't imply == or is
        - == can be True across different types (numeric coercion)
```

---

## Practical Rules

1. **Use `is` for singletons:** `None`, `True`, `False`, `Ellipsis`, and your own sentinel objects.
   ```python
   if x is None: ...
   if flag is True: ...     # (or simply "if flag:")
   ```

2. **Use `==` for values:** anything else.
   ```python
   if name == "Alice": ...
   if items == [1, 2, 3]: ...
   ```

3. **Never use `is` as a shortcut for `==`** — even for strings or integers. Interning is an implementation detail, not a guarantee. `x is 256` may work today and break tomorrow (or on another interpreter).

4. **Remember `==` can be overridden** — `is` cannot. For sentinel checks, `is` is the safe choice.

5. **Numeric equality is value-based**, not type-based. `10 == 10.0 == 10+0j` — the numeric tower handles coercion.

---

## Summary

```text
Two independent questions:

    is   →  identity      (same object? same memory address?)
    ==   →  equality      (same value? uses __eq__, may coerce types)

Shared identity implies equality:
    a is b   ⇒   a == b       (always)
    a == b   ⇏   a is b       (not always)

Distinct objects, equal values:
    a = [1, 2, 3]
    b = [1, 2, 3]
    a is b   → False
    a == b   → True

Cross-type equality (numeric tower):
    a = 10
    b = 10.0
    a is b   → False   (int vs. float)
    a == b   → True    (numerically equal)

None:
    NoneType is immutable, so the singleton is shared across the process.
    a = None;  b = None
    a is b   → True    (one object)
    a is None → True   (identity check — preferred)
    
    Use "is None", not "== None":
        - faster (pointer compare)
        - correct (identity is the right question)
        - immune to __eq__ overrides

Rules of thumb:
    is    → singletons only (None, True, False, sentinels)
    ==    → value comparison, everything else
```

The distinction between `is` and `==` is the practical face of the deeper model we've been building: **names reference objects; objects have identity and value; these are different properties and require different operators to inspect**. Once you internalize that `10`, `10.0`, and `10+0j` can be `==` but never `is`, and that `None` is `is` to every reference of itself, the semantics stop being a collection of edge cases — they become the only possible outcomes of a single coherent data model.