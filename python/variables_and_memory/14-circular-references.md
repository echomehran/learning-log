# Circular References in Python
## A Practical Guide to Reference Counting, Object Lifetimes, and Garbage Collection

## Table of Contents

1. [Learning Objectives](#1-learning-objectives)
2. [Understanding Object References](#2-understanding-object-references)
3. [What Is a Circular Reference?](#3-what-is-a-circular-reference)
4. [Creating Circular References](#4-creating-circular-references)
5. [Reference Counting in CPython](#5-reference-counting-in-cpython)
6. [Why Reference Counting Cannot Collect Every Object](#6-why-reference-counting-cannot-collect-every-object)
7. [The Cyclic Garbage Collector](#7-the-cyclic-garbage-collector)
8. [Experiment: Observe a Circular Reference](#8-experiment-observe-a-circular-reference)
9. [Experiment: Compare Objects With and Without Cycles](#9-experiment-compare-objects-with-and-without-cycles)
10. [Understanding `sys.getrefcount()`](#10-understanding-sysgetrefcount)
11. [Understanding `gc.collect()`](#11-understanding-gccollect)
12. [Weak References](#12-weak-references)
13. [Common Misconceptions](#13-common-misconceptions)
14. [Practical Exercises](#14-practical-exercises)
15. [Summary](#15-summary)

---

# 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain how Python variables reference objects.
- Create circular references using lists and custom classes.
- Distinguish reference counting from garbage collection.
- Explain why reference counting alone cannot reclaim an unreachable reference cycle.
- Use `sys.getrefcount()` to investigate reference counts.
- Use `gc.collect()` to request garbage collection.
- Use `weakref.ref()` to observe whether an object is still alive.
- Understand why circular references do not automatically cause memory leaks.

**Scope:** This lesson focuses on object references and memory management in CPython, the standard Python implementation used by most Python installations. Other implementations may manage memory differently.

# 2. Understanding Object References

## 2.1 Objects and variables

In Python, variables are names bound to objects. A variable does not contain the object itself in the same way a box contains a physical item; instead, it refers to an object.

Consider:

```python
a = []
b = a
```

Both names refer to the same list object.

You can verify this with the identity operator `is`:

```python
print(a is b)  # True
```

Modifying the list through either name affects the same object:

```python
a.append(10)

print(b)  # [10]
```

The important distinction is that `a` and `b` are two names referring to one object, not two independent lists.

## 2.2 References between objects

Objects can also hold references to other objects.

```python
a = []
b = [a]
```

Here:

- `a` refers to a list.
- `b` refers to another list.
- The list referenced by `b` contains a reference to the list referenced by `a`.

We can verify the relationship:

```python
print(b[0] is a)  # True
```

This ability to connect objects creates object-reference graphs. A circular reference occurs when following references eventually leads back to an object already visited.

# 3. What Is a Circular Reference?

A **circular reference** is a relationship in which an object refers to itself, either directly or indirectly through other objects.

There are two common forms.

## 3.1 Direct circular reference

An object refers to itself.

```python
items = []
items.append(items)

print(items[0] is items)  # True
print(items)              # [[...]]
```

The list contains a reference to itself.

The output `[[...]]` is Python's way of representing a recursive structure without trying to print it infinitely.

## 3.2 Indirect circular reference

Two or more objects refer to one another.

```python
a = []
b = []

a.append(b)
b.append(a)

print(a[0] is b)  # True
print(b[0] is a)  # True
```

The relationship is:

```text
a ──────> b
^         |
|         |
└─────────┘
```

This is the simplest example of a two-object reference cycle.

**Important:** A circular reference describes the structure of object references. It does not, by itself, tell you whether the objects are still reachable from your program or whether they can be garbage-collected.

# 4. Creating Circular References

Let's use custom objects to make the relationships easier to understand.

## 4.1 Define a `Node` class

```python
class Node:
    def __init__(self, name):
        self.name = name
        self.reference = None
```

Each `Node` has:

- A `name` attribute.
- A `reference` attribute that can refer to another object.

Initially, the reference is `None`.

## 4.2 Create two nodes

```python
a = Node("A")
b = Node("B")
```

At this point, each variable refers to its own node.

```text
a ──────> Node A

b ──────> Node B
```

## 4.3 Connect the nodes

```python
a.reference = b
b.reference = a
```

Now the nodes reference one another:

```text
Variable a ──> Node A ──reference──> Node B
                  ^                       |
                  |                       |
                  └────────reference──────┘
                                          ^
                                          |
                                      Variable b
```

The external variables `a` and `b` provide ways to access the objects. The objects also maintain references to one another.

Verify the cycle:

```python
print(a.reference.name)            # B
print(b.reference.name)            # A

print(a.reference.reference is a)  # True
print(b.reference.reference is b)  # True
```

The last two expressions demonstrate that following the references returns to the original object.

# 5. Reference Counting in CPython

## 5.1 What is reference counting?

**Reference counting** is a memory-management technique that tracks how many strong references point to an object.

In CPython, when an object's reference count reaches zero, the object can generally be deallocated immediately, provided no special implementation details interfere.

For example:

```python
class Node:
    pass


a = Node()
b = a

del a
```

Deleting `a` removes one reference, but `b` still refers to the object.

```python
print(b)  # The Node object still exists.
```

When you subsequently execute:

```python
del b
```

the ordinary reference from `b` is removed. If there are no other strong references, the object's reference count can reach zero.

## 5.2 A subtle but important distinction

The number of variable names is not necessarily the same as an object's reference count.

An object can be referenced by:

- Variables.
- Attributes of other objects.
- Elements of lists, tuples, and dictionaries.
- Other internal structures.
- Temporary references created while evaluating expressions.

Therefore, reference counting is about references, not simply the number of variable names you can see.

# 6. Why Reference Counting Cannot Collect Every Object

Now consider the two-node cycle again.

```python
class Node:
    def __init__(self, name):
        self.name = name
        self.reference = None


a = Node("A")
b = Node("B")

a.reference = b
b.reference = a
```

The relationships include:

```text
a ──> Node A ──> Node B <── b
       ^              |
       |              |
       └──────────────┘
```

Each node has a reference from the other node.

Now delete the external names:

```python
del a
del b
```

There are no longer any variables named `a` or `b` that allow your code to reach these objects. However, the objects still refer to each other.

Conceptually:

```text
Node A ──────> Node B
   ^              |
   |              |
   └──────────────┘
```

Each object still has an incoming reference from the other object.

Consequently, their reference counts do not reach zero merely because the external variables have been deleted.

This is the central limitation of reference counting: **a group of objects can keep one another alive even when the group is unreachable from the rest of the program.**

Reference counting alone cannot reclaim this cycle.

# 7. The Cyclic Garbage Collector

CPython supplements reference counting with a cyclic garbage collector.

Its purpose includes detecting certain unreachable reference cycles that ordinary reference counting cannot reclaim.

The module that exposes the relevant interface is `gc`.

```python
import gc

print(gc.isenabled())
```

In a normal CPython environment, automatic cyclic garbage collection is enabled by default, so this typically prints:

```text
True
```

You can also request a collection explicitly:

```python
collected = gc.collect()

print(collected)
```

The return value is the number of unreachable objects collected during that invocation, not the number of bytes of memory freed.

For ordinary unreachable cycles without complicating finalization behavior, the cyclic collector can reclaim the objects even though they reference one another.

**Key distinction:**

- Reference counting usually handles objects whose reference counts reach zero.
- Cyclic garbage collection can handle unreachable reference cycles whose objects still reference one another.

# 8. Experiment: Observe a Circular Reference

This experiment combines custom objects, circular references, weak references, and explicit garbage collection.

## 8.1 Complete program

```python
import gc
import weakref


class Node:
    def __init__(self, name):
        self.name = name
        self.reference = None


# Create two objects.
a = Node("A")
b = Node("B")

# Create a circular reference.
a.reference = b
b.reference = a

# Keep weak references to observe object lifetimes.
weak_a = weakref.ref(a)
weak_b = weakref.ref(b)

# Remove the external strong references.
del a
del b

# The objects are still alive because they reference each other.
print(weak_a() is not None)  # True
print(weak_b() is not None)  # True

# Request cyclic garbage collection.
collected = gc.collect()

print(collected)             # Usually 2
print(weak_a())              # None
print(weak_b())              # None
```

## 8.2 Step-by-step explanation

### Step 1: Create the objects

```python
a = Node("A")
b = Node("B")
```

The variables provide strong references to their respective objects.

### Step 2: Create the cycle

```python
a.reference = b
b.reference = a
```

Each object now holds a strong reference to the other.

### Step 3: Create weak references

```python
weak_a = weakref.ref(a)
weak_b = weakref.ref(b)
```

A weak reference allows you to observe an object without keeping it alive by itself.

Unlike a normal reference, a weak reference does not prevent its target from being collected.

### Step 4: Delete the external references

```python
del a
del b
```

The external strong references are removed, but the cycle remains.

### Step 5: Check the objects' lifetimes

```python
print(weak_a() is not None)
print(weak_b() is not None)
```

Before garbage collection, both expressions are expected to evaluate to `True` in this controlled experiment.

### Step 6: Request collection

```python
collected = gc.collect()
```

The cyclic garbage collector detects and collects the unreachable cycle.

For this simple example in CPython, the return value is usually `2`, corresponding to the two collected `Node` instances.

### Step 7: Verify the result

```python
print(weak_a())
print(weak_b())
```

Both expressions return `None` after their targets have been collected.

**Experimental note:** The precise return value of `gc.collect()` can vary with the Python runtime, its version, and other objects awaiting collection. The essential observation is that the weak references no longer resolve to the collected nodes.

# 9. Experiment: Compare Objects With and Without Cycles

Let's compare a normal object with an object that belongs to a circular reference.

## 9.1 An object without a cycle

```python
import weakref


class Node:
    pass


obj = Node()
reference = weakref.ref(obj)

print(reference() is not None)  # True

del obj

print(reference())              # None
```

In CPython, after `obj` is deleted, the object has no remaining strong references in this example. Its reference count reaches zero, so it is normally deallocated immediately.

The weak reference then returns `None`.

No cyclic garbage collection is needed.

## 9.2 Objects with a cycle

```python
import gc
import weakref


class Node:
    def __init__(self):
        self.reference = None


a = Node()
b = Node()

a.reference = b
b.reference = a

weak_a = weakref.ref(a)
weak_b = weakref.ref(b)

del a
del b

print(weak_a() is not None)  # True
print(weak_b() is not None)  # True

gc.collect()

print(weak_a())              # None
print(weak_b())              # None
```

The difference is that the second example contains an unreachable cycle.

| Property | No cycle | Unreachable cycle |
|---|---|---|
| External strong reference removed | Yes | Yes |
| Internal strong references remain | No | Yes |
| Reference count can reach zero immediately | Normally yes | Not from removing external references alone |
| Cyclic garbage collection needed in this experiment | No | Yes |
| Weak reference eventually returns `None` | Yes | Yes |

# 10. Understanding `sys.getrefcount()`

The `sys` module provides `getrefcount()`, which lets you inspect an object's reference count in CPython.

```python
import sys


class Node:
    pass


obj = Node()

print(sys.getrefcount(obj))  # Usually 2
```

Why is the result usually `2`?

1. The variable `obj` refers to the object.
2. Calling `sys.getrefcount(obj)` temporarily creates another reference to the object for the function call.

The temporary reference explains the extra count in this simple example.

Now create another name:

```python
another_name = obj

print(sys.getrefcount(obj))  # Usually 3
```

There are now two variable references and the temporary reference created by the call.

Remove one name:

```python
del another_name

print(sys.getrefcount(obj))  # Usually 2
```

## 10.1 Inspect a circular reference

```python
import sys


class Node:
    def __init__(self):
        self.reference = None


a = Node()
b = Node()

a.reference = b
b.reference = a

print(sys.getrefcount(a))  # Usually 3
print(sys.getrefcount(b))  # Usually 3
```

In this simple setup, each node has:

- One reference from its variable.
- One reference from the other node.
- One temporary reference from the `getrefcount()` call.

The result is usually `3` for each node.

After deleting `a` and `b`, you cannot use those names to inspect the objects anymore. That is why the weak-reference experiment is useful.

**Caution:** `sys.getrefcount()` is an implementation-specific diagnostic tool, not a general-purpose memory profiler. Exact counts can be affected by temporary references, interpreter implementation details, and the way expressions are evaluated.

# 11. Understanding `gc.collect()`

The function `gc.collect()` requests garbage collection.

```python
import gc

collected = gc.collect()

print(collected)
```

It is useful when experimenting with object lifetimes because it allows you to request collection at a specific point.

However, it is not necessary to call it after every `del` statement.

For example:

```python
import gc


class Node:
    def __init__(self):
        self.reference = None


a = Node()
b = Node()

a.reference = b
b.reference = a

del a
del b

# Request collection explicitly.
gc.collect()
```

The objects in this cycle are eligible for collection because no external strong references remain.

## 11.1 What does the return value mean?

```python
collected = gc.collect()
```

The return value is a count of unreachable objects collected by that call. It is not:

- The number of bytes freed.
- The total number of objects in memory.
- The number of references removed.
- A guarantee that all memory used by the program will be returned to the operating system.

Some memory may remain in the Python allocator for reuse even after objects have been deallocated.

# 12. Weak References

A weak reference is a reference that does not keep its target object alive.

The `weakref` module provides this functionality for supported object types.

```python
import weakref


class Node:
    pass


obj = Node()
reference = weakref.ref(obj)

print(reference() is obj)  # True
```

The expression `reference()` returns the target object while it is alive.

After the object is collected:

```python
del obj

print(reference())  # None
```

## 12.1 Why weak references are useful

Weak references are helpful when you want to observe an object's lifetime without extending it.

They are commonly useful in:

- Caches that should not keep objects alive unnecessarily.
- Registries that track objects without owning them.
- Debugging object lifetimes.
- Experiments involving garbage collection.

Not every Python object type supports weak references. Ordinary instances of the custom `Node` class above do support them.

## 12.2 Weak references do not break a cycle automatically

Consider:

```python
a.reference = b
b.reference = a
```

Both references are strong. Creating a separate weak reference for observation does not change the relationship between `a` and `b`.

To prevent the cycle itself, you would need to redesign one of the relationships as a weak reference, where appropriate.

# 13. Common Misconceptions

## Misconception 1: Every circular reference causes a memory leak

**Incorrect.**

An unreachable cycle can be collected by CPython's cyclic garbage collector. A cycle is not automatically a memory leak.

A leak-like problem can occur if your program unintentionally keeps objects reachable through long-lived references, such as global collections or caches.

## Misconception 2: `del` always destroys an object

**Incorrect.**

`del` removes a name or reference. It does not guarantee that the object is destroyed.

```python
a = []
b = a

del a

print(b)  # []
```

The list remains alive because `b` still refers to it.

## Misconception 3: Reference counting and garbage collection are the same thing

**Incorrect.**

Reference counting is one memory-management mechanism. Cyclic garbage collection is an additional mechanism that can identify certain unreachable reference cycles.

## Misconception 4: A weak reference keeps an object alive

**Incorrect.**

A weak reference does not keep its target alive. When the target is collected, calling the weak reference returns `None`.

## Misconception 5: Calling `gc.collect()` is always necessary

**Incorrect.**

CPython can collect unreachable cycles automatically. Explicit collection is particularly useful for controlled experiments, diagnostics, and certain specialized workloads.

## Misconception 6: A nonzero reference count always means an object is reachable from the program

**Incorrect.**

An unreachable cycle can contain objects with nonzero reference counts. Their references may come entirely from other objects in the same unreachable cycle.

This is the central reason reference counting alone cannot handle every object-lifetime situation.

# 14. Practical Exercises

Try each exercise before reading the suggested solution.

## Exercise 1: Create a self-referencing list

Create a list that contains a reference to itself.

Verify that the following expression evaluates to `True`:

```python
items[0] is items
```

### Suggested solution

```python
items = []
items.append(items)

print(items[0] is items)  # True
```

## Exercise 2: Create a cycle between two nodes

Create two `Node` instances named `"A"` and `"B"`.

Make each node reference the other, then verify that following two references returns to the original node.

### Suggested solution

```python
class Node:
    def __init__(self, name):
        self.name = name
        self.reference = None


a = Node("A")
b = Node("B")

a.reference = b
b.reference = a

print(a.reference.reference is a)  # True
print(b.reference.reference is b)  # True
```

## Exercise 3: Observe object lifetime

Create a cycle between two nodes, make weak references to both, and delete the external variables.

Before calling `gc.collect()`, check whether the weak references still resolve to objects.

Then request collection and check again.

### Suggested solution

```python
import gc
import weakref


class Node:
    def __init__(self):
        self.reference = None


a = Node()
b = Node()

a.reference = b
b.reference = a

weak_a = weakref.ref(a)
weak_b = weakref.ref(b)

del a
del b

print(weak_a() is not None)  # True
print(weak_b() is not None)  # True

gc.collect()

print(weak_a())              # None
print(weak_b())              # None
```

## Exercise 4: Remove the cycle

Modify the previous program so the nodes no longer reference each other before deleting the variables.

Observe what happens to the weak references.

### Suggested solution

```python
import weakref


class Node:
    def __init__(self):
        self.reference = None


a = Node()
b = Node()

a.reference = b
b.reference = a

weak_a = weakref.ref(a)
weak_b = weakref.ref(b)

# Break the cycle.
a.reference = None
b.reference = None

del a
del b

print(weak_a())  # None
print(weak_b())  # None
```

In CPython, both nodes are normally deallocated immediately after their last strong references disappear. The important change is that the internal cycle has been broken.

## Exercise 5: Explain the following result

Consider:

```python
a = []
b = a

del a

print(b is not None)  # True
```

Why does the object remain alive?

**Answer:** Deleting `a` removes only one reference. The variable `b` still refers to the same list, so the list remains alive.

## Exercise 6: Explain the limitation of reference counting

Consider:

```python
a = []
b = []

a.append(b)
b.append(a)

del a
del b
```

Why might the two lists remain allocated until cyclic garbage collection runs?

**Answer:** The lists still hold strong references to each other. Their reference counts therefore do not reach zero merely because the external names have been deleted. CPython's cyclic garbage collector can detect and reclaim the unreachable cycle.

# 15. Summary

The central concepts are:

1. **Object references:** Variables and objects can hold references to other objects.
2. **Circular references:** Following references eventually leads back to an object already visited.
3. **Reference counting:** CPython tracks references and can normally deallocate objects when their reference counts reach zero.
4. **The limitation of reference counting:** Objects in an unreachable cycle can keep one another's reference counts above zero.
5. **Cyclic garbage collection:** CPython can detect and collect certain unreachable cycles.
6. **Weak references:** These let you observe object lifetimes without keeping target objects alive.
7. **Explicit collection:** `gc.collect()` is useful for controlled experiments but is not normally required after every deletion.

## Final mental model

Think of an object graph as a network of nodes connected by references.

If an object has no remaining strong references, CPython can generally deallocate it through reference counting.

If several objects reference one another but nothing outside the group can reach them, their reference counts may remain nonzero. Cyclic garbage collection exists to identify and reclaim such unreachable cycles.

**The key lesson:** An object can have references pointing to it and still be unreachable from the rest of your program. Understanding that distinction is fundamental to understanding Python's memory management.
