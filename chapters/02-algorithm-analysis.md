# Algorithm Analysis

## TL;DR

Algorithm analysis is the practice of predicting how an algorithm's running time and memory grow as input grows. Reach for this chapter when you need to compare two candidate algorithms before benchmarking, justify a complexity claim in a code review, or diagnose why an `O(n log n)` solution is somehow slower than an `O(n²)` one in production. After consulting it, you can read asymptotic complexity off pseudo-code, apply the master theorem to standard recurrences, distinguish worst-case / average / amortized bounds, and recognize when those bounds mislead.

## When asymptotic analysis applies

You have two algorithms that solve the same problem, or one algorithm and a question of whether it scales. The input size is variable: today it is a thousand records, next quarter it might be a billion. You want to predict behavior in the regime you have not yet measured.

The signal is *scale uncertainty*. If the input is small and bounded — fewer than a few hundred items, always — asymptotic analysis is overkill. The constant factors dominate. A linear scan is fine. If the input is fixed and you have a profiler, measurement beats prediction. Asymptotic analysis is the right tool when you cannot afford to measure every regime, and you need a defensible answer about how cost grows.

The second signal is *algorithm comparison*. You have two candidates and want to know, in advance, which one wins as `n` grows. Asymptotic notation gives you a vocabulary for that comparison that is independent of language, machine, and constant factors.

The third signal is *recurrence*. Your algorithm is recursive, and you want to know its total cost without unrolling the recursion by hand. The master theorem (and its companions) turns recurrences into closed-form bounds.

A signal that asymptotic analysis is *not* the right tool: cache and memory-hierarchy effects dominating real-world performance. An `O(n log n)` algorithm that thrashes the cache can lose to an `O(n²)` algorithm that streams. Asymptotic analysis is the floor of the discussion in that regime, not the ceiling. §6 of this chapter handles that case explicitly.

## What you need to know first

This chapter assumes you can read pseudo-code, follow a loop nest count, and tolerate light algebra. Logarithms are used freely. Recurrences are introduced in §4 and used in Chapters 7 and 8 — if recursion itself is rusty, the recursion refresher on the companion page is the warm-up. Amortized analysis cross-references Chapter 3, where the dynamic table example is fully worked.

## Big O, Big Omega, Big Theta — the notation toolkit

Three notations, three jobs. They are constantly conflated in casual conversation; the chapter uses them precisely.

**Big O** describes an upper bound on growth. `T(n) = O(f(n))` means: there exist constants `c > 0` and `n₀` such that for all `n ≥ n₀`, `T(n) ≤ c · f(n)`. Read it as "no worse than `f(n)` up to constants." Big O is the most common notation in practice because the upper bound is what you usually care about — the worst the algorithm will ever do.

**Big Omega (Ω)** describes a lower bound. `T(n) = Ω(f(n))` means there exist `c > 0` and `n₀` such that for all `n ≥ n₀`, `T(n) ≥ c · f(n)`. Read it as "no better than `f(n)` up to constants." Use it when you want to claim an algorithm cannot be faster than some bound — for example, a comparison sort cannot be faster than `Ω(n log n)` in the worst case.

**Big Theta (Θ)** is a tight bound — both upper and lower. `T(n) = Θ(f(n))` means `T(n) = O(f(n))` and `T(n) = Ω(f(n))`. Use it when the cost matches the bound on both sides.

The casual use of "O" to mean "Θ" is everywhere, including in serious texts. The convention in this book: when a bound is tight on both sides, the prose says "Θ" or "exactly"; when the upper bound is what is being claimed, "O" stands as written. The distinction matters when you read a textbook proof or a paper — `O(n log n)` for sorting is a *guarantee* an algorithm achieves; `Ω(n log n)` is a *limit* below which no comparison sort can go.

The standard growth rates, from cheapest to most expensive, are: `O(1)` constant, `O(log n)` logarithmic, `O(n)` linear, `O(n log n)` linearithmic, `O(n²)` quadratic, `O(n³)` cubic, `O(2ⁿ)` exponential, `O(n!)` factorial. The gap between linearithmic and quadratic is where most production trade-offs live; the gap between polynomial and exponential is where Chapter 10 (NP-Completeness) lives.

## Recurrences and the master theorem

Recursive algorithms have running times described by recurrence relations. Solving the recurrence gives the closed-form bound.

The canonical form is `T(n) = a·T(n/b) + f(n)`, where `a` is the number of recursive calls per level, `b` is the factor by which each call shrinks the input, and `f(n)` is the work done outside the recursive calls (combining results, dividing the input, etc.). Merge sort, for instance, makes two calls on halves and merges in linear time: `a = 2`, `b = 2`, `f(n) = n`.

The **master theorem** classifies the solution into three cases, depending on how `f(n)` compares to `n^(log_b a)`:

- Case 1: `f(n) = O(n^(log_b a − ε))` for some `ε > 0`. The recursive work dominates. `T(n) = Θ(n^(log_b a))`.
- Case 2: `f(n) = Θ(n^(log_b a))`. The work is balanced across levels. `T(n) = Θ(n^(log_b a) · log n)`.
- Case 3: `f(n) = Ω(n^(log_b a + ε))` for some `ε > 0`, plus a regularity condition on `f`. The combine work dominates. `T(n) = Θ(f(n))`.

For merge sort: `n^(log_b a) = n^(log_2 2) = n`, and `f(n) = n`, so case 2 applies and `T(n) = Θ(n log n)`.

Two alternatives when the master theorem does not fit. The **substitution method** guesses a bound and proves it by induction. The **recursion tree method** draws the tree of recursive calls, sums work per level, and totals across levels. The recursion tree is the most useful when the recurrence is irregular — for example, `T(n) = T(n/3) + T(2n/3) + n`, where the master theorem does not apply directly but a recursion tree shows total work per level is `Θ(n)` across `Θ(log n)` levels, giving `T(n) = Θ(n log n)`.

A master theorem reference card lives on the companion page. Use it.

## Worst-case, average, and amortized

A complexity bound is meaningful only if you specify *over what*. Three common framings:

**Worst-case** is the maximum cost over all inputs of size `n`. It is the bound a hostile adversary would force. Most textbook bounds — `O(n²)` for insertion sort, `O(n log n)` for merge sort — are worst-case unless otherwise marked. Worst-case is the right framing for systems where adversarial input is possible (a public web service, a parser exposed to attackers, anything where the input is not under your control).

**Average-case** is the expected cost over a probability distribution of inputs. Quicksort's `O(n log n)` is an average-case bound assuming random pivots and a uniform distribution of inputs; its worst-case is `O(n²)`. Average-case bounds are useful when the distribution assumption is realistic (or when you can enforce it — randomized quicksort, Chapter 12, makes the assumption true by randomizing the algorithm).

**Amortized** is the average cost over a sequence of operations on a data structure, in the worst case. The dynamic array — a list that doubles its underlying buffer when it runs out of space — has `O(n)` worst-case cost on a single append (the resize) but `O(1)` amortized cost per append over `n` appends, because the resize cost is paid down by the cheap appends that follow it. Three methods exist for proving amortized bounds: the aggregate method (sum total cost, divide by `n`), the accounting method (charge each operation a fixed cost, save the surplus for later), and the potential method (define a potential function whose change funds expensive operations). Chapter 3 carries the dynamic table case study in full.

These framings are not interchangeable. A bound stated without specifying the framing is incomplete.

## Cache effects — when asymptotic analysis lies

Modern CPUs have a memory hierarchy: registers (sub-nanosecond), L1 cache (about 1 ns, ~32–64 KB), L2 (about 4 ns, hundreds of KB), L3 (about 12–40 ns, tens of MB), main memory (about 100 ns), SSD (microseconds), HDD (milliseconds). [verify] specific latency numbers for the chapter's intended publication year — these vary by architecture and improve over time.

The latency ratio between L1 and main memory is on the order of 100×. A program that hits L1 for every memory access runs roughly 100× faster than one that misses to main memory for every access, holding instruction count equal. Asymptotic analysis counts instructions and ignores this ratio. It is the floor of performance reasoning, not the ceiling.

Three practical consequences.

**Locality matters more than instruction count at small `n`.** Insertion sort runs in `O(n²)` worst-case, merge sort in `O(n log n)`. By instruction count, merge sort wins beyond a tiny `n`. By wall-clock time, insertion sort wins up to roughly `n = 32` to `n = 64`, depending on the implementation and machine, because it operates on a contiguous slice that fits in L1 cache, performs no recursive allocation, and the inner loop is a tight register operation. Production sort libraries — Python's Timsort, Java's Dual-Pivot Quicksort, the C++ standard library's introsort — all delegate small subarrays to insertion sort for this reason. The crossover point is the worked example below.

**Pointer-chasing data structures pay a cache penalty asymptotic analysis hides.** A linked list and a dynamic array both support `O(1)` append. The array, in practice, is several times faster for almost all workloads because its elements are contiguous and prefetched. The linked list pays a cache miss per node traversal. CLRS and most textbook treatments do not mention this; the asymptotic bound is identical, the constant is not.

**Algorithm choice should account for the working set.** A hash table with `O(1)` expected lookup is faster than a balanced tree's `O(log n)` lookup at small `n`, but if the hash table is sized so its buckets do not fit in L2 and the tree's nodes do, the tree wins on cache locality. The misconception "`O(n log n)` is always good enough" — engaged in §10 — is a special case of this: asymptotic analysis describes a regime, not a conclusion.

The chapter's claim is not that asymptotic analysis is wrong. It is that asymptotic analysis is necessary but not sufficient. You use it to narrow the field; you measure to pick the winner.

## Decision rules

| You are asking | Use |
| --- | --- |
| Worst-case upper bound on running time | Big O |
| Lower bound — provably cannot be faster | Big Omega |
| Tight bound, upper and lower match | Big Theta |
| Recursive algorithm, standard form `T(n) = aT(n/b) + f(n)` | Master theorem |
| Recursive algorithm, irregular split | Recursion tree |
| Sequence of operations on a data structure | Amortized analysis |
| Algorithm exposed to adversarial input | Worst-case bound |
| Algorithm operating on randomized inputs or with internal randomness | Average-case |
| Performance disagrees with the asymptotic prediction at production scale | Profile, then re-read §6 |

## Worked example — insertion sort vs merge sort crossover

Two algorithms. Insertion sort: walk left-to-right, inserting each element into the sorted prefix on its left. Time `Θ(n²)` worst-case, `Θ(n)` best-case (already-sorted input). Merge sort: divide the array in half, recursively sort each half, merge. Time `Θ(n log n)` worst-case and best-case.

By the bounds, merge sort wins for `n ≥ 2`. By the wall clock, insertion sort wins up to roughly `n = 32` on most modern hardware. [verify] specific cutoff per implementation; CPython's `list.sort()` uses 32 as the run threshold, the JDK's `Arrays.sort()` for primitive arrays uses around 47 [verify], the GNU libstdc++ `std::sort` uses 16 [verify].

Why does insertion sort win at small `n`? Three reasons.

First, the constants. A merge sort iteration involves a recursive call (stack frame setup, return), a buffer allocation (or use of a scratch buffer), and a merge step that touches each element of both halves. An insertion sort iteration is a comparison and a possible shift in a register-resident loop. The per-element constant for insertion sort is several times smaller.

Second, cache behavior. A 32-element subarray of 8-byte integers is 256 bytes — a few cache lines, easily resident in L1. Insertion sort streams through that contiguous block. Merge sort, even on the same block, allocates or addresses a scratch buffer and shuffles between two regions, doubling the working set.

Third, branch prediction. Insertion sort's inner loop is a tight conditional with high predictability on partially-sorted data. Modern branch predictors hit close to 100% on its dominant branches. Merge's branch on which side to take an element from is harder to predict.

The crossover point is determined empirically by each library's authors and frozen as a constant. CPython's Timsort, for instance, calls binary insertion sort on runs shorter than its `MIN_RUN` constant (typically 32 to 64, computed from input length to balance run count) [verify]. The JDK's `Arrays.sort()` for primitives switches to insertion sort below `INSERTION_SORT_THRESHOLD = 47` in the Dual-Pivot Quicksort implementation [verify]. Introsort in libstdc++ inserts below 16 [verify].

The asymptotic bound told you merge sort would win. At production scale on lists of millions, it does. At the recursion's leaf — subarrays of size 32 — insertion sort is faster, and every production sort library knows this.

The lesson: the bound is the field of comparison, not the conclusion. Real implementations are hybrids that respect both the asymptotic bound (at large `n`) and the constants (at small `n`).

## Failure modes — when asymptotic analysis misleads

The misconception this chapter engages: "`O(n log n)` is always good enough." It is not.

`O(n log n)` is always good enough *as an asymptotic bound* when you are sorting unstructured input at scale. It is wrong as a substitute for thinking about constants and cache effects.

Three concrete failure modes.

**Cache-oblivious bounds beating cache-friendly bounds in practice.** A radix sort runs in `O(n)` for fixed-width keys but pays a cache penalty per pass over the data and is often slower than a well-tuned `O(n log n)` quicksort on real-world string data, especially when keys are short and number of distinct values is large. The bound predicts radix wins; the cache says otherwise. Measure.

**Hidden constants from data structure overhead.** A balanced binary search tree gives `O(log n)` lookup. A hash table gives `O(1)` expected lookup. The hash table seems to dominate. But for `n` smaller than a few thousand, the tree's contiguous memory layout and lower memory overhead can beat the hash table's allocation and pointer-chasing. The bound says hash table; the constants say "depends on `n`." The engagement of this misconception is in §6 above.

**Worst-case framing where average-case is the real workload.** Quicksort is `O(n²)` worst-case, `O(n log n)` average. Used naively, an adversarial input — already sorted, with first element as pivot — degrades to `O(n²)`. Used with random pivots, the worst-case is essentially never hit. Stating "quicksort is `O(n²)`" is true and misleading; "randomized quicksort has expected running time `O(n log n)` and `O(n²)` worst-case with vanishing probability" is the honest framing. Chapter 12 carries this fully.

**Average-case framing where worst-case is the real workload.** Hash tables have `O(1)` expected lookup. Used in a public web service, an adversary can craft inputs that all hash to the same bucket and force `O(n)` per lookup — the algorithmic complexity attack. Stating "hash table is `O(1)`" is true and dangerous when the bound being violated is the one that matters.

The corrective heuristic: state the bound, state the framing (worst-case / average / amortized), and state the regime of `n` where the bound dominates the constants. A complete complexity claim has all three.

## Cross-references

For the master theorem applied to specific divide-and-conquer algorithms, see Chapter 7. For amortized analysis worked through dynamic tables, hashing, and Fibonacci heaps, see Chapter 3. For cache effects in sorting and replacement algorithms, see Chapter 4. For randomized analysis as a complexity framing, see Chapter 12. For NP-hardness as a different complexity claim entirely, see Chapter 10.

## Companion-page handoffs

Extended Python and C++ implementations of the algorithms benchmarked above, plus a profiler-driven crossover-finding harness, master theorem reference card, and a visualization gallery showing growth-rate curves at practitioner-relevant scales (`n = 10^3`, `10^6`, `10^9`), are available on the companion page at bearbrown.co/algorithms-by-bear-vol1/chapter-2.

## What this chapter does not enable

This chapter does not enable you to derive tight lower bounds for arbitrary problems. Lower-bound proofs — adversary arguments, decision-tree models, communication complexity — are graduate-level machinery; for that level of treatment, consult CLRS Chapter 8 or *Computational Complexity: A Modern Approach* by Arora and Barak. This chapter also does not cover smoothed analysis, parameterized complexity, or fine-grained complexity. Those are research frontiers; pointers live on the companion page.

## Capability statement

You can now read the asymptotic complexity off a piece of pseudo-code, compare two algorithms' complexities at scale, apply the master theorem to standard recurrences, distinguish worst-case / average-case / amortized framings, and diagnose when asymptotic analysis is misleading in practice. The next time a code review asks "what's the complexity of this," you have a defensible answer and the framing it requires.


---

## LLM Exercise — Chapter 2: The Benchmark Harness

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** The reusable benchmark harness — timing utility, log-log growth-rate fitter, plotting, and a master-theorem helper. Every subsequent chapter calls into this code.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 2 task in the algorithms-by-bear-toolkit. This is the
load-bearing infrastructure chapter — the harness I build here will be
imported by every later chapter to verify complexity claims empirically.

In `chapters/ch02_algorithm_analysis/`:

1. `harness.py` — exports these functions:

   - `time_function(fn, input_generator, sizes, repeats=5, warmup=2)`
     returns a dict `{size: [t_ns, ...]}` of per-run timings using
     `time.perf_counter_ns`. Discards `warmup` runs before measurement.
     `input_generator(size)` produces a fresh input per repeat so
     state effects don't contaminate.
   - `fit_growth_rate(timings)` returns a dict comparing log-log fits
     against `O(1)`, `O(log n)`, `O(n)`, `O(n log n)`, `O(n^2)`,
     `O(n^3)`, `O(2^n)` candidates. Reports best-fit label, R^2,
     and the inferred constant factor.
   - `plot_growth(timings, title, out_path, theoretical=None)` saves a
     log-log scatter plot with the median timing per size and an
     overlaid reference curve.
   - `master_theorem(a, b, f_exponent, log_factor=0)` — given
     `T(n) = a·T(n/b) + Theta(n^f_exponent · log(n)^log_factor)`,
     returns which case applies (1, 2, or 3) and the resulting
     asymptotic bound.

2. `implementations.py` — three demo algorithms whose complexities are
   uncontested: `linear_scan(array, target)` returning the index or -1
   (O(n)); `binary_search(sorted_array, target)` (O(log n));
   `naive_matmul(A, B)` (O(n^3)).

3. `test_implementations.py` — correctness tests against the stdlib
   equivalents on random inputs with a fixed seed.

4. `benchmarks.py` — runs each demo algorithm through the harness on
   sizes `[100, 300, 1000, 3000, 10000]` (skip 10000 for matmul; use
   `[20, 40, 80, 160, 320]` there). Assert `fit_growth_rate` correctly
   identifies the dominant term for each. Save plots to
   `chapters/ch02_algorithm_analysis/figures/`.

5. `README.md` — populated. Three decision cards. A
   "Surprising findings" section reporting at least one observation
   where empirical behavior diverges from theory — small-n constants,
   cache crossover, or branch prediction effects.

Commit with `ch02: benchmark harness and verification on demo
algorithms`. Make sure the surprising-finding note is grounded in your
actual benchmark output, not invented.
```

**What this produces:** A reusable `harness.py` (~200 lines), three benchmarked demo algorithms, three decision cards, and figures showing growth curves. Crucially: the harness is the dependency every later chapter imports.

**How to adapt this prompt:**

- *For your own project:* If you're not Python, the math of log-log regression is identical — translate the harness to your language. Plotting can use any library; matplotlib is the obvious default in Python.
- *For ChatGPT / Gemini:* Both can produce the harness. Ask for `harness.py` first, then test it on a single demo before commissioning the others.
- *For Claude Code:* Native fit. Let it benchmark and observe the surprising-findings note — do not let it invent the observation.
- *For a Claude Project:* Not the right tool. Use Claude Code.

**Connection to previous chapters:** Builds in the directory created by Chapter 1's exercise. Populates the decision-card template Chapter 1 produced.

**Preview of next chapter:** Chapter 3 implements the canonical data structures from scratch and uses the harness you just built to verify each structure's operation-cost claims empirically.
