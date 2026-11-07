# Variable Reassignment

## Every Rebinding Changes the Address

Note how the memory address of `a` is different each time it's rebound:

```python
a = 10
hex(id(a))
# '0x7559eaf0'

a = 15
hex(id(a))
# '0x7559eb90'

a = 5
hex(id(a))
# '0x7559ea50'

a = a + 1
hex(id(a))
# '0x7559ea70'
```

Each assignment moves `a` to a **different object** in memory. The name `a` is not a container that gets a new value written into it — it's a **label** that gets re-pointed to a different object.

```text
    a = 10      →   a ────► int(10)    (addr A)

    a = 15      →   a ────► int(15)    (addr B)   ← re-pointed; int(10) still exists

    a = 5       →   a ────► int(5)     (addr C)

    a = a + 1   →   a ────► int(6)     (addr D)   ← new object; int(5) untouched
```

The integer objects themselves are **never modified** (integers are immutable). Only the name `a` moves from one object to another.

### Note on `a = a + 1`

This is worth highlighting:

1. `a + 1` computes a **new integer object** `6`.
2. `a` is rebound to this new object.
3. The old object `5` still exists — if nothing else references it, it will be garbage collected.

The `+` never mutates `a`. It creates a new object and then assignment updates the name.

### Why the addresses look "close" for small integers

The addresses above — `0x7559eaf0`, `0x7559eb90`, `0x7559ea50` — belong to the **interned small-integer cache** (recall: CPython pre-creates integers in `[-5, 256]`). So `a = 10`, `a = 15`, `a = 5` aren't creating new integer objects — they're re-pointing `a` to **pre-existing** interned objects. The different addresses reflect *which cached object* `a` is now pointing at, not new allocations.

---

## The Twist: Two Names, One Address

Now look at what happens when two names receive the **same value**:

```python
a = 10
b = 10
print(hex(id(a)))   # 0x7559eaf0
print(hex(id(b)))   # 0x7559eaf0   ← identical!
```

Both names point to the **same object**.

```text
    a ────┐
          ├──► int(10)   (interned singleton at addr A)
    b ────┘
```

This is **not** a contradiction of the previous section. Both behaviors are consequences of the same rule:

> **Assignment binds a name to an object. It never copies the object.**

- When we rebind `a = 15`, `a` moves to a *different* object — because `15` is a different value.
- When we do `b = 10` while `a = 10`, `b` binds to the *same* object `a` already points to — because `10` is an interned singleton, and there's only one `10` to bind to.

### The unifying rule

```text
    Assignment  →  "name ⇒ object"     (a binding, not a copy)

    Which object does the name bind to?
        For immutable literals like 10, 10.0, "hello", True, None:
            the compiler / memory manager may reuse an existing object.
        For expressions like a + 1, list.append(...):
            a new object may be created.

    Either way:
        the name is re-pointed to some object —
        never a copy written into the name.
```

---

## Rebinding vs. Sharing: Same Mechanism, Different Outcomes

Two seemingly different behaviors, one mechanism:

| Code | Effect | Why |
|---|---|---|
| `a = 10` then `a = 15` | `a` points to a **new** object (addr A → addr B) | Different values, different interned objects |
| `a = 10` then `b = 10` | `a` and `b` point to the **same** object (addr A = addr A) | Same interned singleton — one object, two names |

The mechanism is identical — **`name ⇒ object`**. What differs is whether the two statements end up referencing the *same* object or *different* objects. That's not decided by the assignment itself; it's decided by whether the values are immutable singletons (shared) or arbitrary values (potentially distinct).

### Extending to non-interned cases

For values outside the interned range, the "same literal" trick stops working:

```python
a = 500
b = 500
print(hex(id(a)))   # 0x1d1e2f3a4b0
print(hex(id(b)))   # 0x1d1e2f3c1b0   ← different!
```

Same rule, different outcome: `500` is not interned, so each literal produces a **fresh object**. Two names, two objects.

```python
a = 500
b = a              # ← explicit aliasing
print(hex(id(a)))  # 0x1d1e2f3a4b0
print(hex(id(b)))  # 0x1d1e2f3a4b0   ← identical
```

Explicit aliasing (`b = a`) always produces the same object — this is guaranteed. Implicit sharing (`a = 500; b = 500`) is **not guaranteed** — it depends on the memory manager's caching decisions.

---

## The Deeper Picture

Let's consolidate everything into the two-layer model we've been building:

### Layer 1: Names and bindings

```text
    Names are references (bindings) to objects.
    Assignment: name ⟶ object.
    Rebinding: name ⟶ different object.
    Names never store values; they store references.
```

### Layer 2: Objects and their properties

```text
    Every object has:
        - identity   (address, tested by `is`)
        - type       (tested by `type` / `isinstance`)
        - value      (tested by `==`)

    Immutable objects (int, str, tuple, None) can be safely shared.
    Mutable objects (list, dict, set) are never auto-shared.
```

### How they compose

The behaviors from this and prior lessons are all direct consequences:

| Observation | Cause |
|---|---|
| `a = 10; a = 15` changes address | Rebinding moves name to a different object |
| `a = 10; b = 10` gives same address | Both bind to the interned `10` singleton |
| `a = 500; b = 500` gives different addresses | `500` is not interned — two objects created |
| `a = 500; b = a` gives same address | Explicit aliasing — same object guaranteed |
| `a = [1,2]; b = a; b.append(3)` affects both | Same object; mutation is visible through all aliases |
| `a = [1,2]; b = [1,2]` gives different addresses | Lists are mutable; never auto-shared |

Every one of these is the same underlying rule applied in a different context: **names are references; objects have identity; sharing depends on immutability**.

---

## Practical Implications

### 1. `id()` shows you which object a name points to *right now*

```python
a = 10
print(id(a))   # id of the interned 10
a = 15
print(id(a))   # id of the interned 15
```

`id()` is not the id of the *name*; it's the id of the *object the name currently references*. `a` has no fixed identity — its "identity" changes as it's rebound.

### 2. Rebinding is not mutation

```python
a = 10
a = a + 1     # rebind, not mutate

lst = [1, 2, 3]
lst.append(4)  # mutate, not rebind
```

Two operations that look similar (`a = ...` vs. `lst.method()`) but have different semantics. The first changes **where a name points**; the second changes **the object a name points to**.

### 3. "Same value" ≠ "same object"

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b   # True    (same value)
a is b   # False   (different objects)
```

Don't assume equal values mean equal objects — especially for mutable types. If you need identity, test with `is`; if you need value equality, test with `==`.

### 4. You can't "modify" an int — only rebind

```python
a = 10
# There is no way to change the object 10.
# You can only do:
a = 15        # rebind `a` to a different object
```

This is exactly why integers are safe to share: the only "operation" possible is to rebind a name, which never affects other names.

---

## Summary

```text
Rebinding:

    a = 10
    a = 15      →  a now references a different object (new address)
    a = a + 1   →  new object computed; a rebound to it

    Rebinding moves a NAME to a different OBJECT.
    Old objects remain unchanged; they may be garbage-collected.

Sharing:

    a = 10
    b = 10      →  same address (interned singleton)
    
    a = 500
    b = 500     →  different addresses (not interned)
    
    a = 500
    b = a       →  same address (explicit aliasing — guaranteed)

    Sharing means two NAMES reference the same OBJECT.

Unified rule:

    Assignment = "name ⟶ object" (never copies).
    Whether two names share an object depends on:
        - whether the value is an immutable singleton,
        - or whether the assignment explicitly aliased (`b = a`).

Mental model:

    names:       no type, no identity — just a reference
    objects:     have type, identity, and value
    rebinding:   changes which object a name references
    mutating:    changes an object (only possible for mutable types)
```

Every behavior we've seen in this course — interning, shared references, function arguments, dynamic typing — crystallizes into one rule: **names are labels; objects are the things they label; binding a name to an object never copies the object**. The reason `a = 10; a = 15` changes address, while `a = 10; b = 10` shares one, is not two different mechanisms — it's the same mechanism (binding a name to an object) applied to values that differ in whether they're cached. Once you internalize that, you stop memorizing rules and start reasoning from a single, coherent model.