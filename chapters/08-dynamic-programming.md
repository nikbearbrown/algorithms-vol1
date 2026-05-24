# Chapter 8 — Dynamic Programming

*What it means to remember what you've already figured out.*

---

Here is a question about counting. How many ways can you make change for a dollar using quarters, dimes, nickels, and pennies? You could enumerate all the combinations — there are 242 of them — but the more interesting question is how you'd write a program to count them. A naive approach would recurse: to count ways to make a dollar, count ways to make a dollar minus one penny with pennies available, plus ways to make a dollar minus one nickel with nickels available, and so on. Work through the recursion tree and you'll find the same sub-question appearing hundreds of times. "How many ways to make 50 cents using only dimes and nickels?" gets computed not once but over and over, identical work repeated at every branch of the tree that leads to it.

This is the problem dynamic programming solves. Not a lack of insight, not a complicated algorithm — just the observation that in a large class of problems, a naive recursion recomputes the same thing exponentially many times when it only needs to compute it once.

---

## The Two Properties You're Looking For

Before you can apply dynamic programming, you have to recognize when it applies. Two properties together signal it.

The first is *optimal substructure*. The optimal solution to the whole problem can be built from optimal solutions to smaller subproblems. If you're finding the shortest path from A to Z and the optimal path goes through M, then the portion of the path from A to M must itself be a shortest path from A to M — otherwise you could swap it for a shorter one and improve the overall path. This is optimal substructure. Greedy algorithms and divide-and-conquer both rely on it too; it's not unique to dynamic programming.

The second is *overlapping subproblems*. The same subproblems appear repeatedly in a naive recursive formulation. This is what distinguishes dynamic programming from divide-and-conquer. Merge sort divides a list and recursively sorts each half — but the two halves are independent, they never share subproblems. Fibonacci, on the other hand: computing `F(n)` calls `F(n-1)` and `F(n-2)`; computing `F(n-1)` calls `F(n-2)` and `F(n-3)`; `F(n-2)` appears twice already, and the repetition compounds exponentially. There are only `n` distinct Fibonacci numbers to compute, but naive recursion computes `O(2ⁿ)` of them.

Dynamic programming collapses that redundancy. Compute each subproblem once. Store the result. When you need it again, look it up.

![Recursion tree for F(6) ](images/08-dynamic-programming-fig-01.png)
*Figure 8.1 — Recursion tree for F(6) *

A third thing worth checking is that the subproblem space is manageable. DP works when the number of distinct subproblems is polynomial. For the longest common subsequence of two strings of lengths `m` and `n`, there are `mn` distinct subproblems — fine. For the traveling salesman problem with bitmask states, there are `n · 2ⁿ` subproblems — still exponential, but manageable up to about `n = 25`. For problems whose subproblem space is genuinely exponential with no compressible structure, DP won't save you.

---

## How to Design a Recurrence

The actual skill in dynamic programming is not implementing the algorithm — once you have a recurrence, implementation is mechanical. The skill is designing the recurrence. Four steps, with the difficulty concentrated in the third.

**Define the subproblem.** Say precisely what `OPT(i)`, or `OPT(i, j)`, or whatever your subproblem index is, actually means. Not "the DP value at `i`" — that's circular. Something like: "the maximum total value achievable from items `1` through `i` in a knapsack of remaining capacity `w`." The definition names the state space. Everything else derives from it.

**Write the recurrence.** Express `OPT(i)` in terms of strictly smaller subproblems. This is usually a `min` or `max` over the choices available at step `i`: take item `i` or don't, match characters or skip, split the matrix chain here or there. Each choice has a cost, and the optimal is the best of them.

**Identify base cases.** What is `OPT(0)`? `OPT(0, j)`? `OPT(i, 0)`? These are the smallest subproblems, where the recursion bottoms out. Wrong base cases are the most common source of subtle bugs — the logic of the recurrence is right but the answer is off by one everywhere. Test against small examples you can verify by hand before you trust the code on large inputs.

**Choose evaluation order.** Two options: top-down or bottom-up. I'll come back to this.

The recurrence is the artifact. If the recurrence is right, the rest is just filling in a table.

![The four-step recurrence design process as a vertical](images/08-dynamic-programming-fig-02.png)
*Figure 8.2 — The four-step recurrence design process as a vertical*

---

## Top-Down and Bottom-Up

There are two ways to evaluate a DP recurrence, and they're equivalent in the answers they produce. They differ in how they traverse the subproblem space.

**Top-down memoization** writes the recursion as the recurrence specifies, but caches results. When the function is called with argument `i`, it first checks whether `OPT(i)` has already been computed; if so, it returns the cached value immediately. If not, it computes the value by calling itself recursively on smaller subproblems, stores the result, and returns it. The function touches each distinct subproblem exactly once, but only the subproblems that are actually needed.

This is the closer formulation to the mathematical recurrence. You write what the problem means, not how to fill a table. It's easier to get right initially and easier to extend with new state dimensions. The cost is function-call overhead, recursion depth limits (Python defaults to 1,000 frames), and slightly harder space optimization.

**Bottom-up tabulation** allocates an array or grid indexed by subproblem identifier and fills it in dependency order — ensuring that when you compute `OPT(i)`, all smaller subproblems it references have already been computed. For a 1D recurrence this usually means iterating `i` from 0 to `n`. For a 2D recurrence this usually means filling row by row or along diagonals, depending on which earlier entries each cell reads.

The bottom-up approach has lower overhead — it's just array accesses in a loop — and it makes space optimization straightforward. If the recurrence at row `i` depends only on row `i-1`, you only need to store two rows at a time. The cost is that you must think explicitly about dependency order, and you compute all subproblems even if some are never needed.

In practice: top-down for prototyping and correctness, bottom-up for production when speed or memory is the constraint. For most problems the choice is stylistic. For very large inputs where space optimization matters, bottom-up wins.

| Item | Meaning |
| --- | --- |
| implementation form, subproblems computed, recursion overhead, space optimization ease, best used when | A concrete checkpoint for applying the chapter concept. |
| fill each cell concisely | A concrete checkpoint for applying the chapter concept. |
| student should be able to make the choice in under a minute for a new problem | A concrete checkpoint for applying the chapter concept. |

---

## The Idea, Made Concrete: Edit Distance

The best single example of dynamic programming — the one that shows both the technique and its reach — is edit distance.

Given two strings, the edit distance between them is the minimum number of single-character operations — insertions, deletions, or substitutions — needed to transform one string into the other. Transform "kitten" into "sitting" and you need six operations. Transform "Saturday" into "Sunday" and you need three. The algorithm to compute this was described by Levenshtein in 1965 and the standard DP formulation by Wagner and Fischer in 1974.

Define `OPT(i, j)` as the edit distance between the first `i` characters of string `A` and the first `j` characters of string `B`.

Base cases: `OPT(i, 0) = i` — to transform `A[1..i]` to an empty string, delete all `i` characters. `OPT(0, j) = j` — to build `B[1..j]` from nothing, insert `j` characters.

Recurrence: for any `i > 0` and `j > 0`, look at the last characters `A[i]` and `B[j]`. Three choices:

- Delete `A[i]`: costs 1, and the remaining problem is `OPT(i-1, j)`.
- Insert `B[j]`: costs 1, and the remaining problem is `OPT(i, j-1)`.
- Substitute (or match): if `A[i] = B[j]`, cost is 0; if not, cost is 1; and the remaining problem is `OPT(i-1, j-1)`.

So:

$$\text{OPT}(i, j) = \min\left(\text{OPT}(i-1, j) + 1,\; \text{OPT}(i, j-1) + 1,\; \text{OPT}(i-1, j-1) + [A[i] \neq B[j]]\right)$$

where `[A[i] ≠ B[j]]` is 1 if the characters differ and 0 if they match.

Fill in an `(m+1) × (n+1)` table, where `m` and `n` are the string lengths. Every cell looks left, up, and diagonally up-left. The answer is in the bottom-right corner.

![Fully filled edit distance table for "kitten" vs](images/08-dynamic-programming-fig-03.png)
*Figure 8.3 — Fully filled edit distance table for "kitten" vs*

Time: `O(mn)`. Space: `O(mn)` for the full table, or `O(min(m, n))` if you only need the distance and not the alignment — because each row depends only on the previous row, you can alternate between two arrays.

Why does this work? Because the problem has optimal substructure: any optimal edit sequence either matches the last characters, deletes the last character of `A`, or inserts the last character of `B`. Whatever it does at the end, the remaining operations on the shorter prefix must themselves be optimal — if they weren't, you could replace them with something better. And the subproblems overlap: `OPT(i-1, j-1)` is referenced from `OPT(i, j)`, `OPT(i+1, j)`, `OPT(i, j+1)`, and so on.

A greedy algorithm fails here. The locally cheapest edit at one position can force expensive edits elsewhere — there's no way to make a locally good decision without knowing the full context of what follows. Dynamic programming is the matched tool.

---

## Space Optimization

The `O(mn)` table for edit distance is fine when `m` and `n` are thousands. When they're hundreds of thousands — as in DNA sequence alignment, where sequences can be millions of characters long — the full table is unworkable. Two strings of length 10⁵ would require `10¹⁰` cells. That's too much memory by several orders of magnitude.

The space optimization exploits a simple observation: the recurrence for row `i` reads only from row `i-1`. You don't need the entire table. Keep two arrays — the current row and the previous row — and alternate. Memory drops from `O(mn)` to `O(min(m, n))`.

![Comparison of full table vs](images/08-dynamic-programming-fig-04.png)
*Figure 8.4 — Comparison of full table vs*

The cost is that you lose the ability to reconstruct the optimal alignment. With the full table you can trace back from `OPT(m, n)` to `OPT(0, 0)`, following the choices that produced the minimum at each cell, and recover the actual edit sequence. With two rows you only have the final distance, not the path.

When you need both the distance and the alignment on long sequences, Hirschberg's algorithm (1975) recovers the alignment in `O(m + n)` space using a divide-and-conquer combination of forward and backward passes over the table. The alignment is recovered recursively. The time remains `O(mn)`, but the space drops from `O(mn)` to `O(m + n)`. The derivation is one of the more elegant constructions in the algorithms canon, and it applies the divide-and-conquer idea from Chapter 7 to a DP problem.

---

## The Catalog of Standard Patterns

There is a small set of DP patterns that recur across a wide range of problems. Recognizing your problem as an instance of a known pattern is most of the work.

**1D recurrence over a sequence.** The subproblem is defined over a single index, and the recurrence looks a few positions back. Fibonacci is the trivial case. Weighted interval scheduling — pick a non-overlapping set of intervals to maximize total weight — reduces to a 1D recurrence once intervals are sorted by finish time.

**2D recurrence over two sequences.** The subproblem is defined over a pair of indices, one from each sequence. Longest common subsequence and edit distance are both this pattern. The table is filled row by row.

**2D recurrence over an interval.** The subproblem is defined over a contiguous subarray `[i, j]`, and the recurrence considers all splits of that interval. Matrix chain multiplication — given a sequence of matrices, find the parenthesization that minimizes scalar multiplications — is this pattern. The table is filled by increasing interval length.

**0/1 knapsack.** Pick a subset of items to maximize value subject to a weight constraint. The subproblem is `OPT(i, w)` — optimal value from items `1..i` with remaining capacity `w`. The recurrence either includes item `i` (reducing capacity by `w_i`) or excludes it. Time `O(nW)` where `W` is the capacity — pseudo-polynomial, meaning it's polynomial in the numeric value of `W` but exponential in the bit representation. For `W = 10⁹` this is impractical; for `W = 10⁴` it's instant.

**Bitmask DP over subsets.** When the subproblem is defined over a subset of a small set of elements, and the set is small enough that subsets can be represented as bitmasks, DP over subsets is feasible. The Held-Karp algorithm for TSP runs in `O(n² · 2ⁿ)` — exponential, but vastly better than the `O(n!)` brute force, and exact for `n` up to about 20 to 25.

**Graph DP: Bellman-Ford and Floyd-Warshall.** Bellman-Ford computes single-source shortest paths with possible negative edges by iterating over `(vertex, iteration)` pairs — the subproblem is "shortest path to vertex `v` using at most `k` edges." Floyd-Warshall computes all-pairs shortest paths via `OPT(i, j, k)` — shortest path from `i` to `j` using only intermediate vertices from the set `{1, ..., k}`. Chapter 5 covers both in the context of graph algorithms; the DP framing is what makes them correct on negative-weight graphs.

| subproblem definition form | table shape | fill order | canonical example | time complexity |
| --- | --- | --- | --- | --- |
| 1D sequence, 2D two-sequence, 2D interval, 0 | 1 knapsack, bitmask subsets, graph DP | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. |
| columns: subproblem definition form, table shape, fill order, canonical example, time complexity | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. |
| student should be able to match an unfamiliar problem to its pattern by reading down the "subproblem definition form" column | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. |

---

## When DP Is Not the Answer

Dynamic programming is not the universal optimization tool. Three concrete ways it fails — and what to use instead.

**Greedy would have worked.** Interval scheduling, Huffman coding, minimum spanning trees, and the fractional knapsack all admit provably optimal greedy algorithms. The exchange argument is rigorous; greedy is not a sloppy shortcut. Forcing DP onto these problems buys you an extra `O(n)` or `O(n²)` factor for no benefit. Chapter 6 shows how to recognize when greedy is provably optimal.

**Subproblem space exponential, no compression.** TSP with 100 cities has too many subset states for bitmask DP. The right tools at that scale are exact-exponential algorithms (Chapter 10), approximation algorithms (Chapter 11), or heuristics. The bitmask DP earns its place up to `n ≈ 25`; past that, something else takes over.

**Pseudo-polynomial bounds that look polynomial.** Knapsack with `n = 100` items and capacity `W = 10⁹` runs in `O(nW) = 10¹¹` operations — exponential in the bit length of `W`. The same algorithm with `W = 10⁴` runs in `10⁶` operations — instant. The word "polynomial" in "pseudo-polynomial" is doing work: the algorithm is polynomial in the numeric values of the inputs, which may themselves be exponentially large relative to the number of bits used to represent them.

There is one more failure mode worth naming because it's the most common: a DP solution that computes wrong answers because the subproblem definition is imprecise. "The best value at step `i`" is not a subproblem definition — it doesn't identify the state. "The maximum total value achievable from items `1` through `i` in a knapsack with remaining capacity `w`" is. When a DP is mysteriously wrong, the subproblem definition is almost always where the bug lives. Make it precise, make it testable, verify it on small cases by hand.

---

## What You Can Now Do

You can recognize when a problem has the two properties — optimal substructure and overlapping subproblems — that make dynamic programming applicable. You can define a subproblem precisely, write a recurrence, identify base cases, and choose between top-down memoization and bottom-up tabulation. You can apply space optimization when the full table is too large. You can recognize the canonical patterns — 1D sequence, 2D sequence, interval, knapsack, bitmask subsets, graph DP — and know when each applies. And you can recognize when greedy or divide-and-conquer is the right tool instead, and reach for it.

The technique has a reach that extends far beyond the problems in this chapter. The defining question is always the same: does the problem have optimal substructure? And does the naive recursion recompute the same subproblems? When both answers are yes, write the recurrence. The rest is filling in a table.

Chapter 9 applies this to network flow — where the maximum flow through a network is found by a sequence of augmentations, each exploiting optimal substructure over the residual graph. Chapter 11 shows how DP recurrences can be adapted to produce approximation algorithms, including the FPTAS for knapsack, when the exact DP is too slow.

---

## Exercises

### Warm-Up

**1.** For each of the following problems, state whether it has (a) optimal substructure, (b) overlapping subproblems, both, or neither. For any that have both, briefly explain why. For any that have only one, explain why the other is absent and name the algorithm paradigm that fits instead.

- Computing the `n`th Fibonacci number
- Sorting an array using merge sort
- Finding the shortest path in a weighted graph with non-negative edges
- Determining whether a string is a palindrome
- Counting all possible ways to tile a 2×n board with 1×2 dominoes

*Tests: recognizing the two DP signals; distinguishing DP from D&C and greedy at the recognition stage.*

**2.** A friend implements the following recurrence for the 0/1 knapsack problem: "the best value I can get from the remaining items." Their implementation passes small test cases but gives wrong answers on larger ones.

(a) What is wrong with the subproblem definition?

(b) Write a precise subproblem definition that correctly captures the state space. What are the two dimensions of state, and why are both necessary?

(c) Write the base cases for your corrected definition.

*Tests: diagnosing an imprecise subproblem definition; identifying the state space; establishing base cases.*

**3.** Fill in the edit distance table for the strings `A = "cat"` and `B = "cart"` by hand (a 4×5 grid). Show every cell's value and the three arrows (from left, above, and diagonally above-left) that could have produced it. Trace the optimal path from the bottom-right corner back to the top-left and name each operation in the alignment.

*Tests: executing the edit distance recurrence manually; reading a traceback path; naming edit operations.*

---

### Application

**4.** Consider the weighted interval scheduling problem: you have `n` intervals, each with a start time, finish time, and value. You want to select a non-overlapping subset with maximum total value. This reduces to a 1D DP.

(a) Sort the intervals by finish time. Define the subproblem `OPT(i)` precisely.

(b) Write the recurrence. The key step is: for each interval `i`, you either include it (gaining its value plus the best solution on all intervals compatible with `i`) or exclude it (inheriting the best solution through interval `i-1`). How do you find "the last interval compatible with `i`" efficiently?

(c) What is the time complexity of the full solution, including the preprocessing?

*Tests: reducing a scheduling problem to a 1D DP recurrence; designing the recurrence with a binary search sub-step; reasoning about time complexity.*

**5.** The space-optimized edit distance (two alternating rows) computes the edit distance but cannot recover the alignment. Explain Hirschberg's insight: why can the alignment be recovered using only `O(m + n)` space if you're willing to do `O(mn)` total work spread across a divide-and-conquer structure?

Your answer should describe (a) what the forward pass computes, (b) what the backward pass computes, (c) how the midpoint of the optimal alignment is identified from the two passes, and (d) why this recurses correctly to give the full alignment.

*Tests: reasoning about the space-time tradeoff for traceback; understanding Hirschberg's divide-and-conquer combination with DP.*

**6.** Knapsack with `n = 50` items and capacity `W = 10⁶`. Your colleague says: "DP gives `O(nW) = 5 × 10⁷` operations — that's fine, we should just run it."

(a) Compute the actual memory required if each table cell is a 64-bit integer. Is this feasible on a machine with 8 GB RAM?

(b) Now suppose `W = 10⁹`. Recompute. What is the correct characterization of the algorithm's complexity that explains this failure?

(c) Suggest two alternative approaches for the `W = 10⁹` case: one exact (for what range of `n` does it apply?) and one approximate (name the technique and its guarantee).

*Tests: applying the pseudo-polynomial analysis; memory calculation; knowing when to switch from exact DP to approximation.*

---

### Synthesis

**7.** You are given a sequence of matrix dimensions, and you want to find the parenthesization that minimizes the total number of scalar multiplications to compute the product. This is the matrix chain multiplication problem.

(a) Define the subproblem `OPT(i, j)` precisely. What does it represent, and what is its domain?

(b) Write the recurrence. The key branching structure is over all possible split points `k` in the range `[i, j)`. What are the base cases?

(c) In what order must the table be filled, and why? (Hint: think about what "subproblem size" means for an interval recurrence.)

(d) Trace through the computation for the chain `[10, 30, 5, 60]` (four matrices with dimensions 10×30, 30×5, 5×60). What is the optimal cost and the optimal parenthesization?

*Tests: designing a 2D interval recurrence; determining fill order; executing a small instance by hand; reading off the optimal solution.*

**8.** Bellman-Ford and Floyd-Warshall are both shortest-path algorithms from Chapter 5. Restate both as DP recurrences explicitly.

For Bellman-Ford: define the subproblem in terms of (vertex, number of edges allowed), write the recurrence, identify the base case, and explain why `|V|−1` iterations suffice.

For Floyd-Warshall: define the subproblem in terms of (source, destination, set of allowed intermediate vertices), write the recurrence, identify the base cases, and explain how the algorithm's three-nested-loop structure corresponds to the DP table fill order.

Then: explain how seeing both algorithms as DP reveals why they are correct on negative-weight edges when Dijkstra is not.

*Tests: translating graph algorithms into DP recurrences; connecting Chapter 5 and Chapter 8; using DP framing to explain correctness.*

---

### Challenge

**9.** The longest increasing subsequence (LIS) of a sequence is the longest subsequence in which each element is strictly greater than the previous one. For the sequence `[3, 1, 4, 1, 5, 9, 2, 6]`, the LIS has length 4 (one example: `[1, 4, 5, 9]`).

(a) Define a DP subproblem and recurrence that solves LIS in `O(n²)`.

(b) An `O(n log n)` solution exists using a patience sorting structure. Describe the key insight: what does the patience sort maintain, and how is binary search used to achieve the logarithmic factor?

(c) LIS and LCS are both "longest subsequence" problems. Can LIS be reduced to LCS? If so, describe the reduction and analyze the resulting time complexity. Is it better or worse than the direct `O(n²)` DP?

*Tests: designing a novel DP recurrence; understanding an algorithmic improvement from O(n²) to O(n log n); recognizing reductions between DP problems.*

**10.** The edit distance recurrence as presented uses unit costs for insertion, deletion, and substitution. In DNA sequence alignment (Needleman-Wunsch), the substitution cost is not uniform — it is drawn from a scoring matrix that assigns different costs to different character substitutions (e.g., replacing adenine with guanine costs less than replacing adenine with cytosine, reflecting biological plausibility).

(a) Modify the edit distance recurrence to accommodate a general substitution cost function `sub_cost(A[i], B[j])`. Does the modification affect the time or space complexity?

(b) Smith-Waterman is the *local* alignment variant: instead of aligning the full strings, it finds the highest-scoring alignment of any substring of `A` against any substring of `B`. What modification to the recurrence and base cases achieves this? What does the change reveal about the relationship between global and local alignment as DP problems?

(c) For sequences of length 10⁶ (a realistic genome length), even the `O(min(m,n))` space-optimized version requires `O(mn)` time — about `10¹²` operations. Real bioinformatics pipelines don't do this. What class of technique do they use instead, and what does that technique trade away for its speed?

*Tests: extending a DP recurrence to handle non-uniform costs; understanding global vs. local alignment as DP variants; reasoning about the gap between theoretical algorithms and production bioinformatics.*

---

## LLM Exercise — Chapter 8: DP Catalog with Memoization vs Tabulation

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** The canonical DP catalog — LCS, edit distance, 0/1 knapsack, matrix chain — implemented twice (top-down memoized and bottom-up tabulated), with empirical comparison of speed and memory between the two.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 8 task in the algorithms-by-bear-toolkit. The book's signature
worked example is edit distance in production (spell checkers, DNA
alignment, fuzzy database matching). The exercise implements the
canonical DP catalog in both top-down and bottom-up forms and measures
the trade-offs Bear claims.

In `chapters/ch08_dynamic_programming/`:

1. `implementations.py` — each algorithm implemented twice:

   - `lcs_memo(x, y)` and `lcs_tab(x, y)` — longest common
     subsequence. Return both length and reconstructed subsequence.
   - `edit_distance_memo(s, t)` and `edit_distance_tab(s, t)` — with
     the standard Levenshtein operations (insert, delete, substitute).
     The tab version should support O(min(m,n)) space optimization
     when only the distance is needed; expose this via a flag.
   - `knapsack_01_memo(weights, values, capacity)` and
     `knapsack_01_tab(weights, values, capacity)`.
   - `matrix_chain_memo(dimensions)` and
     `matrix_chain_tab(dimensions)` — return both the minimum cost
     and the optimal parenthesization.
   - `fibonacci_naive(n)`, `fibonacci_memo(n)`, `fibonacci_tab(n)`,
     `fibonacci_iterative(n)` — the canonical four, included
     because the contrast is pedagogically perfect.

2. `test_implementations.py` — both versions of each algorithm
   produce identical results on the same input. The memo vs. tab
   equivalence is the test.

3. `benchmarks.py` — three studies:

   **Study A: memo vs. tab speed and memory.** For each algorithm,
   measure wall time and peak memory (use `tracemalloc`) at varying
   input sizes. Plot. Report which approach wins where.

   **Study B: space optimization.** For edit distance with the
   O(min(m,n)) optimization on vs. off, measure memory savings on
   strings of length 10000. Report the saving and the speed
   trade-off.

   **Study C: when memoization beats recomputation.** Plot
   `fibonacci_naive` against the memoized variant for n in
   `[10, 20, 25, 30, 35]`. The exponential blow-up of the naive
   version is the textbook moment — capture it as numbers.

4. `README.md` — decision cards for each algorithm and a meta-card
   "when to choose memo vs. tab." "Surprising findings": which DP
   algorithm benefited most from space optimization, and the largest
   memory delta you observed.

Commit with `ch08: DP catalog with memo-vs-tab and space-optimization
study`.
```

**What this produces:** Eight DP algorithm pairs, two-axis comparison (time × memory), space-optimization measurements, and decision cards. The Fibonacci naive-vs-memo blowup is the most memorable plot in the whole repo.

**How to adapt this prompt:**

- *For your own project:* If you work in NLP, replace the LCS demo with diff-style sequence alignment on real text. If bioinformatics, alignment against real DNA sequences.
- *For ChatGPT / Gemini:* The space-optimized edit distance is subtle (you only need two rows at a time); ask for explicit tracing of which cell-row you're keeping. Mistakes are common.
- *For Claude Code:* Native fit. Run `tracemalloc` from inside the script; do not let it estimate memory usage.
- *For a Claude Project:* Skip.

**Connection to previous chapters:** Imports `harness` from Chapter 2. The fibonacci comparison reinforces the recurrence analysis from Chapter 7.

**Preview of next chapter:** Chapter 9 implements network flow — max-flow via Ford-Fulkerson and Edmonds-Karp — and the canonical bipartite-matching reduction that turns "match workers to jobs" into "push flow through a graph."

## Prompts

Use these prompts with Claude to generate interactive D3 v7 versions of the
figures in this chapter. Each produces a standalone HTML file you can open
in a browser and modify freely.

**Prerequisites:** Load `brutalist/CLAUDE.md` and `brutalist/DESIGN.md` into
your Claude project context before using these prompts. They define the stack,
naming conventions, color system, and typography the figures use.

---

### Figure 8.1 — Recursion tree for F(6) 

Create a standalone D3 v7 HTML file for Figure Recursion tree for F(6) . Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: recursion tree for F(6) — show the full binary tree with each node labeled F(k); highlight duplicate subtrees in a distinct color (F(4) appears twice, F(3) three times, F(2) five times); annotate total node count as O(2ⁿ) vs. distinct subproblems as n; student should see viscerally why the redundancy is exponential and why caching collapses it to linear. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/08-dynamic-programming-fig-01.html`

---

### Figure 8.2 — The four-step recurrence design process as a vertical

Create a standalone D3 v7 HTML file for Figure The four-step recurrence design process as a vertical. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: the four-step recurrence design process as a vertical flow — (1) Define subproblem: annotate that the definition must name the state precisely, show a good vs. bad example side by side; (2) Write recurrence: show the min/max branching structure; (3) Base cases: flag this as where most bugs live; (4) Evaluation order: branch to top-down and bottom-up; student should be able to use this as a checklist when designing their first recurrence. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/08-dynamic-programming-fig-02.html`

---

### Figure 8.3 — Fully filled edit distance table for "kitten" vs

Create a standalone D3 v7 HTML file for Figure Fully filled edit distance table for "kitten" vs. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: fully filled edit distance table for "kitten" vs "sitting" (7×8 grid) — annotate the base case row and column; trace the three arrow directions (left=insert, up=delete, diagonal=substitute/match) from one interior cell; highlight the traceback path from bottom-right to top-left that recovers the optimal alignment; student should be able to read off the edit operations from the path and verify the count of 6. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/08-dynamic-programming-fig-03.html`

---

### Figure 8.4 — Comparison of full table vs

Create a standalone D3 v7 HTML file for Figure Comparison of full table vs. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: side-by-side comparison of full table vs. two-row optimization for the same edit distance computation — left: full (m+1)×(n+1) grid with all cells filled, shaded to show that only the last two rows are ever read; right: two horizontal arrays labeled "prev" and "curr" alternating, with arrows showing which cells feed into the current computation; annotate memory cost of each; student should see exactly what is discarded and what is retained. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/08-dynamic-programming-fig-04.html`
