# Chapter 2 — Algorithm Analysis

*What it means to say one way of doing something is better than another.*

---

Here is a question that sounds simple until you sit with it: how do you know, before you run a program, whether it will finish in a second or a year?

![Line chart showing wall-clock time vs](images/02-algorithm-analysis-fig-01.png)
*Figure 2.1 — Line chart showing wall-clock time vs*

Not roughly. Not "it seems fast." Actually know, with an argument you could defend, that this particular approach to the problem scales and that one doesn't.

This is what algorithm analysis is for. Not to give you a number — not to tell you "it will take 4.7 milliseconds" — but to give you a language for describing how cost *grows*. The distinction matters because growth is what breaks things. A program that takes four milliseconds on a thousand records and four years on a billion records is not a program with a performance problem. It is a program with a *scaling* problem, and no amount of hardware fixes it.

The language for talking about growth is called asymptotic notation. I want to explain what it actually means, because it is one of those things where the name — "Big O" — gets learned long before the concept does, and once the name is in your head, it's easy to stop thinking.

---

## The Thing You're Actually Measuring

Start with a concrete situation. You have a list of names and you want to find one particular name in it. The simplest approach: start at the beginning, check each name, stop when you find it or run out of names.

How long does that take? It depends on two things: where the name is in the list, and how long the list is.

If you're lucky, the name is first. One check. If you're unlucky, it's last — or not there at all. `n` checks, where `n` is the length of the list.

Now suppose the list doubles. Instead of a thousand names, ten thousand. In the worst case, your unlucky search now takes ten times longer. The cost *grows proportionally* to the size of the input.

This is what `O(n)` means. Not "the algorithm takes `n` steps." It means: as the input grows, the cost grows *in proportion* to the input. Double the input, roughly double the cost. Multiply the input by a million, multiply the cost by a million. The `O` is a shorthand for "on the order of" — a class of growth, not a precise measurement.

The formal definition, which I'll give you because it's worth having, says: `T(n) = O(f(n))` if there exist constants `c > 0` and `n₀` such that for all `n ≥ n₀`, `T(n) ≤ c · f(n)`. What that says in plain English is: past some threshold input size, the cost is bounded above by `f(n)` up to a constant multiplier. The constant absorbs all the details we're choosing to ignore — how fast the machine is, what programming language you used, how the memory is laid out — and leaves behind only the shape of growth.

The shape is what survives. Everything else is a detail.

![Visual breakdown of the formal Big O definition](images/02-algorithm-analysis-fig-02.png)
*Figure 2.2 — Visual breakdown of the formal Big O definition*

---

## Three Notations, Three Questions

In practice you'll encounter three versions of this idea, and they answer three different questions.

**Big O** answers: what's the worst the algorithm could do? It's an upper bound. "This will never be worse than `f(n)` up to constants." Most of the time, this is what you care about — the ceiling, not the floor. A system that's usually fast but occasionally catastrophic is a broken system.

**Big Omega (Ω)** answers: what's the best it could possibly do? It's a lower bound. You use it when you want to make a claim like "no algorithm can solve this problem faster than `f(n)`." The most famous such claim: no comparison-based sorting algorithm can sort `n` items faster than `Ω(n log n)` in the worst case. It's not that no one has found a faster algorithm — it's that *no such algorithm can exist*, and the argument for why requires the Ω notation to even state.

**Big Theta (Θ)** answers: what does the algorithm *actually* cost, tightly? It's both an upper and lower bound — the cost matches `f(n)` on both sides up to constants. When you say merge sort runs in `Θ(n log n)`, you're saying the cost is exactly linearithmic in growth: not just "at most `n log n`" but "at least `n log n`" too. The bound is tight.

In casual conversation — including, honestly, in textbooks — "O" often does the work of "Θ." People say "merge sort is `O(n log n)`" when they mean "Θ(n log n)." The distinction matters when precision matters: an `O(n log n)` claim guarantees an upper bound; a `Θ(n log n)` claim says the algorithm actually achieves that and can't do fundamentally better.

I'll try to use them precisely, and flag when I'm simplifying.

| Item | Meaning |
| --- | --- |
| "lower bound", "proving no algorithm can do better", "comparison-sort floor argument" | student should be able to pick the right notation for a given claim |

---

## The Standard Growth Rates

There is a hierarchy of growth rates you should internalize, because they tell you immediately whether you're in a good situation or a bad one.

From cheapest to most expensive:

`O(1)` — constant. The cost doesn't grow with input at all. Fetching an element from an array by index. A hash table lookup (on average). Whatever `n` is, the cost is roughly the same.

`O(log n)` — logarithmic. Binary search on a sorted array. Each step cuts the remaining problem in half, so the total number of steps is the number of times you can halve `n` before reaching 1 — which is `log₂ n`. If `n` is a billion, `log₂ n` is about 30. Thirty comparisons to find an element in a billion-element sorted list. This is why binary search feels almost magical the first time you really understand it.

`O(n)` — linear. The linear scan above. Reading every element in a list. Growth is direct and proportional.

`O(n log n)` — linearithmic. Merge sort. Quicksort (on average). The comparison-sort lower bound sits here. Most of what you reach for in practice when you need to sort lives in this class.

`O(n²)` — quadratic. Two nested loops, each running over `n` elements. Insertion sort in the worst case. Naive string matching. The gap between `O(n log n)` and `O(n²)` is where most of the consequential production trade-offs live — an `O(n²)` algorithm that's fast enough at ten thousand records will not be fast enough at a million.

`O(2ⁿ)` and `O(n!)` — exponential and factorial. These describe algorithms that enumerate all possible combinations or permutations of the input. They are essentially off the table for any non-trivial `n`. Chapter 10 covers why certain problems seem to force you into this territory and what you can do about it.

![Log-scale plot of all seven growth classes from](images/02-algorithm-analysis-fig-03.png)
*Figure 2.3 — Log-scale plot of all seven growth classes from*

---

## What Recursion Costs

A recursive algorithm's running time is described by a *recurrence* — an equation where the cost at size `n` is defined in terms of the cost at smaller sizes. To know the total cost, you solve the recurrence.

The canonical form is:

$$T(n) = a \cdot T\!\left(\frac{n}{b}\right) + f(n)$$

where `a` is the number of recursive subproblems at each level, `b` is the factor by which each subproblem shrinks the input, and `f(n)` is the work done outside the recursive calls — dividing the input, combining the results.

Merge sort, to make this concrete: you make two recursive calls (`a = 2`), each on half the input (`b = 2`), and the merge step takes linear time (`f(n) = n`). So `T(n) = 2T(n/2) + n`.

To solve this recurrence without unrolling it by hand, there's a theorem — the *master theorem* — that classifies the solution into three cases based on how `f(n)` compares to `n^(log_b a)`. Think of `n^(log_b a)` as the cost that would come from the recursive calls alone if the subproblems dominated; `f(n)` is the cost from the combining work.

**Case 1:** The recursive work dominates — `f(n)` grows slower than `n^(log_b a)`. The total cost is `Θ(n^(log_b a))`.

**Case 2:** The work is balanced — `f(n)` grows at the same rate as `n^(log_b a)`. The total cost picks up a logarithmic factor: `Θ(n^(log_b a) · log n)`.

**Case 3:** The combining work dominates — `f(n)` grows faster than `n^(log_b a)` (with a regularity condition I'll spare you). The total cost is `Θ(f(n))`.

For merge sort: `n^(log₂ 2) = n^1 = n`, and `f(n) = n`, so we're in Case 2. Total cost: `Θ(n log n)`. The theorem delivers the answer in three lines that would otherwise take a page of induction.

A reference card with the master theorem applied to the standard recurrences is on the companion page. The more important thing to understand is the structure of the reasoning: you're asking whether the cost comes from the leaves of the recursion tree (Case 1), from every level equally (Case 2), or from the top (Case 3). The master theorem is just a formula for that question.

When the master theorem doesn't apply — when the split is unequal, or the recurrence is irregular — the *recursion tree method* is your fallback. Draw the tree. Label the work at each level. Sum across levels. For instance, the recurrence `T(n) = T(n/3) + T(2n/3) + n` has unequal splits, so the master theorem doesn't fit directly. But the recursion tree shows that each level sums to `n` (even though the tree is unbalanced), and the tree has `O(log n)` levels, so the total is `Θ(n log n)`.

![Recursion tree for T(n) = T(n/3) + T(2n/3)](images/02-algorithm-analysis-fig-04.png)
*Figure 2.4 — Recursion tree for T(n) = T(n/3) + T(2n/3)*

---

## Three Ways to Frame the Same Algorithm

A complexity bound without a framing is an incomplete statement. The framing specifies what set of inputs you're taking the cost over.

**Worst-case** is the maximum cost over all inputs of size `n` — the cost when an adversary picks your input. Most textbook bounds are worst-case unless otherwise marked. Insertion sort's worst case is `Θ(n²)` — achieved when the input is sorted in reverse. Merge sort's worst case is `Θ(n log n)` — and so is its best case.

**Average-case** is the expected cost over a probability distribution on inputs. Quicksort's average-case running time, assuming random inputs (or a randomized pivot selection), is `O(n log n)`. Its worst case is `O(n²)` — achieved when the pivot is always chosen badly. The distinction matters: in the average case, quicksort is as good as merge sort and usually faster in practice due to smaller constants and better cache behavior. In the worst case, it's catastrophically slow on sorted input, which is one of the most common real-world inputs.

**Amortized** is the average cost per operation over a *sequence* of operations in the worst case — not "what's the average input" but "what's the average cost across many operations on the same data structure."

The canonical example is the dynamic array — a list that doubles its underlying buffer when it runs out of space. A single append that triggers a resize is `O(n)` — it copies all `n` existing elements. But how often does resizing happen? If you start with a buffer of size 1 and keep doubling, resizing happens at sizes 1, 2, 4, 8, ..., n. The total cost of all resizes across `n` appends is `1 + 2 + 4 + ... + n = 2n`, which is `O(n)`. Spread over `n` appends, that's `O(1)` per append, *amortized*. 

The expensive operation is real. But you pay for it in installments across the cheap operations that follow, and the average works out to constant time.

![Bar chart showing per-operation cost of n append](images/02-algorithm-analysis-fig-05.png)
*Figure 2.5 — Bar chart showing per-operation cost of n append*

Three methods exist for proving amortized bounds more carefully: the *aggregate method* (total the cost, divide by operations), the *accounting method* (charge each operation a fixed amortized cost and save the surplus for later), and the *potential method* (define a potential function that acts like a "stored energy" in the data structure, funding expensive operations from the energy accumulated during cheap ones). Chapter 3 works through all three on the dynamic table.

---

## When the Analysis Lies

Modern computers have a memory hierarchy. Registers, sitting inside the processor, are sub-nanosecond. L1 cache — a small, fast memory a centimeter from the computation — is roughly a nanosecond. L2, a few nanoseconds. L3, ten to forty nanoseconds. Main memory, a hundred nanoseconds. An SSD, microseconds. A hard drive, milliseconds.

The ratio between L1 and main memory is roughly 100:1 in latency. A program that hits L1 for every access runs about a hundred times faster than a program that misses to main memory on every access, holding instruction count constant.

| level | approximate latency | approximate size |
| --- | --- | --- |
| registers, L1 cache, L2 cache, L3 cache, main memory, SSD, HDD | student should internalize the 100× gap between L1 and main memory that makes cache behavior decisive at production scale | A concrete checkpoint for applying the chapter concept. |

Asymptotic analysis counts instructions. It does not count cache misses.

This is not a flaw — it's a deliberate choice, the right abstraction for its purpose. But it means the bound is not the whole story.

Here is a concrete example. Insertion sort runs in `Θ(n²)` worst case. Merge sort runs in `Θ(n log n)` worst case. By instruction count, merge sort dominates past `n = 2`. By wall-clock time, on modern hardware, insertion sort wins up to roughly `n = 32` to `n = 64`, depending on the machine and implementation.

Why? At `n = 32`, the elements fit in a few cache lines — a few hundred bytes — easily resident in L1. Insertion sort streams through a contiguous block of memory that's already in cache. Merge sort, even on the same thirty-two elements, does a recursive call (setting up a stack frame), addresses a scratch buffer for merging (potentially elsewhere in memory), and shuffles between two memory regions. The per-element constant for insertion sort is several times smaller, and at small `n`, that constant is all that matters.

This is why every production sort library is a hybrid. Python's Timsort delegates to binary insertion sort on runs shorter than a threshold (typically 32 to 64 elements, computed from input length). The JDK's dual-pivot quicksort switches to insertion sort below a threshold around 47. GNU's `std::sort` inserts below 16. The asymptotic bound governs behavior at large `n`; the constants govern behavior at small `n`; the crossover point is found empirically and baked in as a constant.

![Wall-clock time vs](images/02-algorithm-analysis-fig-06.png)
*Figure 2.6 — Wall-clock time vs*

The misconception worth naming: "`O(n log n)` is always good enough." It is good enough as an asymptotic class when you're sorting unstructured input at scale. It is not a substitute for thinking. Three ways this goes wrong:

First, a radix sort runs in `O(n)` for fixed-width integer keys. Faster asymptotically. But it pays a cache penalty on each of its passes over the data, and on real-world string keys with many distinct values, a well-tuned `O(n log n)` quicksort often wins in wall time. The bound says radix sort wins. The cache says measure.

Second, a hash table offers `O(1)` expected lookup. A balanced binary search tree offers `O(log n)`. The hash table dominates asymptotically. But if the hash table is large enough that its buckets don't fit in cache, and the tree's nodes do, the tree wins on repeated lookups in a working set that fits in L2. The bound says hash table. The cache says "what's `n`, and what fits in cache?"

Third, an algorithm stated with an average-case bound applied to a worst-case workload. Hash tables have `O(1)` expected lookup — expected over a random distribution of keys. Feed a public web service a hash table, and an adversary can craft inputs that all collide in the same bucket, forcing `O(n)` per lookup. This is the algorithmic complexity attack, and it's been used in practice. The bound is correct; the framing was wrong for the context.

The corrective habit: state the bound, state the framing (worst-case, average-case, amortized), and note the regime of `n` where the bound governs. A complete complexity claim has all three.

---

## What to Reach For

When you have two algorithms and want to compare them at scale, reach for Big O and compare growth classes. When you want to prove that no algorithm can do better than a certain rate, reach for Big Omega. When you need a tight bound, reach for Theta.

When your algorithm is recursive and fits `T(n) = aT(n/b) + f(n)`, apply the master theorem. When the split is unequal or the recurrence is irregular, draw the recursion tree.

When you need a worst-case guarantee for adversarial input, use worst-case analysis. When your algorithm has internal randomization or your input distribution is well-understood, use average-case analysis. When you're reasoning about a sequence of operations on a data structure, use amortized analysis.

When the algorithm's empirical behavior disagrees with the asymptotic prediction — when the `O(n²)` algorithm is mysteriously faster than the `O(n log n)` one in production — profile, then re-read the cache effects section above.

The analysis doesn't replace measurement. It narrows the field so measurement is worth doing.

---

## Where This Goes

You can now read the asymptotic complexity off a piece of pseudocode, compare two algorithms' complexities at scale, apply the master theorem to standard recurrences, distinguish worst-case from average-case from amortized framings, and recognize when asymptotic analysis is being used to mislead rather than illuminate.

That's the vocabulary. The next three chapters put it to work: Chapter 3 analyzes the canonical data structures from first principles and uses amortized analysis to justify claims that look wrong until you work them out. Chapter 7 applies the master theorem to divide-and-conquer algorithms beyond merge sort. Chapter 12 makes average-case analysis rigorous through randomization.

The question we started with — how do you know, before running a program, whether it scales? — now has a partial answer: you describe the shape of its growth, you specify the conditions under which that description holds, and you stay honest about what the description doesn't tell you.

---

## Exercises

### Warm-Up

**1.** For each algorithm below, identify its growth class (O(1), O(log n), O(n), O(n log n), O(n²), or O(n³)) and state whether your answer is a worst-case, average-case, or amortized bound. Justify each answer in one sentence.

- Looking up a value in a Python dictionary by key
- Finding the minimum element in an unsorted list of `n` integers
- Printing all pairs `(i, j)` where `0 ≤ i < j < n`
- Binary search on a sorted array of `n` elements

*Tests: reading growth class off an algorithm description; distinguishing notation framings.*

**2.** The following are informal complexity claims. For each one, identify whether it uses Big O, Big Omega, or Big Theta — or whether the framing is ambiguous. Then rewrite it precisely using the correct notation.

- "Merge sort takes `n log n` time."
- "No comparison sort can do better than `n log n`."
- "Bubble sort is about `n²`."

*Tests: mapping informal language onto precise notation; distinguishing O, Ω, Θ.*

**3.** A dynamic array starts empty. You perform 16 appends. Draw or describe the sequence of per-operation costs, marking which appends trigger a resize. What is the total cost of all 16 appends? What is the amortized cost per append?

*Tests: applying the aggregate method for amortized analysis; the dynamic array case.*

---

### Application

**4.** Apply the master theorem to each recurrence. State which case applies, show why, and give the asymptotic bound.

- `T(n) = 4T(n/2) + n`
- `T(n) = 2T(n/2) + n`
- `T(n) = T(n/2) + n²`
- `T(n) = 3T(n/3) + n log n`

For the last one, determine whether the master theorem applies directly and explain your reasoning.

*Tests: applying the master theorem; recognizing Case 1, 2, and 3; identifying when the theorem doesn't apply.*

**5.** You have two implementations of a function that computes whether a list contains any duplicate values:

- **Implementation A:** For each element, scan the rest of the list to check for a match. `O(n²)` worst-case.
- **Implementation B:** Insert each element into a hash set; if insertion fails (element already present), return true. `O(n)` expected.

(a) At what input size does Implementation B's advantage become decisive? At what input size might Implementation A still be competitive, and why?

(b) Suppose this function is called on user-supplied input in a public web service. Explain why Implementation B's expected-case bound could be violated and what the consequence would be.

*Tests: comparing algorithms across complexity classes; applying worst-case vs. average-case framing to a realistic security scenario.*

**6.** A colleague claims: "I benchmarked my `O(n log n)` sort against a library's `O(n²)` sort on lists of 50 elements and mine was slower. Something must be wrong with my implementation."

Explain to your colleague, precisely, what is actually happening. Your explanation should reference at least two distinct reasons why the `O(n²)` sort might win at `n = 50`, and what the claim "`O(n log n)` is always faster" misses.

*Tests: applying cache effects and constant-factor reasoning; diagnosing the misconception that asymptotic class determines wall-clock winner.*

---

### Synthesis

**7.** The recurrence `T(n) = T(n/3) + T(2n/3) + n` describes the cost of a divide-and-conquer algorithm that splits the input unevenly. The master theorem does not apply directly.

(a) Use the recursion tree method to find `T(n)`. Show the work at each level of the tree, argue why each level sums to `n`, and state the depth of the tree.

(b) What bound does this give? Is it tight (Θ) or just an upper bound (O)? Justify.

*Tests: applying the recursion tree method to an irregular recurrence; constructing a Θ argument.*

**8.** You are designing a key-value store that will handle up to 10 million records. Two candidate data structures are under consideration: a balanced binary search tree (O(log n) lookup, worst-case) and a hash table (O(1) lookup, expected).

Write a structured recommendation that: (a) gives the asymptotic comparison, (b) identifies at least one scenario where the tree outperforms the hash table in practice despite the bound, and (c) names the framing condition under which the hash table's bound breaks. Your recommendation should read as a complete complexity claim — bound, framing, and regime.

*Tests: synthesizing asymptotic notation, cache effects, and framing conditions into a practical engineering recommendation.*

---

### Challenge

**9.** The algorithmic complexity attack described in the cache effects section exploits the gap between expected-case and worst-case bounds for hash tables. Research (or reason from first principles) how modern hash table implementations defend against this attack. What mechanism converts the vulnerability from a worst-case guarantee into a practical non-issue? What does the defended implementation trade off to achieve this?

Your answer should name the specific technique, explain why it works in terms of the probability distribution on inputs, and state what cost it imposes.

*Tests: connecting amortized and average-case reasoning to a real security design decision; reasoning about randomized algorithms and their probabilistic guarantees.*

**10.** Smoothed analysis is a complexity framework introduced by Spielman and Teng in 2001 to explain why simplex — an algorithm with exponential worst-case complexity — performs so well in practice. Without looking it up (or after looking it up): explain in your own words what smoothed analysis measures that worst-case and average-case analysis don't. Why would an algorithm have polynomial smoothed complexity but exponential worst-case complexity? What does this say about the limits of worst-case analysis as a predictor of real-world performance?

*Tests: reasoning about complexity frameworks beyond the chapter's scope; evaluating asymptotic analysis as a modeling choice with tradeoffs, not a ground truth.*

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

## Prompts

Use these prompts with Claude to generate interactive D3 v7 versions of the
figures in this chapter. Each produces a standalone HTML file you can open
in a browser and modify freely.

**Prerequisites:** Load `brutalist/CLAUDE.md` and `brutalist/DESIGN.md` into
your Claude project context before using these prompts. They define the stack,
naming conventions, color system, and typography the figures use.

---

### Figure 2.1 — Line chart showing wall-clock time vs

Create a standalone D3 v7 HTML file for Figure Line chart showing wall-clock time vs. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: line chart showing wall-clock time vs. input size n for a concrete O(n) algorithm, e.g. linear scan on a list — student should see that doubling n roughly doubles time, grounding the proportionality intuition before the formal definition appears. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/02-algorithm-analysis-fig-01.html`

---

### Figure 2.2 — Visual breakdown of the formal Big O definition

Create a standalone D3 v7 HTML file for Figure Visual breakdown of the formal Big O definition. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: visual breakdown of the formal Big O definition — annotate T(n) ≤ c·f(n) for all n ≥ n₀ with callouts labeling each component: "c absorbs machine speed, language, memory layout"; "n₀ is the threshold past which the bound holds"; "f(n) is the shape that survives" — student should see exactly what the constants are hiding. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/02-algorithm-analysis-fig-02.html`

---

### Figure 2.3 — Log-scale plot of all seven growth classes from

Create a standalone D3 v7 HTML file for Figure Log-scale plot of all seven growth classes from. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: log-scale plot of all seven growth classes from O(1) to O(n!) evaluated at n = 1 through n = 30 — student should see that the curves are indistinguishable at small n and diverge catastrophically past n = 20; this is the visceral argument for why growth class matters more than constants at scale. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/02-algorithm-analysis-fig-03.html`

---

### Figure 2.4 — Recursion tree for T(n) = T(n/3) + T(2n/3)

Create a standalone D3 v7 HTML file for Figure Recursion tree for T(n) = T(n/3) + T(2n/3). Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: recursion tree for T(n) = T(n/3) + T(2n/3) + n — show the unbalanced tree with level-by-level work labeled; annotate that each level still sums to n despite the imbalance; label the depth as O(log n) — student should see why an irregular split doesn't change the answer here and understand the recursion tree method as a visual proof technique. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/02-algorithm-analysis-fig-04.html`

---

### Figure 2.5 — Bar chart showing per-operation cost of n append

Create a standalone D3 v7 HTML file for Figure Bar chart showing per-operation cost of n append. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: bar chart showing per-operation cost of n append operations on a dynamic array — x-axis is operation number 1..n, y-axis is cost; cheap O(1) appends appear as short bars, resize events appear as tall spikes at powers of 2; amortized cost shown as a flat horizontal line at O(1) — student should see visually that the spikes are real but infrequent enough that the average stays constant. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/02-algorithm-analysis-fig-05.html`

---

### Figure 2.6 — Wall-clock time vs

Create a standalone D3 v7 HTML file for Figure Wall-clock time vs. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: wall-clock time vs. n for insertion sort and merge sort on the same machine, n ranging from 1 to 200 — show the crossover point where merge sort's line dips below insertion sort's; annotate the crossover region with the library thresholds (16, 32–64, 47); student should see that the O(n²) algorithm genuinely wins at small n and understand why hybrid sorts exist. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/02-algorithm-analysis-fig-06.html`
