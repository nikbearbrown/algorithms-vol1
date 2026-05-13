# Chapter 7 — Divide and Conquer Strategies

*The cleverness lives in the combine step, not the recursion.*

---

In 1960, a Soviet mathematician named Anatoly Karatsuba sat through a seminar where Andrei Kolmogorov — one of the great mathematicians of the twentieth century — conjectured that multiplying two n-digit numbers required at least n² elementary operations. The schoolbook algorithm takes exactly that long: each digit of one number multiplies each digit of the other, giving n² partial products to sum. Kolmogorov believed this was fundamental, not just a limitation of the obvious method.

Karatsuba went home and within a week had a counterexample. By splitting each number into two halves and performing only three multiplications instead of four — using a clever algebraic identity to recover the fourth product from the others — he could multiply two n-digit numbers in roughly n^1.585 operations. Not n². Kolmogorov reportedly cancelled his seminar, announced the result, and published the paper himself.

What Karatsuba had found was not just a faster multiplication algorithm. He had found the design pattern that would eventually be called divide and conquer: break the problem into smaller instances of the same problem, solve them recursively, and combine the results. The combination is where the insight lives. The recursion is just the mechanism.

This chapter is about that pattern.

---

## The shape of the technique

Every divide-and-conquer algorithm has three steps, and the name says what they are.

**Divide.** Split the input into smaller subproblems of the same type — typically half the size, sometimes a third, always a constant fraction. The split usually costs `O(n)`: find the midpoint, partition around a pivot, draw a vertical line through the median. Sometimes it costs `O(1)` when the data arrives pre-split. Rarely more.

**Conquer.** Solve the subproblems recursively. When the subproblem shrinks below a threshold — one element, two points, a handful of digits — stop recursing and solve it directly. Every recursive algorithm needs this base case, or it runs forever.

**Combine.** Merge the subproblem solutions into a solution for the original. This is where the cleverness lives. For merge sort, the combine step is the `O(n)` merge of two sorted halves. For Karatsuba, it is the algebraic recombination of three products. For the closest pair of points, it is a geometric argument that limits what you need to check at the boundary. The combine step is almost always the hard part, and it is almost always where the asymptotic improvement comes from.

The running time of a divide-and-conquer algorithm satisfies a recurrence of the form:

$$T(n) = a \cdot T(n/b) + f(n)$$

where `a` is the number of recursive calls, `b` is the factor by which input shrinks, and `f(n)` is the cost of dividing and combining. The master theorem (Chapter 2 §4) turns this into a closed-form bound: compare `f(n)` to `n^(log_b a)` and the larger one dominates. When they match, a logarithm enters. When the recursive work dominates, you get `Θ(n^(log_b a))`. When the combine step dominates, you get `Θ(f(n))`.

Karatsuba's recurrence is `T(n) = 3T(n/2) + n`, giving `T(n) = Θ(n^(log_2 3)) ≈ Θ(n^1.585)`. The schoolbook algorithm's recurrence is `T(n) = 4T(n/2) + n`, giving `T(n) = Θ(n^2)`. One recursive call fewer per level — three products instead of four — compounds across all `log n` levels of recursion into a genuinely different asymptotic bound. That compounding is what divide and conquer buys.

<!-- → [CHART: recursion tree for Karatsuba (a=3, b=2) vs. schoolbook multiplication (a=4, b=2) — two trees side by side, same depth, showing how one fewer subproblem per level compounds to a different exponent at the leaves; student sees why the difference between 3 and 4 subproblems produces n^1.585 vs. n^2] -->

---

## What makes a problem divide-and-conquer shaped

Three signals together suggest the technique will work.

The first is self-similar structure. The problem on `n` items can be expressed as the same problem on `n/b` items for some constant `b > 1`, plus a combine step. Sorting a list reduces to sorting two halves and merging. Multiplying two n-digit numbers reduces to multiplying smaller-digit numbers and recombining. Finding the closest pair of points reduces to finding the closest pair in each half and handling the boundary. If you cannot express the problem on the smaller instance in the same terms as the problem on the larger one, divide and conquer is not the right frame.

The second signal is independent subproblems. The two halves can be solved without reference to each other. This is the critical distinction between divide and conquer and dynamic programming (Chapter 8). Dynamic programming applies when subproblems share work — when the answer to one subproblem is needed by many others, and recomputing it would be expensive. Divide and conquer applies when subproblems are genuinely separate. Sorting the left half of an array does not affect the right half. Multiplying the high digits of one number does not depend on the low digits of the other. If you find yourself wanting to cache subproblem results to avoid recomputing them, the structure is DP-shaped, not divide-and-conquer-shaped.

The third signal is a combine step cheaper than solving from scratch. If combining the results of two `n/2` subproblems costs `O(n²)`, you have gained nothing: the recurrence `T(n) = 2T(n/2) + n²` solves to `T(n) = Θ(n²)`, same as brute force. The combine step has to be cheap enough that the master theorem's recursive-work term dominates, or at worst matches the combine cost at each level. Merge sort's `O(n)` merge is cheap relative to the `O(n log n)` work it organizes. Karatsuba's combine is `O(n)` — three additions and some shifts — cheap relative to the three recursive multiplications. When the combine step is expensive, divide and conquer is wasted machinery.

---

## The canonical algorithms

**Merge sort** is the divide-and-conquer algorithm most engineers know first. Divide the array in half. Sort each half recursively. Merge the two sorted halves in `O(n)`. Recurrence: `T(n) = 2T(n/2) + n`. Bound: `Θ(n log n)`. Stable, `O(n)` auxiliary space. The merge step scans both halves once, comparing front elements and appending the smaller — `O(n)` work per merge, `log n` levels of merging, total `O(n log n)`. Full coverage in Chapter 4; the reason it appears here is that merge sort is the cleanest illustration of the divide-and-conquer pattern before the geometry gets complicated.

**Quicksort** fits the divide-and-conquer frame with a twist: the combination step is empty. Divide by partitioning around a pivot — elements smaller go left, larger go right. Conquer by sorting each side recursively. Combine: nothing. The pivot landed in its final position during the partition; no merge is needed. The cost is all in the divide step. With a random pivot, the expected recurrence is `T(n) = T(αn) + T((1-α)n) + n` for some `α` near `1/2`, which solves to expected `Θ(n log n)`. The randomized analysis lives in Chapter 12.

**Quickselect** finds the `k`-th smallest element in expected `O(n)` time. Same divide step as quicksort — partition around a pivot — but conquer only one side: if `k` is less than the pivot's position, recurse left; if greater, recurse right; if equal, return. The asymmetry is the point. Quicksort recurses on both sides because it needs both sorted; quickselect only needs one side because it is looking for a rank, not an ordering. The recurrence in expectation is `T(n) = T(n/2) + n`, which sums to `Θ(n)`.

**Karatsuba multiplication** is the algorithm from the opening. Divide each `n`-digit number into two `n/2`-digit halves: write `x = x₁ · B + x₀` and `y = y₁ · B + y₀` where `B = 10^(n/2)`. Four products would recover `x₁y₁`, `x₁y₀`, `x₀y₁`, `x₀y₀`. The identity `(x₁ + x₀)(y₁ + y₀) = x₁y₁ + x₁y₀ + x₀y₁ + x₀y₀` lets you compute the middle two terms from three multiplications: compute `x₁y₁`, `x₀y₀`, and `(x₁ + x₀)(y₁ + y₀)`, then subtract the first two from the third to get the middle cross terms. Three multiplications instead of four. Combine with shifts and additions, total `O(n)`. Recurrence: `T(n) = 3T(n/2) + n`. Bound: `Θ(n^(log_2 3)) ≈ Θ(n^1.585)`. In pure Python with small constants the crossover from schoolbook can be surprisingly high — often several hundred digits — because Python's arbitrary-precision integers already carry implementation optimizations. The LLM exercise for this chapter finds the empirical crossover on your machine.

**Strassen's matrix multiplication** applies the same one-subproblem-fewer trick to matrix multiplication. Dividing two `n×n` matrices into `n/2 × n/2` blocks requires eight multiplications in the standard algorithm. Strassen found a way to do it in seven, at the cost of more additions and a more involved combine step. Recurrence: `T(n) = 7T(n/2) + n²`. Bound: `Θ(n^(log_2 7)) ≈ Θ(n^2.807)`, beating the standard `Θ(n³)`. In practice, Strassen's constants are large enough that it loses to schoolbook below roughly `n = 64` to 128; BLAS implementations switch carefully at the crossover. The algebra of the seven products is on the companion page.

**Counting inversions** is a nice example of how an existing divide-and-conquer algorithm — merge sort — can be extended to compute something new at no asymptotic cost. An inversion is a pair `(i, j)` with `i < j` and `a[i] > a[j]`: an element that is ahead of a smaller element. Brute force counts all pairs in `O(n²)`. Modified merge sort counts them in `O(n log n)`: when the merge step copies an element from the right half before exhausting the left, every remaining element in the left half inverts with it. Count those during the merge. The counting piggybacks onto work the merge was doing anyway.

**Fast Fourier Transform (FFT)** computes the discrete Fourier transform of `n` samples in `O(n log n)` rather than `O(n²)`. The divide step splits by parity — even-indexed and odd-indexed samples. The combine step uses roots of unity. Cooley and Tukey published the modern form in 1965, though earlier versions existed. The FFT is foundational for signal processing, polynomial multiplication, and large-integer multiplication (at digit counts where Karatsuba's `n^1.585` is still too slow). A full treatment requires complex numbers and the algebra of roots of unity; the companion page carries a worked walkthrough.

<!-- → [TABLE: canonical D&C algorithms — columns: algorithm, recurrence, bound, combine-step cost, key constraint — one row per algorithm (merge sort, quicksort, quickselect, Karatsuba, Strassen, closest pair, FFT); student sees the full catalog and can compare what each one is trading off at a glance] -->

---

## Closest pair of points: the chapter's worked example

Given `n` points in the plane, find the pair with minimum Euclidean distance. This is the divide-and-conquer algorithm that earns the most respect, because the combine step requires a geometric argument that is not obvious.

Brute force computes all `n(n-1)/2` pairwise distances. For `n = 10⁶`, that is roughly `5 × 10¹¹` distance computations — hours on a single machine. The divide-and-conquer algorithm runs in `O(n log n)`.

**Setup.** Sort the points once by `x`-coordinate: `O(n log n)`.

**Divide.** Draw a vertical line through the median `x`-coordinate. Left half `L`, right half `R`, each of size roughly `n/2`.

**Conquer.** Recursively find the closest pair in `L` — call its distance `δ_L`. Recursively find the closest pair in `R` — call its distance `δ_R`. Set `δ = min(δ_L, δ_R)`. If the closest pair overall lies entirely within one half, we already have it.

**Combine.** The remaining case is a pair with one point in `L` and one in `R`. Any such pair with distance less than `δ` must have both points within horizontal distance `δ` of the dividing line — call this region the strip. Points further than `δ` from the line cannot form a cross-half pair closer than `δ`.

The strip can contain up to `n` points. Naively, checking all pairs in the strip is `O(n²)` and undoes all the work. This is where the geometric argument enters.

Consider any point `p` in the strip. Any point `q` in the strip that could form a closer pair must have `y`-coordinates within `δ` of `p`'s `y`-coordinate. This limits `q` to a `2δ × δ` box centered on the dividing line at `p`'s height. Now ask: how many points can fit in that box? Points in the left half of the box are pairwise at least `δ` apart (because `δ_L ≥ δ`). Points in the right half are pairwise at least `δ` apart (because `δ_R ≥ δ`). A packing argument shows that at most a small constant number of points — 7 is a standard bound, sometimes stated as 6 — can lie in the box without violating one of these spacing constraints. Therefore each point in the strip needs to be compared with only the next 7 points in `y`-sorted order. The strip scan is `O(strip-size)`, not `O(strip-size²)`.

Recurrence: `T(n) = 2T(n/2) + O(n)`. By the master theorem: `T(n) = O(n log n)`. For `n = 10⁶`, roughly `2 × 10⁷` operations — seconds instead of hours.

The pattern is the same as Karatsuba's. The cleverness is not in the recursion; the recursion is mechanical. The cleverness is in the combine step: the geometric argument that a `2δ × δ` box can contain at most a constant number of points, which turns a potentially quadratic strip scan into a linear one.

<!-- → [IMAGE: closest-pair strip diagram — vertical dividing line with δ-wide strip highlighted on each side; a 2δ × δ box shown around a sample point p; the 7-point packing argument illustrated with labeled example points in the box; student sees why the strip scan cannot be quadratic] -->

---

## When divide and conquer wins — and when it does not

Divide and conquer wins when the combine step is cheap relative to the work it organizes, and when the subproblem decomposition reveals structure that linear methods miss.

The closest-pair algorithm is the canonical example of the second point. There is no obvious linear scan that finds the closest pair: you cannot process points one at a time and maintain the answer cheaply, because a new point can be closer to any previous point. The geometric decomposition — left half, right half, strip — reveals structure that the brute-force scan ignores.

Divide and conquer loses in four identifiable ways.

First, recursion without an asymptotically-beating combine step is just recursion. Computing `n!` by `factorial(n) = n × factorial(n-1)` is recursive but not divide-and-conquer in the useful sense: the input shrinks by one, not by a factor, and the "combine" step is a single multiplication. The recurrence is `T(n) = T(n-1) + O(1)`, which solves to `Θ(n)` — same as the iterative loop. No improvement.

Second, overlapping subproblems break the independence requirement. Naive recursive Fibonacci — `F(n) = F(n-1) + F(n-2)` — is `O(2^n)` because the same subproblems are recomputed exponentially many times. The fix is memoization, which is the entry point to dynamic programming (Chapter 8). If your recursive subproblems overlap, you need DP, not divide and conquer. The structure is categorically different.

Third, unbalanced splits ruin the bound. A "divide" step that shrinks the input by one element — `T(n) = T(n-1) + n` — solves to `Θ(n²)`. Quicksort's worst case has exactly this shape: with a bad pivot, every partition produces one subproblem of size `n-1` and one of size `0`. The fix is randomization (Chapter 12) or a median-of-medians pivot guarantee (which adds `O(n)` to the divide step but preserves `O(n log n)` worst-case).

Fourth, an expensive combine step cancels the asymptotic win. If finding the median to split on costs `O(n log n)` and the combine step also costs `O(n log n)`, the recurrence can blow up beyond what the asymptotic improvement justifies. The master theorem's case 3 covers this: when `f(n)` grows faster than `n^(log_b a)`, the combine cost dominates and the total cost is `Θ(f(n))`. If `f(n)` is large, that is a bad outcome.

The corrective discipline is mechanical: write the recurrence `T(n) = aT(n/b) + f(n)` explicitly. Verify `b > 1`. Verify the combine step is correct and complete. Apply the master theorem to predict the bound. Compare the bound to the iterative alternative. If the bound is not better, reconsider whether the subproblems are actually independent and the structure is actually self-similar.

<!-- → [INFOGRAPHIC: four-panel failure-mode diagnostic — each panel shows a failing recurrence shape: (1) T(n)=T(n-1)+O(1) [shrinks by one], (2) T(n)=2T(n-1)+... [exponential overlap], (3) T(n)=T(n-1)+n [unbalanced], (4) T(n)=2T(n/2)+n² [expensive combine] — with the resulting bound and the fix labeled for each] -->

---

## Decision rules

| Problem signature | Algorithm |
|---|---|
| Sort, stability required | Merge sort |
| Sort, in-place, average case matters | Quicksort with random pivot |
| k-th order statistic | Quickselect, expected `O(n)` |
| Large-integer multiplication | Karatsuba for n ≥ a few hundred digits |
| Matrix multiplication | Strassen for n ≥ ~100; BLAS in practice |
| Closest pair of points in 2D | D&C, `O(n log n)` |
| Counting inversions in an array | Modified merge sort, `O(n log n)` |
| Discrete Fourier transform | FFT (Cooley-Tukey), `O(n log n)` |
| Self-similar problem, overlapping subproblems | DP (Chapter 8), not D&C |
| Recurrence `T(n) = aT(n/b) + f(n)` | Master theorem (Chapter 2 §4) |
| Recurrence with irregular split | Recursion tree (Chapter 2 §4) |

---

## What this chapter does not enable

This chapter does not give implementation-ready FFT or Strassen. Both require careful attention to numerical stability and constant-factor tuning that exceeds reference-book scope. For numerical FFT, consult *Numerical Recipes* or FFTW documentation. For Strassen at scale, consult BLAS implementations and the matrix-multiplication research literature. The chapter also does not cover Toom-Cook, Schönhage-Strassen, or Coppersmith-Winograd-style fast matrix multiplication — those live at the research frontier of complexity theory.

---

## Capability statement

You can now design a divide-and-conquer algorithm for a problem with self-similar structure and independent subproblems, write the recurrence and apply the master theorem to predict its running time, and recognize the canonical divide-and-conquer algorithms — merge sort, quicksort, quickselect, Karatsuba, Strassen, closest pair, FFT — along with what each one is for and what makes each combine step the key to its efficiency. You can identify when a recursive algorithm is not actually divide-and-conquer and why that matters. The next time a problem looks like it might split cleanly in half, the path to an algorithm and a bound is in your hands.

---

## Exercises

**Warm-up**

1. Write the recurrence for each of the following algorithms and identify which case of the master theorem applies: (a) merge sort, (b) binary search, (c) Karatsuba multiplication, (d) Strassen's matrix multiplication. For each, state the resulting bound and identify whether the recursive work, the combine step, or both contribute equally to the total. *(Tests: mechanical application of the master theorem to the canonical D&C algorithms.)*

2. Quicksort and quickselect share the same divide step — partitioning around a pivot. Quicksort's expected bound is `Θ(n log n)`; quickselect's is `Θ(n)`. What structural difference produces the different bounds? Write the recurrences for both and explain why they resolve differently. *(Tests: understanding asymmetry in the conquer step as the source of an asymptotic difference.)*

3. The chapter says the combine step is "almost always where the asymptotic improvement comes from." Identify the combine step for each of the following and state what makes it cheaper than recomputing from scratch: merge sort, closest pair of points, counting inversions. For counting inversions specifically, explain what the merge step is counting and why it is correct. *(Tests: reading the combine step as the key to each algorithm, not the recursion.)*

**Application**

4. A team is implementing a closest-pair algorithm and reaches the strip-scan step. They argue that the strip can contain at most `n` points and therefore scanning all pairs in the strip costs `O(n²)`, which ruins the bound. Walk through the geometric argument that limits each point to comparing with only the next 7 points in `y`-sorted order. Identify the two facts about `δ_L` and `δ_R` that make the packing argument work. *(Tests: understanding the geometric combine-step argument as the linchpin of the algorithm.)*

5. Karatsuba's algorithm reduces four multiplications to three using an algebraic identity. Write out the identity explicitly, label which three products are computed, and show how the fourth (the two cross terms together) is recovered by subtraction. Then write the recurrence that results from three subproblems of size `n/2` with `O(n)` combine cost and solve it using the master theorem. *(Tests: working through the algebraic argument that justifies the recurrence.)*

6. Use the decision table to select an algorithm for each of the following. For each, name the algorithm, state the recurrence, and explain which signal — self-similar structure, independent subproblems, or cheap combine — is the reason D&C applies. (a) Finding the median of an unsorted array in expected linear time. (b) Counting how many pairs of items in a list are out of order. (c) Evaluating a polynomial of degree `n` at `n` points simultaneously. *(Tests: translating problem descriptions into D&C structure and applying the recognition signals.)*

**Synthesis**

7. The chapter names four ways divide and conquer fails: recursion without a constant-factor divide, overlapping subproblems, unbalanced splits, and an expensive combine step. For each failure mode, construct a concrete recurrence of the form `T(n) = aT(n/b) + f(n)` (or `T(n) = aT(n-c) + f(n)` where appropriate) and show that the resulting bound is no better than brute force. *(Tests: connecting the four failure modes to their recurrence signatures and master-theorem outcomes.)*

8. Counting inversions uses merge sort's combine step to count cross-half inversions for free. Design an analogous extension: a modified divide-and-conquer algorithm that computes, simultaneously with sorting, some additional property of the array that the merge step can track at no asymptotic cost. State what property you are computing, explain what the merge step observes, and write the recurrence. *(Tests: applying the "piggyback on an existing combine step" pattern to a novel case.)*

**Challenge**

9. Strassen's algorithm reduces eight matrix-block multiplications to seven. The resulting bound is `Θ(n^(log_2 7)) ≈ Θ(n^2.807)`. Suppose a future algorithm reduced eight multiplications to six. What would the resulting bound be? Now suppose it reduced to five. What would that bound be? At what number of subproblems does the D&C matrix multiplication algorithm become `o(n²)`? What does this tell you about why the search for faster matrix multiplication algorithms has been an active research area for decades? *(Tests: applying the master theorem as a predictive tool; reasoning about the asymptotic structure of matrix multiplication improvements.)*

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
