# Shared References and Mutability

## Shared References with Immutable Objects

This code creates a **shared reference** between `my_var_1` and `my_var_2`:

```python
my_var_1 = 'hello'
my_var_2 = my_var_1
print(my_var_1)   # hello
print(my_var_2)   # hello
```

Both names point to the **same object** in memory:

```python
print(hex(id(my_var_1)))   # 0x24c9144ca08
print(hex(id(my_var_2)))   # 0x24c9144ca08   ← identical
```

Now "modify" `my_var_2`:

```python
my_var_2 = my_var_2 + ' world!'
```

```python
print(hex(id(my_var_1)))   # 0x24c9144ca08
print(hex(id(my_var_2)))   # 0x24c9144fab0   ← different
```

### What happened

Strings are **immutable**. `my_var_2 = my_var_2 + ' world!'` does **not** modify the original string. It creates a **new string object** and rebinds `my_var_2` to it. `my_var_1` still points to the original `'hello'`.

```text
Before:
    my_var_1 ────┐
                 ├──► "hello"    (one object)
    my_var_2 ────┘

After my_var_2 = my_var_2 + ' world!':
    my_var_1 ────────► "hello"           (original, unchanged)
    
    my_var_2 ────────► "hello world!"    (new object)
```

This is **safe** because you can never mutate the object that both names shared. Rebinding one name has no effect on the other.

---

## Shared References with Mutable Objects

Now the same pattern, but with a list:

```python
my_list_1 = [1, 2, 3]
my_list_2 = my_list_1
print(my_list_1)   # [1, 2, 3]
print(my_list_2)   # [1, 2, 3]
```

Same memory address — shared reference:

```python
print(hex(id(my_list_1)))   # 0x24c9144fc48
print(hex(id(my_list_2)))   # 0x24c9144fc48   ← identical
```

Now modify the list **through** `my_list_2`:

```python
my_list_2.append(4)
```

`my_list_2` has changed:

```python
print(my_list_2)   # [1, 2, 3, 4]
```

And because `my_list_1` references the **same object**, it changed too:

```python
print(my_list_1)   # [1, 2, 3, 4]   ← surprise!
```

The shared reference is still intact:

```python
print(hex(id(my_list_1)))   # 0x24c9144fc48
print(hex(id(my_list_2)))   # 0x24c9144fc48   ← still identical
```

### What happened

Lists are **mutable**. `my_list_2.append(4)` does **not** create a new object — it **mutates the existing list in place**. Both names see the change because they point to the same object.

```text
Before:
    my_list_1 ────┐
                  ├──► [1, 2, 3]   (one object)
    my_list_2 ────┘

After my_list_2.append(4):
    my_list_1 ────┐
                  ├──► [1, 2, 3, 4]   (same object, contents changed)
    my_list_2 ────┘
```

This is the **essential danger of aliasing with mutable objects**: any mutation through one name is visible through all names. There's no "copy on write" — Python is doing exactly what you told it to.

### The contrast in one sentence

| Action | Immutable (string) | Mutable (list) |
|---|---|---|
| `x = x + ...` | Rebinds → new object → safe | N/A |
| `x.append(...)` | N/A | Mutates in place → visible via all names |

The **operation** matters as much as the **type**. Rebinding is always safe; mutation is always shared.

---

## Behind the Scenes: The Python Memory Manager

Recall from earlier lessons:

```python
a = 10
b = 10
print(hex(id(a)))   # 0x7559eaf0
print(hex(id(b)))   # 0x7559eaf0   ← identical
```

Same memory address. This is **safe** because integers are **immutable**. Even though `a` and `b` initially shared the same object, you can never change `a` by "modifying" `b` — because integers can't be modified at all.

The only way to "change" `b` is to **rebind** it, which has no effect on `a`:

```python
b = 15

print(hex(id(a)))   # 0x7559eaf0
print(hex(id(b)))   # 0x7559eb90   ← different object
```

### But the memory manager refuses to do this for mutable objects

```python
my_list_1 = [1, 2, 3]
my_list_2 = [1, 2, 3]

print(hex(id(my_list_1)))   # 0x24c9146c5c8
print(hex(id(my_list_2)))   # 0x24c913c6848   ← different!
```

Two lists with **identical contents** are nevertheless **distinct objects**. Why? Because sharing them would be **unsafe**:

```text
If the memory manager did this:

    my_list_1 ────┐
                  ├──► [1, 2, 3]   (shared — hypothetically)
    my_list_2 ────┘

Then:
    my_list_2.append(4)
        ↓
    my_list_1 becomes [1, 2, 3, 4]   ← unexpected, silent bug

That would be catastrophic. So the optimizer refuses.
```

The compiler/memory manager only applies sharing optimizations when sharing is **observationally safe** — that is, when the object cannot be mutated. Immutable values can be shared freely; mutable values cannot.

### The general principle

> **Immutability is the precondition for sharing.**

This is the same rule we've seen in every prior optimization lesson:

| Optimization | Only safe when |
|---|---|
| Small-int interning | Integers are immutable |
| String interning | Strings are immutable |
| Constant folding | Operands are immutable (or result is small/immutable) |
| Membership rewriting (list → tuple) | The literal is never mutated |
| Sharing integer objects | Integer values can't change |
| **Sharing list objects** | **Never — lists are mutable** |

Every sharing optimization in Python rests on the same foundation: **if it can be mutated, it cannot be safely shared**.

---

## Why This Matters in Practice

The distinction between rebinding and mutating is one of the most common sources of subtle bugs in Python. Consider:

### Bug 1: Unintended aliasing

```python
def add_item(items, new_item):
    items.append(new_item)
    return items

original = [1, 2, 3]
modified = add_item(original, 4)
print(original)   # [1, 2, 3, 4]   ← caller's list was modified!
print(modified)   # [1, 2, 3, 4]
```

`original` and `modified` are the same object — the function mutated it. If the caller didn't expect that, it's a bug.

**Fix:** copy before mutating.

```python
def add_item(items, new_item):
    result = list(items)      # shallow copy
    result.append(new_item)
    return result
```

Now `original` is untouched.

### Bug 2: Assuming `x = x + ...` is safe everywhere

```python
a = [1, 2, 3]
b = a

b = b + [4]      # ← this CREATES A NEW LIST (concatenation)
print(a)         # [1, 2, 3]       ← a unchanged
print(b)         # [1, 2, 3, 4]    ← b is a new list

b = a
b += [4]         # ← this MUTATES IN PLACE (augmented assignment on mutable)
print(a)         # [1, 2, 3, 4]    ← a changed!
print(b)         # [1, 2, 3, 4]
```

The operations look similar, but `+` (concatenation) creates a new object while `+=` on a mutable object uses `__iadd__`, which mutates in place. **Same variable, different behavior.**

This is one of the most important subtleties to internalize:

| Operator | For a list `b` |
|---|---|
| `b = b + [4]` | Creates a new list; rebinds `b`; no effect on aliases |
| `b += [4]` | Mutates in place (via `__iadd__`); visible via all aliases |

The first is a rebind; the second is a mutation disguised as assignment.

### Bug 3: Passing to a function that mutates

Already shown above, but worth stating explicitly: **functions can mutate their arguments**, and if the caller wasn't expecting that, the argument's contents will change unexpectedly.

---

## Practical Rules of Thumb

1. **Assignment never copies.** `b = a` makes two names for one object.
2. **Immutable objects are safe to alias.** Rebinding one name never affects the other.
3. **Mutable objects are dangerous to alias.** Mutating through one name is visible through all.
4. **Understand the operation, not just the type:**
   - Rebinding (`x = ...`, `x = x + ...`) is safe.
   - Mutation (`x.append(...)`, `x[0] = ...`, `x += ...` on mutables) is shared.
5. **Copy when you don't want sharing:**
   - Shallow: `list(x)`, `x[:]`, `dict(x)`, `x.copy()`
   - Deep: `import copy; copy.deepcopy(x)`
6. **Trust the optimizer only for immutable objects.** Small ints and identifier-like strings can be shared automatically; lists, dicts, and sets are never auto-shared.

---

## Summary

```text
Aliasing (shared references):

    Immutable objects (str, int, tuple of immutables):
        a = "hello"; b = a        → same object
        b = b + " world!"         → new object; a unchanged
        ✅ Safe

    Mutable objects (list, dict, set):
        a = [1, 2, 3]; b = a      → same object
        b.append(4)               → mutates in place; a changes
        ⚠️ Dangerous if unintended

Why the memory manager auto-shares some values and not others:
    Sharing is safe ⟺ the object cannot be mutated.
    - Integers, strings, tuples → auto-shared
    - Lists, dicts, sets → NEVER auto-shared (would be unsafe)

Operations that look like assignment but aren't:
    x = x + y    → rebind (new object)
    x += y       → mutate in place for mutables (same object)
    x.append(y)  → mutate in place
    x[i] = y     → mutate in place

Rules:
    1. Assignment binds names, never copies objects.
    2. Alias immutables freely; copy mutables when in doubt.
    3. Understand the operation: rebind is safe, mutate is shared.
```

The principle connecting every prior lesson reappears here: **immutability enables sharing**, whether the sharing is automatic (interning, constant folding) or user-controlled (`b = a` on a string vs. on a list). Once you internalize that assignment is *binding*, not *copying*, and that mutating a shared mutable object is *always* visible through every alias, the behavior of strings and lists stops being surprising — it becomes the only possible outcome of a single coherent model.