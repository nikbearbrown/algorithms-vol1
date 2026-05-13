# Divide and Conquer Strategies

## TL;DR

Divide and conquer (D&C) breaks a problem into smaller independent subproblems of the same shape, solves them recursively, and combines results. Reach for this chapter when a problem has self-similar structure and the subproblems do not overlap. After consulting it, you can design a D&C algorithm, analyze it via the master theorem or a recursion tree (Chapter 2 §4), and recognize when D&C beats both iterative and dynamic programming alternatives.

## Recognition pattern

Three signals together suggest D&C.

*Self-similar structure.* The problem on `n` items can be expressed as the same problem on `n/2` items (or `n/3`, or any constant fraction) plus a combine step. Sorting a list reduces to sorting two halves and merging. Multiplying two `n`-digit numbers reduces to multiplying smaller-digit numbers and combining. Finding the closest pair of points reduces to finding the closest pair in each half and checking pairs that straddle the boundary.

*Independent subproblems.* The two halves can be solved without reference to each other. This is what distinguishes D&C from dynamic programming (Chapter 8) — DP applies when subproblems share work and need to be remembered. If the recursive calls would benefit from memoization, the structure is DP-shaped, not D&C-shaped.

*Combine step cheaper than the recursive work.* If combining results costs more than solving the original problem from scratch, D&C is wasted machinery. The master theorem (Chapter 2 §4) makes this precise: when `f(n) = O(n^(log_b a − ε))` for some `ε > 0`, the recursive work dominates and D&C wins; when the combine step grows faster than the recursive work, D&C may not help.

A signal D&C is *not* the right tool: subproblems are not independent (DP). Or: the problem is small enough that recursion overhead exceeds the savings (use the iterative version, with insertion sort or naive multiplication at the leaves; see Chapter 4 §3 for the production-sort precedent).

## What you need to know first

This chapter assumes recursion (companion-page refresher if rusty), the master theorem (Chapter 2 §4), and basic sorting (Chapter 4). Closest-pair and merge-sort coverage in this chapter complements rather than duplicates Chapter 4 — sorting algorithms live there in full.

## Anatomy of a D&C algorithm

Every D&C algorithm has three steps.

**Divide.** Split the input into smaller subproblems. The split is usually by a constant factor (halves, thirds), occasionally by a known boundary (the median element, the closest split-line). Cost: usually `O(n)`, sometimes `O(1)` (already-split data) or `O(n log n)` (median-based splits using selection).

**Conquer.** Solve each subproblem recursively. Base case: when the subproblem is small enough to solve directly (often `n ≤ 1` or `n ≤ some constant`).

**Combine.** Merge the subproblem solutions into a solution for the original problem. The combine step is where the cleverness usually lives. For merge sort, it is the `O(n)` merge of two sorted halves. For closest pair, it is the `O(n)` strip scan that handles boundary-crossing candidates. For Strassen's matrix multiplication, it is a clever `O(n²)` recombination of seven smaller products.

The recurrence describing the running time has the form `T(n) = a · T(n/b) + f(n)`, where `a` is the number of recursive calls, `b` is the factor by which input shrinks, and `f(n)` is the divide-plus-combine cost. The master theorem turns this into a closed-form bound (Chapter 2 §4).

## Canonical D&C algorithms

**Merge sort.** Divide: split in half. Conquer: sort each half. Combine: merge two sorted lists in `O(n)`. Recurrence: `T(n) = 2T(n/2) + n`. Bound: `Θ(n log n)`. Stable, `O(n)` auxiliary space, dominant external-memory sort. Full coverage in Chapter 4.

**Quicksort.** Divide: partition around a pivot. Conquer: sort each side. Combine: nothing — partitioning leaves the array sorted across the pivot boundary. Recurrence (with random pivot): `T(n) = T(αn) + T((1-α)n) + n` for some `α`. Expected bound: `Θ(n log n)`; worst case `Θ(n²)` on bad pivots. Full coverage in Chapter 4; randomized analysis in Chapter 12.

**Quickselect.** Find the `k`-th smallest element in `O(n)` expected time. Divide: partition around a pivot. Conquer: recurse on only the side containing position `k`. The asymmetry — recurse on one side only — is what gives `O(n)` rather than `O(n log n)`. The recurrence `T(n) = T(n/2) + n` (in expectation) sums to `Θ(n)`.

**Karatsuba multiplication.** Multiply two `n`-digit numbers in `O(n^(log_2 3)) ≈ O(n^1.585)` time, beating the schoolbook `O(n²)`. Karatsuba 1960 [verify]. Divide: split each number into high and low halves: `x = x₁·B + x₀`, `y = y₁·B + y₀`. Conquer: compute three (not four) products via the algebraic identity `(x₁+x₀)(y₁+y₀) − x₁y₁ − x₀y₀`. Combine: shift and add. Recurrence: `T(n) = 3T(n/2) + n`, giving the unusual `n^(log_2 3)` exponent.

**Strassen's matrix multiplication.** Multiply two `n × n` matrices in `O(n^(log_2 7)) ≈ O(n^2.807)` time, beating the standard `O(n³)`. Strassen 1969 [verify]. The trick: seven recursive multiplications of `n/2 × n/2` matrices instead of eight, via clever combinations. Recurrence: `T(n) = 7T(n/2) + n²`, giving `n^(log_2 7)`. In practice, Strassen's constants are large enough that schoolbook `O(n³)` wins below `n ≈ 64` to 128 [verify], and BLAS implementations switch carefully. The companion page covers the recombination algebra.

**Counting inversions.** Count pairs `(i, j)` with `i < j` and `a[i] > a[j]` in `O(n log n)` time using a modified merge sort. The merge step counts cross-half inversions while merging.

**Closest pair of points.** The chapter's worked example. See §6.

**Fast Fourier Transform (FFT).** Compute the discrete Fourier transform of `n` samples in `O(n log n)` rather than `O(n²)`. Cooley and Tukey 1965 [verify], though earlier Gauss-attributed forms exist. The divide step uses parity (even-indexed and odd-indexed samples); the combine step uses roots of unity. Foundational for signal processing, polynomial multiplication, and large-integer multiplication. Full treatment in dedicated DSP texts; the companion page carries a worked walkthrough.

## When D&C wins over alternatives

D&C wins when the combine step is cheap relative to the work it organizes. For sorting, the merge step is `O(n)` while the work it combines is also `O(n)` per level and there are `log n` levels — work distributes evenly. For multiplication, Karatsuba's `O(n)` combine adds three subproblems of size `n/2`, beating schoolbook's four; the asymptotic improvement compounds.

D&C wins over iteration when the subproblem decomposition reveals structure that linear scans miss. Closest-pair-of-points is the canonical example: brute force is `O(n²)`; D&C is `O(n log n)`. The strip-scan combine step exploits a geometric fact (only a constant number of points can lie within a `δ`-radius strip near the dividing line) that the brute-force scan does not.

D&C wins over DP when subproblems are independent. Edit distance (Chapter 8) has overlapping subproblems and lives in DP territory. Merge sort has independent subproblems — sorting the left half does not affect the right — and lives in D&C territory.

D&C loses when the divide or combine step is expensive. If finding the median to use as a split takes `O(n log n)`, and the combine step is also `O(n log n)`, the recurrence can blow up beyond what the asymptotic improvement justifies. The master theorem's case 3 covers this: when `f(n)` grows faster than the recursive work, D&C may not help.

## Decision rules

| Problem signature | Algorithm |
| --- | --- |
| Sort, stability required | Merge sort |
| Sort, in-place, average case | Quicksort with random pivot |
| `k`-th order statistic | Quickselect |
| Large-integer multiplication, n ≥ a few hundred digits | Karatsuba (or FFT-based for very large) |
| Matrix multiplication, n ≥ ~100 | Strassen (asymptotically) or BLAS dgemm (practically) |
| Closest pair of points in 2D | D&C, `O(n log n)` |
| Counting inversions | Modified merge sort, `O(n log n)` |
| Discrete Fourier transform | FFT (Cooley-Tukey), `O(n log n)` |
| Self-similar problem with overlapping subproblems | DP (Chapter 8), not D&C |
| Recurrence `T(n) = aT(n/b) + f(n)`, standard form | Apply master theorem (Chapter 2 §4) |
| Recurrence with irregular split | Recursion tree (Chapter 2 §4) |

## Worked example — closest pair of points in 2D

Given `n` points in the plane, find the pair with minimum Euclidean distance. Brute force: compute all `n(n−1)/2` pairwise distances, take the minimum. Time: `Θ(n²)`. For `n = 10⁶`, this is `5 × 10¹¹` distance computations [verify] — multi-hour wall-clock on a single machine.

The D&C algorithm runs in `O(n log n)`.

**Setup.** Sort the points once by `x`-coordinate. Maintain a parallel sort by `y`-coordinate (or pass it through the recursion). Total preprocessing: `O(n log n)`.

**Divide.** Split the points by a vertical line at the median `x`-coordinate. Left half `L`, right half `R`, each of size roughly `n/2`.

**Conquer.** Recursively find the closest pair in `L` (distance `δ_L`) and in `R` (distance `δ_R`). Let `δ = min(δ_L, δ_R)`.

**Combine.** The closest pair across the entire set is either entirely in `L`, entirely in `R`, or has one point on each side of the dividing line. The first two cases are handled by recursion. For the third case — pairs straddling the line — only points within horizontal distance `δ` of the line can possibly form a pair closer than `δ`. Call this the *strip*.

The strip can contain up to `n` points. Naively, checking all pairs in the strip is `O(n²)` and ruins the asymptotic bound.

The geometric insight: within the strip, when scanning by `y`-coordinate, any pair closer than `δ` must have `y`-coordinates within `δ` of each other. Within a `2δ × δ` box (the strip's width is `2δ`, height is `δ`), at most a small constant number of points can lie — at most 7, sometimes stated as 6 — because the points in `L` are pairwise at least `δ` apart, and so are the points in `R`, so the box admits a bounded packing [verify]. Therefore each point in the strip needs to compare itself with only the next 7 points in `y`-sorted order, not all subsequent points. The strip scan is `O(strip-size)` linear, not quadratic.

**Recurrence.** `T(n) = 2T(n/2) + O(n)`. By the master theorem, case 2: `T(n) = O(n log n)`.

For `n = 10⁶`, the algorithm runs in roughly `2 × 10⁷` operations [verify] — seconds on a single machine. The asymptotic improvement (`n²` to `n log n`) is roughly five orders of magnitude on this scale.

The cleverness lives in the strip-scan combine step. If the combine step were quadratic, the recurrence would solve to `O(n²)` and D&C would have bought nothing. The geometric bound on points-per-box is what makes the algorithm work.

Applications: nearest-neighbor queries in computational geometry, hierarchical clustering, computer graphics collision detection, geographic information systems. The companion page carries an implementation and a benchmark suite.

## Failure modes — when "D&C is just recursion" misleads

The misconception engaged: "D&C is just recursion."

D&C is recursion *with structure*. Recursion is the mechanism; D&C is the design pattern that uses recursion to attack a specific problem shape. Conflating them leads to four concrete failures.

**Recursion without an asymptotic-beating combine step.** A recursive solution that does the same total work as a brute-force iteration is recursion without D&C benefit. Computing `n!` via `factorial(n) = n * factorial(n-1)` is recursion; it does not beat the iterative `O(n)` loop. There is no divide step that shrinks the problem by a constant factor.

**Recursion with overlapping subproblems.** Naive recursive Fibonacci `F(n) = F(n-1) + F(n-2)` is `O(2^n)` because it recomputes the same subproblems exponentially many times. The fix is memoization (DP, Chapter 8), not faster recursion. The structure is wrong for D&C; the algorithm needs to remember.

**Recursion that does not balance.** A recurrence like `T(n) = T(n-1) + n` is not D&C in the master-theorem sense (the input shrinks by 1, not by a factor). It solves to `O(n²)`. The discipline of D&C is to shrink by a constant factor, not by a constant amount.

**The combine step ignored.** A merge sort with the merge step replaced by concatenation is incorrect; a closest-pair algorithm without the strip scan is `O(n²)` when the boundary case dominates. The combine step is where most of the cleverness lives. "Just recurse" without thinking about combination is a common bug pattern.

The corrective heuristic: state the recurrence `T(n) = aT(n/b) + f(n)` explicitly. Verify `b > 1` (true division). Verify the combine step `f(n)` is correct and complete. Then check the master theorem case to predict the bound. If the bound is not better than the iterative alternative, reconsider whether D&C is the right tool.

## Cross-references

For master theorem and recursion-tree analysis, see Chapter 2 §4. For merge sort and quicksort in production-sort context, see Chapter 4. For randomized quicksort analysis, see Chapter 12. For DP as the partner technique when subproblems overlap, see Chapter 8. For Bellman-Ford as a DP-but-not-D&C shortest-path algorithm, see Chapter 5.

## Companion-page handoffs

Strassen's algorithm with full recombination algebra; FFT walkthrough with butterfly diagrams; Karatsuba implementation in Python and C; closest-pair implementation with benchmark suite; recursion refresher; master theorem reference card (lives with Chapter 2 — cross-reference, not duplicate). Available at bearbrown.co/algorithms-by-bear-vol1/chapter-7.

## What this chapter does not enable

This chapter does not give implementation-ready FFT or Strassen — both require careful attention to numerical stability and constants that exceed reference-book scope. For numerical FFT, consult *Numerical Recipes* or FFTW documentation; for Strassen at scale, consult BLAS implementations and matrix-multiplication research literature. The chapter also does not cover Toom-Cook (generalization of Karatsuba), Schönhage-Strassen (asymptotically faster integer multiplication), or Coppersmith-Winograd-style fast matrix multiplication; those live at research-frontier complexity.

## Capability statement

You can now design a D&C algorithm for a problem with self-similar structure and independent subproblems, write the recurrence and apply the master theorem to predict its bound, recognize the canonical D&C algorithms (merge sort, quicksort, quickselect, Karatsuba, Strassen, closest-pair, FFT) and what each is for, and avoid the failure modes that come from confusing D&C with plain recursion. The next time a problem looks like it might split cleanly in half, the path to an algorithm and a bound is in your hands.


---

## LLM Exercise — Chapter 7: Divide-and-Conquer with Master-Theorem Verification

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** Three D&C algorithms — merge sort (revisited), closest pair of points, Karatsuba multiplication — with empirical verification that each recurrence matches the master-theorem prediction.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 7 task in the algorithms-by-bear-toolkit. The book's worked
example is closest pair of points in 2D — a non-trivial geometric
problem that splits cleanly. The exercise implements three D&C
algorithms and uses the harness from Chapter 2 to verify that the
empirical curves match what the master theorem predicts.

In `chapters/ch07_divide_and_conquer/`:

1. `implementations.py`:

   - `merge_sort` — recursive D&C version, with the merge step
     factored as its own function. (You implemented this in Chapter
     4; re-implement here in the D&C style with explicit recurrence
     for the master-theorem demonstration.)
   - `closest_pair(points)` — the full 2D D&C algorithm. Sort
     by x-coordinate, recurse, then combine using the strip check
     across the dividing line.
   - `karatsuba(x, y)` — multiplies two integers using three
     subproblem multiplications per level (instead of four).
     Compare against Python's native big-int multiplication for
     correctness.
   - `power(base, exponent)` — fast exponentiation via repeated
     squaring, as a small bonus D&C example.

2. `test_implementations.py` — correctness vs. stdlib equivalents.
   For `closest_pair`, verify against a brute-force `O(n^2)`
   reference.

3. `benchmarks.py` — the master-theorem verification study. For
   each algorithm, name the recurrence on paper:

   - merge sort: `T(n) = 2 T(n/2) + Theta(n)` → Theta(n log n)
   - closest pair: `T(n) = 2 T(n/2) + Theta(n)` → Theta(n log n)
   - Karatsuba: `T(n) = 3 T(n/2) + Theta(n)` → Theta(n^log2(3))
     ≈ Theta(n^1.585)
   - fast power: `T(n) = T(n/2) + Theta(1)` → Theta(log n)

   For each: use `harness.fit_growth_rate` to fit the empirical
   curve and assert the fit matches the master-theorem prediction.
   Save plots to `figures/`.

   For Karatsuba specifically, plot a comparison against the naive
   `O(n^2)` multiplication. Find the empirical crossover digit
   length where Karatsuba starts winning (in pure Python this is
   often surprisingly high because constant factors are large).

4. `README.md` — decision cards for each algorithm. "Surprising
   findings": the Karatsuba crossover digit length on your machine,
   and any case where the empirical fit's R^2 fell below 0.95
   (which signals a hidden constant or memory effect the master
   theorem can't see).

Commit with `ch07: divide-and-conquer with master-theorem
verification`.
```

**What this produces:** Four D&C algorithms with empirical recurrence verification, the Karatsuba/naive crossover point on your specific machine, and decision cards. The closest-pair implementation is the structural payoff — the chapter's worked example as runnable code.

**How to adapt this prompt:**

- *For your own project:* If your domain has a self-similar problem (image pyramid processing, spatial partitioning, FFT-shaped signal work), include it as a fifth algorithm.
- *For ChatGPT / Gemini:* Karatsuba is subtle; ask for the implementation in isolation with a clear correctness check before benchmarking.
- *For Claude Code:* Native fit. Let it find the crossover empirically rather than guessing.
- *For a Claude Project:* Skip.

**Connection to previous chapters:** Imports `harness` from Chapter 2; reuses the merge-sort understanding from Chapter 4 but reframes it under master-theorem analysis.

**Preview of next chapter:** Chapter 8 implements dynamic programming — LCS, edit distance, knapsack — with both top-down memoization and bottom-up tabulation, and measures the memory difference space optimization buys.
