# Object Mutability

Some of Python's built-in object types (also called *data types*) are **mutable**.

That is, the internal contents — the **state** of the object in memory — can change.

This is distinct from **rebinding**, where a name is re-pointed to a different object. Mutability is about whether the object *itself* can change.

---

## Mutating a List In Place

```python
my_list = [1, 2, 3]
print(my_list)
print(hex(id(my_list)))
# [1, 2, 3]
# 0x1cf6ab5b208
```

Now mutate it:

```python
my_list.append(4)
print(my_list)
print(hex(id(my_list)))
# [1, 2, 3, 4]
# 0x1cf6ab5b208   ← SAME address
```

The address of `my_list` **did not change**.

But the **contents** of `my_list` changed from `[1, 2, 3]` to `[1, 2, 3, 4]`.

The object is the same one — only its internal state changed. This is **mutation**.

---

## Rebinding a List (Not Mutation)

Compare with:

```python
my_list_1 = [1, 2, 3]
print(my_list_1)
print(hex(id(my_list_1)))
# [1, 2, 3]
# 0x1cf6abd55c8
```

Now use `+` instead of `.append()`:

```python
my_list_1 = my_list_1 + [4]
print(my_list_1)
print(hex(id(my_list_1)))
# [1, 2, 3, 4]
# 0x1cf6ab56888   ← DIFFERENT address
```

The address of `my_list_1` **changed**.

Why? Because `my_list_1 + [4]` does not modify `my_list_1` — it **creates a brand-new list object** whose contents are the concatenation. The name `my_list_1` is then **rebound** to this new object. The original list still exists (though it may now be garbage-collected if nothing else references it).

### The distinction in one line

```text
my_list.append(x)     → mutation       → same object, new contents
my_list = my_list + [x]  → rebinding   → new object, name re-pointed
```

Both produce a list that *looks* different. Only one mutates the original.

---

## Mutable Dictionaries

Dictionaries are also **mutable**:

```python
my_dict = dict(key1='value 1')
print(my_dict)
print(hex(id(my_dict)))
# {'key1': 'value 1'}
# 0x1cf6abdcdc8
```

Modify an existing key:

```python
my_dict['key1'] = 'modified value 1'
print(my_dict)
print(hex(id(my_dict)))
# {'key1': 'modified value 1'}
# 0x1cf6abdcdc8   ← SAME address
```

Add a new key:

```python
my_dict['key2'] = 'value 2'
print(my_dict)
print(hex(id(my_dict)))
# {'key1': 'modified value 1', 'key2': 'value 2'}
# 0x1cf6abdcdc8   ← SAME address
```

Once again: the **contents** of the dict change, but the memory address of `my_dict` does not. The object is mutated in place.

---

## The Subtle Case: Immutable Tuples Containing Mutable Elements

Now consider the **tuple**, an immutable sequence type.

A tuple is immutable, so its elements cannot be added, removed, or replaced:

```python
t = (1, 2, 3)
```

This tuple will **never** change as a whole — it contains exactly three elements (the integers 1, 2, and 3), and it will keep them for as long as the name `t` references it. You cannot write `t[0] = 99`; that raises `TypeError`.

But now consider:

```python
a = [1, 2]
b = [3, 4]
t = (a, b)
```

The tuple `t` is still immutable — it holds references to the object `a` and the object `b`. **This** will never change unless `t` is reassigned. You cannot swap `a` for a different object in slot 0, nor replace `b` in slot 1.

**However**, the objects `a` and `b` are themselves mutable:

```python
a.append(3)
b.append(5)
print(t)
# ([1, 2, 3], [3, 4, 5])
```

Observe: the contents of `a` and `b` **have changed**!

So immutability can be a bit subtler than simply thinking "this thing can never change."

### What actually happened

```text
Before:
    t ──► ( a ,  b )
            │    │
            ▼    ▼
         [1, 2] [3, 4]

After a.append(3), b.append(5):
    t ──► ( a ,  b )        ← t is unchanged
            │    │
            ▼    ▼
         [1,2,3] [3,4,5]    ← the referenced objects mutated
```

The tuple `t` **did not change** — it still contains exactly two elements, which are the references `a` and `b`. That will not change. But because the referenced elements are themselves mutable, it *appears* as though the tuple has changed.

It has not. This is a subtle but important distinction.

---

## The Mental Model

```text
Two independent axes:

    1. Is the OBJECT mutable?
        - list, dict, set     → mutable      (state can change in place)
        - str, tuple, int, ... → immutable   (state cannot change)

    2. Is the NAME being rebound?
        - name = <new object>  → rebinding   (name points elsewhere)
        - obj.method()          → mutation    (if obj is mutable)

Key fact:
    Immutability is SHALLOW by default.

    An immutable container may hold references to mutable objects.
    Mutating those inner objects changes the observable contents
    of the container — but the container itself never changes.

    The tuple's slots are frozen.
    The objects those slots reference are not.
```

### Why this matters

- **"Immutable" ≠ "safe from change."** A tuple is safe from reassignment of its slots — not from mutation of what's inside.
- **Identity vs. value.** `id()` tells you whether *the same object* is still in play. A changed `id()` always means a new object; an unchanged `id()` with changed contents means mutation.
- **`+` vs `+=` on sequences.** With lists, `x += [1]` mutates in place (like `extend`), while `x = x + [1]` creates a new list and rebinds. With tuples, both create a new tuple.

---

## Summary

| Operation | Effect | `id()` |
|---|---|---|
| `lst.append(x)` | mutates `lst` in place | unchanged |
| `lst[0] = x` | mutates `lst` in place | unchanged |
| `dct[k] = v` | mutates `dct` in place | unchanged |
| `lst = lst + [x]` | creates new list, rebinds name | **changed** |
| `t[0] = x` (tuple) | raises `TypeError` — not allowed | — |
| `inner.append(x)` inside a tuple | mutates inner list | tuple unchanged |

```text
Rules of thumb:

    - Mutation changes an object; rebinding changes a name.
    - Mutable types: list, dict, set, bytearray, custom objects (usually).
    - Immutable types: int, float, str, tuple, frozenset, bytes, None.
    - "Immutable" is shallow: a tuple can contain a list, and that list can change.
    - To reason about what happened, ask: did the id() change?
        - No  → mutation of the same object
        - Yes → rebinding to a new object
```

Once you internalize that **mutability is about the object** and **rebinding is about the name**, examples like `my_list.append(4)` vs. `my_list = my_list + [4]` — and the seemingly paradoxical "mutable tuple" — stop being surprising. They're two different mechanisms, each behaving exactly as defined.