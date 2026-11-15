# Building a Generic Function Timer

The goal: a function that times **any** function, accepts **any** combination of positional and keyword arguments for it, repeats the call a specified number of times, and returns the **average** duration per call. This is a compact but complete exercise in all the parameter features covered so far: `*args`, `**kwargs`, keyword-only parameters with defaults, and forwarding.

Rather than an inline `time_it`, we'll build up a version that:

1. Takes the function to call.
2. Forwards arbitrary positional and keyword arguments to it.
3. Has a **keyword-only** `rep` parameter (must be named — never confused with a positional argument).
4. Returns the mean elapsed time per call.

---

## Step 1: The Skeleton

The signature:

```python
import time

def time_it(fn, *args, rep=5, **kwargs):
    print(args, rep, kwargs)
```

This is the key design decision. The parameter layout:

| Parameter | Kind | Purpose |
|---|---|---|
| `fn` | required positional | the function to time |
| `*args` | catch-all positionals | forwarded to `fn` as positional |
| `rep` | keyword-only (default `5`) | how many times to run `fn` |
| `**kwargs` | catch-all keywords | forwarded to `fn` as keyword |

### Why `rep` is keyword-only

If `rep` were positional — `def time_it(fn, *args, rep=5)` with `rep` after `*args` — it would **already** be keyword-only because of where it sits in the signature. But the *intent* matters: `rep` should never be mistaken for one of the arguments to `fn`. Making it keyword-only guarantees that:

```python
time_it(print, 1, 2, 3, sep='-')             # rep stays 5
time_it(print, 1, 2, 3, sep='-', rep=3)      # rep=3, sep forwarded
```

If `rep` were a positional parameter (e.g., `def time_it(fn, *args, rep=5)` is fine — but `def time_it(fn, rep=5, *args)` would be wrong), then `time_it(print, 3, 1, 2, 3)` would interpret `3` as `rep`. The keyword-only placement prevents that.

### Test the skeleton

```python
time_it(print, 1, 2, 3, sep='-')
# args=(1, 2, 3), rep=5, kwargs={'sep': '-'}
```

The print is just a diagnostic to confirm the parameter binding. No function has been called yet.

---

## Step 2: Forward the Arguments

Replace the diagnostic print with the actual call:

```python
def time_it(fn, *args, rep=5, **kwargs):
    for _ in range(rep):
        fn(*args, **kwargs)
```

The `*args, **kwargs` at the call site **re-expands** the captured arguments and forwards them unchanged:

```python
time_it(print, 1, 2, 3, sep='-')
# 1-2-3
# 1-2-3
# 1-2-3
# 1-2-3
# 1-2-3
```

The `print` function receives `1, 2, 3` positionally and `sep='-'` by keyword. The `rep` parameter is **not** forwarded — it was consumed by `time_it` and has no bearing on `print`.

### Adding more arguments

Nothing needs to change in `time_it`. Extra keyword arguments for the target function pass straight through:

```python
time_it(print, 1, 2, 3, sep='-', end=' *** ', rep=3)
# 1-2-3 *** 1-2-3 *** 1-2-3 ***
```

`end` is captured by `**kwargs` and forwarded to `print`; `rep` is captured by the named parameter and consumed by `time_it`. The two are handled independently.

### Why the forwarding works

This is the whole point of `*args, **kwargs`:

- On the **definition** side: they collect everything not otherwise bound.
- On the **call** side: they expand back into separate arguments.

The same names (`args`, `kwargs`) serve both purposes — the collection at the parameter list, the expansion at the call. `fn(*args, **kwargs)` reconstructs exactly the call the original caller intended for `fn`.

---

## Step 3: Add the Timing

Wrap the loop in `time.perf_counter()` calls, and divide by `rep` to get the average:

```python
def time_it(fn, *args, rep=5, **kwargs):
    start = time.perf_counter()
    for _ in range(rep):
        fn(*args, **kwargs)
    end = time.perf_counter()
    return (end - start) / rep
```

### Why `perf_counter`

| Clock | Best for |
|---|---|
| `time.time()` | Wall-clock time, but susceptible to system clock changes |
| `time.monotonic()` | Elapsed time that can't go backward — good for measuring |
| **`time.perf_counter()`** | **Highest resolution**; ideal for microbenchmarks |
| `time.process_time()` | CPU time only (excludes sleep, I/O) |

`perf_counter()` has the highest resolution available on the system and is the correct choice for short-duration benchmarks. `time.time()` would be coarse enough that millisecond-scale operations would show as zero.

### Why average, not total

Dividing by `rep` gives the mean per call, which is more interpretable than a total. But it assumes the calls are comparable in cost — for warm-up-sensitive functions (JIT compilation, disk cache cold start), the first call may be much slower than the rest, inflating the average. For a crude timer, that's acceptable; for a serious benchmark, you'd discard the first run or use `timeit.repeat` and take the minimum.

### Why no print

`time_it` returns the average instead of printing it. This makes the function **composable** — callers can capture the result, compare timings, sort by speed, or pass the value into further computation:

```python
t = time_it(compute_powers_2, 2, end=20000, rep=4)
print(f"compute_powers_2: {t:.4f} s per call")
```

---

## Step 4: Three Functions to Compare

Rather than timing the same trivial function repeatedly, let's pick a computation that has genuinely different implementations with meaningfully different performance characteristics. We'll compute **Fibonacci numbers** three ways.

### Iterative

```python
def fib_iter(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a
```

Runs in O(n) time, O(1) space. No recursion.

### Recursive (naive)

```python
def fib_rec(n):
    if n < 2:
        return n
    return fib_rec(n - 1) + fib_rec(n - 2)
```

Runs in O(2ⁿ) time — exponential, because the same subproblems are recomputed. This is intentionally slow; it's the classic example of why naive recursion is dangerous.

### Memoized recursive

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib_memo(n):
    if n < 2:
        return n
    return fib_memo(n - 1) + fib_memo(n - 2)
```

Same recursion structure as `fib_rec`, but `lru_cache` stores previously computed results. Each subproblem is solved once, giving O(n) time — the *right* way to make recursion efficient.

### Correctness check

Before timing, confirm all three return the same value:

```python
assert fib_iter(20) == fib_rec(20) == fib_memo(20) == 6765
```

This is important: **always verify correctness before benchmarking**. A fast but wrong function is not a useful data point.

### Adjusting the inputs for fair comparison

The recursion depth of `fib_rec` grows exponentially, so we can't call it with the same input as the other two. That's fine — `time_it` takes the input as an argument:

```python
time_it(fib_iter, 1000, rep=1000)      # iterative, large n
time_it(fib_rec, 25, rep=10)           # naive recursive, small n
time_it(fib_memo, 1000, rep=1000)      # memoized, large n
```

Each function is timed on an input that runs in a reasonable time. The comparison isn't apples-to-apples on the *same n*, but it *is* informative about each implementation's practical performance on the kind of input you'd actually give it.

---

## Results

```python
>>> time_it(fib_iter, 1000, rep=1000)
2.11e-05        # ~21 µs per call

>>> time_it(fib_rec, 25, rep=10)
4.32e-04        # ~430 µs per call — and that's for n=25, not n=1000

>>> time_it(fib_memo, 1000, rep=1000)
1.05e-07        # ~0.1 µs per call — cached after first run
```

### Reading the numbers

- **`fib_iter`** — 21 µs for n=1000. Linear in n, consistent per call.
- **`fib_rec`** — 430 µs for n=25. To do n=30 it would take ~30× longer; to do n=1000 would take longer than the age of the universe.
- **`fib_memo`** — 0.1 µs for n=1000. But this is *after the cache is warm* — the first call builds the cache, subsequent calls hit it. The measured "average" depends entirely on how many times `rep` runs after the cache is populated.

### A crucial caveat about `fib_memo`

The 0.1 µs number is misleading on its own. The first call in the `rep=1000` loop does the real work; the other 999 hit the cache. `time_it` averages over all 1000 calls, so the average is dominated by the cached calls. If you ran `time_it(fib_memo, 1000, rep=1)` you'd see a much larger number — the actual cost of computing `fib(1000)` once.

This is one of the **falsest kinds of benchmark**: a memoized function's timing depends entirely on whether the cache is warm, which depends on whether it was called before. A fair memoization benchmark would measure "cold" (first call) and "warm" (subsequent calls) separately.

---

## What This Exercise Actually Teaches

The timer is small, but it exercises every parameter feature from the previous lessons:

```text
    def time_it(fn, *args, rep=5, **kwargs):
               │   │        │       │
               │   │        │       └── catch-all keywords → forwarded to fn
               │   │        └────────── keyword-only, default 5
               │   └─────────────────── catch-all positionals → forwarded to fn
               └─────────────────────── required positional: the function
```

- **`fn`** — required positional, always provided.
- **`*args`** — collects anything positional the caller passes after `fn`.
- **`rep`** — keyword-only with a default. Must be named so it can never be confused with an argument to `fn`.
- **`**kwargs`** — collects anything keyword the caller passes that isn't `rep`.

And the call site inside:

```python
    fn(*args, **kwargs)
```

re-expands both, forwarding them to `fn` exactly as the caller intended. This is **argument pass-through** — the mechanism underlying decorators, wrapper functions, and generic APIs.

### Real-world use of this pattern

The same structure appears in:

- **`functools.wraps`** — decorators that preserve the wrapped function's signature via `*args, **kwargs`.
- **Middleware** — HTTP middleware that receives requests, forwards to handlers, and measures timings.
- **Test frameworks** — `pytest.mark.parametrize`, timeouts, retries.
- **Instrumentation libraries** — OpenTelemetry, Sentry, and similar, wrapping arbitrary callables.

Once you've written `time_it` from scratch, you understand **why** every logging/metrics/tracing library uses `*args, **kwargs` — it's the only way to intercept and forward a call without knowing the target function's signature.

### The measurement caveat, restated

`time_it` is a **crude** timer. It has real limitations:

1. **Single measurement** — no statistical repetition across separate runs; `rep` is within one run, not across many runs.
2. **First-call skew** — caches, JIT, and OS effects make the first call unrepresentative; the average is sensitive to warm-up.
3. **No isolation from other processes** — wall-clock timing includes any interference from the OS or other programs.
4. **Overhead not measured** — the loop overhead, the function-call overhead, and `perf_counter` itself are all included in `(end - start) / rep`; for extremely fast operations, this overhead may dominate.

For production-quality benchmarking, use the **`timeit`** module, which handles warm-up, repetition, and statistical reporting. `time_it` is a teaching example, not a replacement for `timeit`.

---

## Summary

```text
    Signature:
        def time_it(fn, *args, rep=5, **kwargs):
            ...
            fn(*args, **kwargs)
            ...
            return average_duration

    Parameter roles:
        fn        — required positional
        *args     — forwarded positionals
        rep       — keyword-only, default 5
        **kwargs  — forwarded keywords

    Forwarding:
        fn(*args, **kwargs)   re-expands the captured args
                              into a call that matches what the
                              original caller intended for fn

    Why this pattern matters:
        - decorators: preserving wrapped signatures
        - middleware: intercepting and forwarding
        - instrumentation: measuring arbitrary callables
        - any generic wrapper that doesn't know fn's signature
```

The timer is a microcosm of the entire parameter story: required and optional positionals, keyword-only parameters to prevent ambiguity, catch-alls to accept anything the target might need, and forwarding to reconstruct the call. Every non-trivial Python framework uses this pattern somewhere — mastering it once means recognizing it everywhere.

And, crucially: the timing results themselves teach the deeper lesson that **a benchmark is only as meaningful as the comparison it represents.** `fib_memo` at 0.1 µs is not "faster than `fib_iter` at 21 µs" — it's a different thing entirely (cached vs. uncached). The exercise's real output isn't the numbers; it's the understanding that fast-looking numbers require skeptical reading.