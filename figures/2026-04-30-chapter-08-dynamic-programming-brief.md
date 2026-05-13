# Figure brief — Chapter 08: Dynamic Programming

## Recommended figures

### Figure 1 — DP table for edit distance
**Path:** `../images/chapter-8-edit-distance-table.jpg`
**Description:** 2D table for `A = "kitten"` and `B = "sitting"`. Each cell labeled with `OPT(i, j)`. Optimal path traced through the table from `(0,0)` to `(6,7)`. Operations annotated along the path (substitute, insert, etc.). Final answer (3) circled.
**Use:** referenced in §7 (Worked example).

### Figure 2 — Top-down vs bottom-up
**Path:** `../images/chapter-8-topdown-vs-bottomup.jpg`
**Description:** Side-by-side diagrams. Left: recursion tree with memoized cells highlighted (top-down). Right: filled table with arrows showing dependency direction (bottom-up). Same recurrence, two evaluation orders.
**Use:** referenced in §4 (Top-down vs bottom-up).

### Figure 3 — Space optimization for edit distance
**Path:** `../images/chapter-8-space-optimization.jpg`
**Description:** Three panels: (1) full `(m+1)×(n+1)` table; (2) two-row optimization with previous and current row; (3) Hirschberg's divide-and-conquer for `O(m+n)` space with traceback. Memory annotations.
**Use:** referenced in §5 (Space optimization).

### Figure 4 — Naive Fibonacci recursion vs memoized
**Path:** `../images/chapter-8-fibonacci-tree.jpg`
**Description:** Left: full `O(2^n)` recursion tree for `F(6)` showing repeated subproblems. Right: memoized version with each subproblem computed once.
**Use:** referenced in §1 (Recognition pattern) as the introductory example.

### Figure 5 — DP table evolution for 0/1 knapsack
**Path:** `../images/chapter-8-knapsack-table.jpg`
**Description:** `(n+1) × (W+1)` table for a small knapsack instance. Rows = items, columns = capacities. Cells filled in row-major order with arrows showing which neighbors each cell reads.
**Use:** referenced in §6 (Canonical DP problems).

## Inline figure-call markers
None inserted in current draft.
