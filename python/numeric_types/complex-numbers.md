# Complex Numbers

Python has a built-in `complex` class for working with complex numbers. Complex numbers are defined in **rectangular coordinates** — a real part and an imaginary part — either via the constructor or a literal expression.

---

## Creating Complex Numbers

The complex number `1 + 2j` can be defined in two equivalent ways:

```python
a = complex(1, 2)
b = 1 + 2j

a == b   # True
```

The `j` suffix marks the imaginary part. Note that `j` (not `i`) is Python's choice — a convention inherited from electrical engineering, where `i` is reserved for current.

### Parts are floats

The real and imaginary parts are stored as **floats**, regardless of whether you passed integers:

```python
a.real, type(a.real)   # (1.0, float)
a.imag, type(a.imag)   # (2.0, float)
```

Even `complex(1, 2)` — with integer arguments — produces a complex number whose parts are `1.0` and `2.0`. There is no "integer complex" type in Python.

### The conjugate

Every complex number has a conjugate, obtained by negating the imaginary part:

```python
a = complex(1, 2)
a.conjugate()   # (1-2j)
```

Mathematically, if `z = a + bi`, then `z̄ = a - bi`. The conjugate is used in division, in computing magnitudes, and in many complex-number algorithms.

---

## Arithmetic: Polymorphism in Action

The standard arithmetic operators (`+`, `-`, `*`, `/`) are polymorphic — they're defined for complex numbers as well as for other numeric types:

```python
a = 1 + 2j
b = 3 - 4j
c = 5j
d = 10

a + b   # (4-2j)     complex + complex
b * c   # (20+15j)   complex * complex
c / d   # 0.5j       complex / int
d - a   # (9-2j)     int - complex
```

Each operator follows the standard complex arithmetic rules:

| Operation | Formula |
|---|---|
| Addition | `(a + bi) + (c + di) = (a+c) + (b+d)i` |
| Subtraction | `(a + bi) - (c + di) = (a-c) + (b-d)i` |
| Multiplication | `(a + bi)(c + di) = (ac - bd) + (ad + bc)i` |
| Division | `(a + bi)/(c + di) = ((ac + bd) + (bc - ad)i) / (c² + d²)` |

### What's *not* defined: `//` and `%`

Two arithmetic operators refuse to work with complex numbers:

```python
a // b
# TypeError: can't take floor of complex number.

a % b
# TypeError: can't mod complex numbers.
```

Why? Because floor division and modulo require an **ordering** — you need to be able to say which integer lies "below" a value. Complex numbers have **no total order**, so these operations are undefined.

This is a recurring theme: whenever an operation requires comparison or ordering, complex numbers cannot participate.

---

## Equality: Value Comparison with Float Precision

`==` and `!=` **are** supported for complex numbers. But — and this is important — the real and imaginary parts are floats, so **every floating-point caveat applies**:

```python
a = 0.1j
a + a + a == 0.3j
# False
```

Why? Because `0.1 + 0.1 + 0.1` in binary floating-point doesn't equal `0.3`. The same precision loss that affects `0.1 + 0.2 != 0.3` propagates directly to complex equality, since the parts are floats.

### The takeaway

Never compare complex numbers with `==` for "mathematical" equality. Use a tolerance-based comparison:

```python
import cmath
cmath.isclose(a + a + a, 0.3j)   # True
```

---

## Ordering: Not Defined

The comparison operators `<`, `<=`, `>`, `>=` are **not supported** for complex numbers:

```python
a = 1 + 1j
b = 100 + 100j
a < b
# TypeError: '<' not supported between instances of 'complex' and 'complex'
```

This isn't an oversight — the complex plane is **two-dimensional**. There's no natural total ordering on it that respects both addition and multiplication. Any ordering you invent would violate one or both.

Some languages (and some Python libraries) offer **magnitude-based** comparison (comparing `|a| < |b|` or `abs(a) < abs(b)`), but Python refuses to build a definition that isn't mathematically standard. If you need to order complex numbers, do it explicitly:

```python
abs(a) < abs(b)                # magnitude comparison
(a.real, a.imag) < (b.real, b.imag)   # lexicographic on (real, imag)
```

Being explicit is better than a built-in rule with surprising semantics.

---

## Math Functions: Use `cmath`, Not `math`

The standard `math` module does not work with complex numbers:

```python
import math
math.sqrt(1 + 5j)
# TypeError: can't convert complex to float
```

`math.sqrt` (and most other `math` functions) expect real numbers. For complex-number equivalents, Python provides the **`cmath` module**:

```python
import cmath

a = 1 + 5j
cmath.sqrt(a)
# (1.7462845577958914+1.4316108957382214j)
```

`cmath` mirrors `math` function-for-function: `cmath.sqrt`, `cmath.exp`, `cmath.log`, `cmath.sin`, `cmath.cos`, `cmath.atan`, and so on — each returning complex results.

### The `isclose` method

`cmath` also provides a complex-aware version of `math.isclose`:

```python
cmath.isclose(a, b, rel_tol=1e-9, abs_tol=1e-12)
```

This is the right way to compare complex numbers that should be "equal" in the mathematical sense — avoid `==`.

---

## Polar ↔ Rectangular Conversions

Complex numbers can be represented in two coordinate systems:

| System | Representation | Notation |
|---|---|---|
| **Rectangular** | `a + bi` | `(a, b)` — Cartesian coordinates |
| **Polar** | `r·(cos φ + i·sin φ)` | `(r, φ)` — magnitude and phase |

### Converting from rectangular to polar

Two built-in tools:

- **`abs(z)`** → magnitude (Euclidean norm): `√(a² + b²)`
- **`cmath.phase(z)`** → phase (argument) in radians: `atan2(b, a)`

```python
a = 1 + 1j

r = abs(a)               # √2 ≈ 1.4142135623730951
phi = cmath.phase(a)     # π/4 ≈ 0.7853981633974483

print('{0} = ({1},{2})'.format(a, r, phi))
# (1+1j) = (1.4142135623730951,0.7853981633974483)
```

### Converting from polar to rectangular

Use `cmath.rect(r, phi)`:

```python
r = math.sqrt(2)       # √2
phi = cmath.pi / 4     # π/4

cmath.rect(r, phi)
# (1.0000000000000002+1.0000000000000002j)
```

Note the tiny floating-point noise: `1.0000000000000002` instead of exactly `1.0`. This is expected — conversions between polar and rectangular coordinates amplify float imprecision.

### Why polar matters

Polar form simplifies multiplication and exponentiation dramatically:

| Operation | Rectangular | Polar |
|---|---|---|
| Multiply | `(a+bi)(c+di)` — 4 real multiplications, 2 additions | `(r₁r₂, φ₁+φ₂)` — 1 multiply, 1 add |
| Power | Repeated multiplication | `(rⁿ, nφ)` — direct |
| Root | Complex algebra | `(r^(1/n), φ/n + 2πk/n)` — n roots, trivial |

Any time you're computing powers, roots, or multiplying many complex numbers, polar coordinates are the natural choice.

---

## Euler's Identity and the Need for `isclose`

Euler's identity is one of the most famous equations in mathematics:

> **e^(iπ) + 1 = 0**

In Python:

```python
RHS = cmath.exp(cmath.pi * 1j) + 1
print(RHS)
# 1.2246467991473532e-16j
```

The result is **almost** zero — specifically `1.22 × 10⁻¹⁶ · i`. The non-zero result is due to **limited floating-point precision**, not a mathematical error. `cmath.pi` is a float approximation of π, and `cmath.exp` is computed with float precision.

### Testing with `isclose`

Because the result is *close to* zero but not exactly zero, `==` fails:

```python
RHS == 0   # False
```

Use `cmath.isclose` instead. But there's a subtlety: when comparing numbers **close to zero**, the default **relative tolerance** (`rel_tol`) doesn't help — it multiplies against the *larger* of the two operands, and one of them is `0`, so relative tolerance degenerates to zero.

You need to specify an **absolute tolerance**:

```python
cmath.isclose(RHS, 0, abs_tol=0.00001)
# True
```

Without `abs_tol`:

```python
cmath.isclose(RHS, 0)
# False
```

### The lesson about tolerance

| Comparison | When to use |
|---|---|
| **Relative tolerance** (`rel_tol`) | Large numbers whose *magnitudes* matter |
| **Absolute tolerance** (`abs_tol`) | Numbers near zero |

For `cmath.isclose(a, b)`, both tolerances are considered; the result is `True` if either one is satisfied. If neither is set, the default relative tolerance (typically `1e-09`) applies — which fails for comparisons to zero.

---

## A Complete Mental Model

```text
    complex
      │
      ├── Two float parts: .real and .imag
      ├── Literal syntax:  3 + 4j
      ├── Constructor:     complex(3, 4)
      │
      ├── Methods / properties:
      │       z.real             (float)
      │       z.imag             (float)
      │       z.conjugate()      z̄ = a − bi
      │
      ├── Arithmetic (defined):
      │       +   -   *   /      standard complex algebra
      │
      ├── Arithmetic (NOT defined):
      │       //  %              requires ordering
      │
      ├── Comparison:
      │       ==  !=             value comparison (float precision!)
      │       <   <=  >  >=      NOT supported (no total order)
      │
      ├── Math:
      │       math.*             does NOT work
      │       cmath.*            complex versions of math functions
      │       abs(z)             magnitude  |z| = √(a² + b²)
      │       cmath.phase(z)     phase      arg(z) = atan2(b, a)
      │       cmath.rect(r, φ)   polar → rectangular
      │
      └── Equality testing:
              ==  !=             not recommended (float precision)
              cmath.isclose      the right tool
                  rel_tol        for non-zero comparisons
                  abs_tol        for comparisons near zero
```

---

## Summary

| Feature | Details |
|---|---|
| **Creation** | `complex(a, b)` or `a + bj` — equivalent |
| **Parts** | `.real`, `.imag` — always floats |
| **Conjugate** | `.conjugate()` → negate imaginary part |
| **Arithmetic** | `+`, `-`, `*`, `/` — fully supported |
| **Not supported** | `//`, `%` — require ordering |
| **Equality** | `==`, `!=` — value comparison (float precision) |
| **Ordering** | `<`, `<=`, `>`, `>=` — not defined |
| **Math functions** | Use `cmath.*` — `math.*` rejects complex |
| **Rect → polar** | `abs(z)` (magnitude), `cmath.phase(z)` (phase) |
| **Polar → rect** | `cmath.rect(r, phi)` |
| **Equality testing** | `cmath.isclose` with `abs_tol` when comparing to zero |

Complex numbers are a striking illustration of Python's design philosophy: **the language refuses to define operations that aren't mathematically well-founded**, even at the cost of user convenience. Floor division and modulo are meaningless for complex numbers, so they raise `TypeError`. Ordering has no natural extension to the complex plane, so `<` and `>` are absent. The boundary between what's supported and what isn't isn't arbitrary — it's exactly the boundary of what complex arithmetic makes sense. That refusal, more than anything else, tells you the language takes its math seriously.