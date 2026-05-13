# Dynamic Programming

## TL;DR

Dynamic programming (DP) solves a problem by combining solutions to overlapping subproblems, remembering each subproblem's answer rather than recomputing it. Reach for this chapter when a problem has optimal substructure *and* overlapping subproblems — when a recursive formulation would recompute the same subproblems exponentially many times. After consulting it, you can recognize DP-amenable structure, write a recurrence, choose top-down memoization or bottom-up tabulation, and apply space optimization when the table is the constraint.

## Recognition pattern

Two properties together signal DP.

*Optimal substructure.* The optimal solution to the problem can be constructed from optimal solutions to its subproblems. This is the same property greedy and D&C require — DP is not unique here.

*Overlapping subproblems.* The same subproblems are encountered multiple times during a naive recursive computation. This is what distinguishes DP from D&C. Naive recursive Fibonacci computes `F(n-2)` twice, `F(n-3)` three times, `F(n-4)` five times — `O(2^n)` total work for `O(n)` distinct subproblems. Memoization collapses the recomputation.

A third signal worth checking: the subproblem space is small enough to enumerate. DP works when the number of distinct subproblems is polynomial. For the longest common subsequence of two strings of lengths `m` and `n`, there are `mn` distinct subproblems — manageable. For the traveling salesman with bitmask states, there are `n · 2^n` subproblems — exponential, but small enough for `n ≤ 20` or so. For problems whose subproblem space is genuinely exponential and not amenable to bitmask compression, DP is not the tool.

A signal DP is *not* the right approach: subproblems are independent (D&C, Chapter 7) or the matroid structure holds (greedy, Chapter 6). When greedy works, DP also works but pays an unnecessary `O(n)` or `O(n²)` factor. The misconception engaged in §10 — continuing from Chapter 6 — is the inverse: that DP is the rigorous default and greedy is the sloppy shortcut. When greedy is provably optimal, it is rigorous and cheaper.

## What you need to know first

This chapter assumes Big O and recurrences (Chapter 2 §3–4), basic recursion (companion-page refresher), and the contrast with D&C (Chapter 7). For Bellman-Ford as a DP algorithm on graphs, see Chapter 5 §5. For the contrast with greedy, see Chapter 6 §10. Recurrence design — the load-bearing skill of this chapter — has no shortcut; the worked examples and §3 walkthrough are how you build it.

## Designing a DP recurrence

Designing a DP solution is a four-step process. The third step is where most of the difficulty lives.

**1. Define the subproblem.** State precisely what `OPT(i)` (or `OPT(i, j)`, etc.) represents. This is harder than it looks. "Maximum value achievable using items 1 through `i`" is a definition. "The DP value at index `i`" is not. The subproblem definition determines everything that follows.

**2. State the recurrence.** Express `OPT(i)` in terms of smaller subproblems. The recurrence usually has the form `OPT(i) = best of (option A using OPT(i-1), option B using OPT(i-2), ...)`. Each option corresponds to a decision the algorithm makes at this step.

**3. Identify the base case.** What is `OPT(0)`, `OPT(empty)`, or whatever the smallest subproblem is? Wrong base cases produce subtle off-by-one errors that pass small tests and fail at scale.

**4. Choose evaluation order.** Top-down (recursion + memoization, fill the table on demand) or bottom-up (iteration, fill the table in dependency order). Top-down is closer to the recursive specification and easier to write correctly; bottom-up has lower constant overhead and enables space optimization.

The recurrence is the artifact. Once it is written and verified, the implementation is mechanical.

## Top-down vs bottom-up

**Top-down (memoization).** Write the recursive function as the recurrence specifies. Cache results in a hash table or array. On every call, check the cache first; compute and store on miss.

```
def OPT(i):
    if i in memo: return memo[i]
    if i == base_case: return base_value
    result = recurrence(...)
    memo[i] = result
    return result
```

Pros: tracks the recurrence directly; computes only the subproblems actually needed; easy to add new state dimensions.

Cons: function-call overhead; stack-depth limits on recursion; harder to space-optimize.

**Bottom-up (tabulation).** Allocate a table indexed by subproblem identifier. Fill the table in an order that ensures every dependency is computed before it is read.

```
table[base_case] = base_value
for i in increasing order of dependency:
    table[i] = recurrence(table[smaller_indices])
return table[final]
```

Pros: tight loop, no recursion overhead; enables space optimization (often only the last row or two is needed); cache-friendly when access pattern is regular.

Cons: must determine dependency order explicitly; computes all subproblems even if some are not needed.

The choice is often style or constraint-driven. For prototyping, top-down is faster to write. For production, bottom-up is faster to run and easier to optimize.

## Space optimization

A DP table that is `O(n)` wide and `O(n)` tall costs `O(n²)` space. Often, the recurrence at row `i` depends only on row `i-1` (and sometimes `i-2`). When that is true, the table can be reduced to two rows (or one), bringing space down to `O(n)`.

Edit distance is a canonical example. The full `(m+1) × (n+1)` table costs `O(mn)` space. Each row depends only on the previous row, so two `O(n)` arrays alternate — current row and previous row. Memory drops from gigabytes to megabytes for long sequences.

The cost: traceback (reconstructing the optimal solution, not just its value) becomes harder. With the full table you can walk back from `(m, n)` to `(0, 0)`. With two rows you can compute the optimal value but not the alignment. Hirschberg's algorithm (1975) [verify] recovers the alignment in `O(m + n)` space using a divide-and-conquer combination of forward and backward passes.

Space-optimize when the optimal value is what you need; keep the full table when traceback is required (or use Hirschberg).

## Canonical DP problems

**Fibonacci.** The introductory DP problem. `F(n) = F(n-1) + F(n-2)`. Naive recursion: `O(2^n)`. DP: `O(n)` time, `O(n)` space, or `O(1)` space with rolling variables.

**0/1 knapsack.** Pick a subset of `n` items, each with weight `w_i` and value `v_i`, maximizing total value subject to total weight `≤ W`. Recurrence: `OPT(i, w) = max(OPT(i-1, w), OPT(i-1, w - w_i) + v_i)` if `w_i ≤ w`, else `OPT(i-1, w)`. Time and space: `O(nW)`. This is *pseudo-polynomial* — polynomial in the values of the inputs but not in their bit length. For `W` exponentially large in `n`, the algorithm is exponential.

**Longest common subsequence (LCS).** Length of the longest subsequence shared by two strings. Recurrence: if `A[i] = B[j]`, `OPT(i, j) = OPT(i-1, j-1) + 1`; else `max(OPT(i-1, j), OPT(i, j-1))`. Time `O(mn)`, space `O(mn)` or `O(min(m, n))` optimized.

**Edit distance.** Minimum edits (insertion, deletion, substitution) to transform one string into another. Worked example below.

**Matrix chain multiplication.** Given matrices `A_1, A_2, ..., A_n` with compatible dimensions, find the parenthesization that minimizes scalar multiplications. Time `O(n³)`, space `O(n²)`. The recurrence: `OPT(i, j) = min over k in [i, j) of (OPT(i, k) + OPT(k+1, j) + cost(merge))`.

**Optimal binary search tree.** Given access frequencies for keys, build the BST minimizing expected access cost. Time `O(n³)`, reducible to `O(n²)` via Knuth's optimization (1971) [verify].

**Bellman-Ford.** Single-source shortest paths with possibly-negative edges via `|V|−1` iterations of edge relaxation. The recurrence is over `(vertex, iteration)` pairs. See Chapter 5 §5.

**Floyd-Warshall.** All-pairs shortest paths via `OPT(i, j, k) = min(OPT(i, j, k-1), OPT(i, k, k-1) + OPT(k, j, k-1))` for paths using intermediate vertices in `{1, ..., k}`. Time `O(V³)`, space `O(V²)`. See Chapter 5 §5.

**Subset sum / partition.** Pseudo-polynomial DP on a set's possible subset sums. Used in cryptanalysis, combinatorial design, and as a teaching example for NP-hard problems with practical DP solutions.

**Held-Karp TSP.** Traveling salesman in `O(n² · 2^n)` time and `O(n · 2^n)` space using bitmask DP over subsets. Beats brute-force `O(n!)` substantially but still exponential. Solves TSP exactly for `n` up to about 20–25 [verify]. See Chapter 10 for what to do when `n` is larger.

## Decision rules

| Problem signature | Approach |
| --- | --- |
| Optimal substructure + overlapping subproblems | DP |
| Optimal substructure + independent subproblems | D&C (Chapter 7) |
| Optimal substructure + matroid / exchange argument | Greedy (Chapter 6) |
| Recurrence depends only on previous row | Bottom-up with `O(width)` space |
| Recurrence has irregular dependency pattern | Top-down with memoization |
| Need optimal value only | Space-optimize |
| Need optimal solution (traceback) | Keep full table or use Hirschberg's algorithm |
| Subproblem space exponential, no bitmask compression | DP fails — try approximation (Chapter 11) or randomization (Chapter 12) |
| Pseudo-polynomial OK (small numeric input) | DP works (knapsack, subset-sum) |

## Worked example — edit distance in production

The edit distance (Levenshtein distance) between two strings is the minimum number of single-character edits — insertions, deletions, substitutions — to transform one string into the other. Levenshtein 1965 [verify], with the standard DP solution attributed to Wagner-Fischer 1974 [verify].

**Recurrence.** Let `OPT(i, j)` = edit distance between `A[1..i]` and `B[1..j]`.

Base cases: `OPT(i, 0) = i` (delete `i` characters), `OPT(0, j) = j` (insert `j` characters).

Recurrence:
```
OPT(i, j) = min(
    OPT(i-1, j) + 1,                                  # delete A[i]
    OPT(i,   j-1) + 1,                                # insert B[j]
    OPT(i-1, j-1) + (0 if A[i] == B[j] else 1)        # substitute (or match)
)
```

Time: `O(mn)`. Space: `O(mn)` for the full table or `O(min(m, n))` for the value-only version. For two strings of length 10⁵, the full table is 10¹⁰ cells — unworkable. The space-optimized version is 10⁵ cells — trivial.

**Production applications.**

*Spell checking.* The autocomplete and spell-check engines in mobile keyboards, web browsers, and IDEs use edit distance (often with weighted operations or transposition included — Damerau-Levenshtein) to rank suggestions. Time per word is `O(d · L)` where `d` is the dictionary size and `L` is word length, made tractable in practice by indexing structures like BK-trees or Levenshtein automata. [verify] specific implementation details for any cited engine.

*DNA sequence alignment.* The Needleman-Wunsch algorithm (1970) [verify] is global edit distance with a substitution-cost matrix tuned to biological substitution rates. Smith-Waterman (1981) [verify] is the local-alignment variant — find the best-scoring substring alignment rather than aligning entire sequences. Both are `O(mn)` DP. Modern bioinformatics pipelines use heuristic prefilters (BLAST, HMMER) to reduce the candidate set, then run full DP on the survivors.

*Fuzzy string matching in databases.* Searching for near-matches of a user-supplied query string is edit distance computed against an index. PostgreSQL's `pg_trgm` extension uses trigram similarity as a fast filter; full edit distance is computed on candidates that survive the filter. [verify] specific extension behavior.

*Source-code diff.* The `diff` utility computes the longest common subsequence of two file versions, equivalent to a unit-cost edit distance. Myers' algorithm (1986) [verify] computes the diff in `O((m+n)D)` time where `D` is the number of differences, beating standard `O(mn)` when the files are similar.

*Code-completion ranking.* Modern editors rank completion candidates by a combination of frequency, scope, and edit distance from the typed prefix. The DP runs on every keystroke in tight time budgets; space optimization is essential.

**Why DP is the right tool here.** The subproblem space is `(m+1)(n+1)` — polynomial in input size. The same subproblems are referenced many times in a naive recursive formulation. The recurrence is local — each cell depends only on three neighbors. The optimal substructure is provable — an optimal alignment of `A[1..m]` to `B[1..n]` either matches the last characters, deletes `A[m]`, or inserts `B[n]`, and the rest of the alignment is optimal for the corresponding smaller subproblem.

A greedy algorithm fails on edit distance: the locally optimal edit at one position can force suboptimal edits elsewhere. A D&C algorithm is wasteful: subproblems overlap heavily and would be recomputed. DP is the matched tool.

## Failure modes — when DP is not the answer

The misconception engaged here continues from Chapter 6: "DP is the rigorous one; greedy is sloppy."

DP is not the universal optimization hammer. Three concrete failures.

**Greedy would have worked, and would have been faster.** Interval scheduling, Huffman coding, MST, and fractional knapsack all admit provably optimal greedy algorithms. Forcing DP onto these problems is over-engineering. The exchange argument is a rigorous proof; greedy is a rigorous algorithm; together they are correct and cheaper.

**Subproblem space exponential.** TSP with `n = 100` has too many subset states even for bitmask DP. DP is the wrong tool; the right tools are exact-exponential algorithms (Chapter 10), approximation (Chapter 11), or heuristics. The Held-Karp DP earns its place at moderate `n`; beyond that, it fails.

**Pseudo-polynomial bound looks polynomial.** Knapsack with `n = 100` items and `W = 10⁹` is `O(nW) = 10¹¹` — exponential in the bit length of `W`. The same knapsack with `W = 10⁴` is `O(nW) = 10⁶` — instant. The pseudo-polynomial nature means the algorithm's practicality depends on the magnitude of numeric inputs, not just their count.

**Top-down memoization without an iteration limit.** Naive memoization in a language with recursion-depth limits (Python defaults to 1000) blows up on long chains. Increase the limit, switch to iterative bottom-up, or use a manual stack.

**Subproblem definition wrong.** The most common DP bug is a recurrence that solves the wrong problem. "Maximum value of items in a knapsack of capacity `w`, considering items 1 through `i`" is a working definition. "Best value at this step" is not — it does not name the state space. When a DP solution is mysteriously wrong, the subproblem definition is usually the place to look.

**Off-by-one in base cases.** The empty-input base cases (`OPT(0)`, `OPT(0, j)`, `OPT(i, 0)`) are where most subtle bugs live. Test against small hand-computed examples before scaling.

The corrective heuristic: state the subproblem precisely, write the recurrence, identify the base case, and verify on a small example. If the subproblem space is polynomial, DP works. If it is exponential and not bitmask-compressible, switch tools.

## Cross-references

For the contrast with D&C (independent subproblems), see Chapter 7. For the contrast with greedy (matroid + exchange argument), see Chapter 6. For Bellman-Ford and Floyd-Warshall as graph DP, see Chapter 5 §5. For DP-based approximation algorithms (FPTAS for knapsack, etc.), see Chapter 11 §3. For randomized DP variants, see Chapter 12.

## Companion-page handoffs

Extended DP problem catalog with implementations; visualizations of DP table evolution for edit distance, LCS, and matrix-chain; recursion refresher; Hirschberg's algorithm walkthrough; bitmask DP tutorials; comparison harness — DP vs greedy vs D&C — on benchmark problems. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-8.

## What this chapter does not enable

This chapter does not give a procedure for solving every optimization problem with DP. The diagnostic move — does the problem have optimal substructure and overlapping subproblems? — is craft skill that develops with practice. The chapter also does not cover advanced DP topics — Knuth-Yao optimization, divide-and-conquer DP optimization, convex hull trick, Aliens trick — which live in competitive-programming literature. Stochastic DP (Bellman equations on Markov decision processes) is a different mathematical framework and lives in Vol. 2.

## Capability statement

You can now formulate a DP recurrence given a problem with optimal substructure and overlapping subproblems, choose between top-down memoization and bottom-up tabulation, apply space optimization when the table is the constraint, recognize the canonical DP catalog, and avoid the failure modes that come from forcing DP where greedy or D&C would suffice. The next time a problem looks like it might have repeating subproblems, the path to a working algorithm is in your hands.


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
