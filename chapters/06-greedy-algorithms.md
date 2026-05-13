# Greedy Algorithms

## TL;DR

A greedy algorithm makes the locally best choice at each step and never reconsiders. Reach for this chapter when a problem has the structure of repeated independent choices and you suspect — or can prove — that local optimality compounds into global optimality. After consulting it, you can recognize when greedy is provably optimal (matroids and exchange arguments), when it gives a known approximation ratio, and when it is a heuristic with no guarantee at all.

## Recognition pattern

You face a sequence of decisions, each with a local cost or value, and you want a globally optimal outcome. The signal that greedy might apply: the problem has *optimal substructure* — an optimal solution contains optimal solutions to subproblems — and a *greedy choice property* — there exists a locally optimal choice that extends to a global optimum.

Three concrete signals worth checking. *Sortable input.* Many greedy algorithms work by sorting (intervals by end time, edges by weight, characters by frequency) and then scanning. If your problem admits a useful sort order, greedy is worth trying. *Matroid structure.* The problem can be cast as picking a maximum-weight independent set in a matroid; matroid theory guarantees greedy is optimal. *Exchange argument available.* You can argue that any optimal solution can be transformed into the greedy solution without losing value.

A signal that greedy is *not* the right tool: the problem has overlapping subproblems whose optimal solutions depend on each other. That is the dynamic-programming shape (Chapter 8). A signal that greedy is the *wrong* tool: a small example exists where local optima compound into a globally suboptimal result. The 0/1 knapsack with arbitrary weights and values is the canonical case where greedy fails and DP succeeds.

The misconception engaged in §8 is the inverse — the belief that greedy is sloppy and DP is the rigorous one. When greedy is provably optimal, it is fully rigorous, and it costs less.

## What you need to know first

This chapter assumes Big O (Chapter 2) and basic graph terminology (Chapter 5) for MST examples. Union-Find (Chapter 3 §6) is referenced in Kruskal's MST algorithm. For the contrast with dynamic programming, see Chapter 8. For approximation-ratio analysis of greedy algorithms applied to NP-hard problems, the full treatment is in Chapter 11; this chapter introduces vertex cover and set cover briefly.

## What greedy means

A greedy algorithm builds a solution by making the locally best choice at each step, never revisiting prior choices. The two properties a problem must have for a greedy algorithm to be provably optimal:

**Greedy choice property.** A globally optimal solution can be obtained by making a locally optimal (greedy) choice at each step. Equivalently: at each decision point, there exists an optimal solution that includes the greedy choice.

**Optimal substructure.** An optimal solution to the problem contains optimal solutions to its subproblems. After making a greedy choice, the remaining problem is a smaller instance whose optimal solution combined with the greedy choice gives the global optimum.

Without both properties, a greedy algorithm is a heuristic — it may produce a good solution, but no proof guarantees optimality. With both, it is provably optimal and usually the cheapest correct algorithm.

## Interval scheduling — the canonical proof template

The interval scheduling problem: given `n` intervals `[s₁, e₁], ..., [sₙ, eₙ]`, select the largest subset of pairwise non-overlapping intervals.

The optimal greedy choice is **earliest finish time**: sort intervals by end time, scan left to right, take each interval that does not overlap the most recently selected one. Time: `O(n log n)` for the sort plus `O(n)` for the scan. Optimal subset size: maximum.

Why earliest finish time? An *exchange argument*. Suppose an optimal solution `O` exists that does not include the earliest-finishing interval `g₁`. Let `o₁` be the first interval in `O` (sorted by start time). Replace `o₁` with `g₁`. The replacement does not overlap any other interval in `O` because `g₁` ends no later than `o₁` (by the greedy choice) and the next interval in `O` starts after `o₁` ends. So `O'` = `O` with `o₁` replaced by `g₁` is also optimal. Repeat the argument for the remaining intervals; the greedy solution is constructed exchange by exchange from `O` without ever reducing the count.

Why not earliest start time? Counterexample: one long interval `[0, 100]` and ten short ones `[1, 2], [3, 4], ..., [19, 20]`. Earliest start picks the long one and stops, count 1. Earliest finish picks all ten short ones, count 10. The earliest-start heuristic fails because committing to a long interval blocks many short ones.

Why not shortest interval? Counterexample: three intervals `[0, 10]`, `[9, 11]`, `[10, 20]`. Shortest picks `[9, 11]`, blocking the other two, count 1. Earliest finish picks `[0, 10]` then `[10, 20]`, count 2.

The exchange argument is the central proof template for greedy optimality. When you see one, the algorithm is rigorously correct.

## Interval partitioning — counting resources

A variant: schedule all intervals across the minimum number of resources (machines, classrooms, lecture halls), where each resource handles non-overlapping intervals.

Greedy: sort intervals by start time; assign each interval to any available resource (one whose last interval has finished); if none is available, create a new one. Time: `O(n log n)`.

The minimum number of resources equals the **depth** of the schedule — the maximum number of intervals overlapping at any point in time. This is a lower bound (you cannot do with fewer resources than the simultaneous count at peak), and the greedy algorithm achieves it.

The lower-bound argument is the second canonical template. Identify a structural lower bound, design a greedy algorithm that meets it, prove the match.

## Minimum spanning trees — three greedy algorithms, all optimal

A minimum spanning tree (MST) of a connected weighted undirected graph is a subset of edges that connects all vertices with minimum total weight. Three greedy algorithms, all proven optimal by different exchange arguments.

**Kruskal's algorithm.** Sort edges by weight ascending. For each edge in order, add it to the MST unless it creates a cycle (Union-Find detects cycles in `O(α(n))` amortized; see Chapter 3 §6). Time: `O(E log E)`. Wins on sparse graphs.

**Prim's algorithm.** Start from any vertex. Maintain a priority queue of edges from the tree to non-tree vertices. Repeatedly extract the cheapest edge whose other endpoint is not yet in the tree, add it. Time: `O(E log V)` with a binary heap (Chapter 3 §4). Wins on dense graphs.

**Borůvka's algorithm.** In each phase, every component finds its cheapest outgoing edge; merge components along those edges. Time: `O(E log V)`. The oldest of the three (Borůvka, 1926) [verify]; useful in parallel implementations because per-component work is independent.

The correctness of all three rests on the **cut property**: for any cut of the graph (a partition of vertices into two sets), the cheapest edge crossing the cut belongs to some MST. This is an exchange argument in graph form: if an MST excludes the cheapest cut-crossing edge, swap any heavier crossing edge for it; the result is still a spanning tree, with smaller total weight, contradicting MST-ness.

The MST problem fits a richer abstraction: **graphic matroid**. A matroid is a structure that generalizes "independent set" — in this case, a set of edges is independent if it forms a forest (no cycles). The fundamental matroid theorem: greedy is optimal on the maximum-weight independent set problem if and only if the structure is a matroid. Kruskal applied to the graphic matroid is the canonical example. Other matroids — uniform matroid, partition matroid, transversal matroid — yield greedy-optimal algorithms for their respective independent-set problems.

## Huffman coding — frequency-driven greed

Huffman coding builds an optimal prefix-free binary code for a set of symbols with given frequencies. The greedy step: repeatedly combine the two least-frequent nodes into a single internal node whose frequency is the sum, until one tree remains.

Time: `O(n log n)` with a priority queue (Chapter 3 §4). The output is an optimal prefix-free code in the sense of minimum expected code length given symbol frequencies.

Why combine the least-frequent nodes? The proof is again an exchange argument: in any optimal prefix tree, the two deepest leaves can be made siblings (depth and sibling structure can be swapped without changing total weighted depth), and swapping them with the two least-frequent symbols can only decrease (never increase) total cost.

Huffman is the foundation of practical compression schemes (DEFLATE in gzip and PNG, JPEG entropy coding) [verify], usually combined with other techniques (Lempel-Ziv dictionary compression in DEFLATE). The greedy core is the same; the wrapping varies.

## When greedy approximates rather than solves

When optimal substructure or the greedy choice property fails, greedy can still earn its place by giving a known approximation ratio. Two examples introduced here; the full ratio analysis lives in Chapter 11.

**Vertex cover (greedy 2-approximation).** Find a minimum set of vertices that covers every edge. Greedy strategy: repeatedly pick any uncovered edge, add both endpoints to the cover, remove all edges incident to either endpoint. The result is at most twice the optimal size. Worse than a more careful approximation but simple, fast, and acceptable when "within a factor of 2" is good enough.

**Set cover (greedy `ln n`-approximation).** Find a minimum subcollection of sets whose union is the universe. Greedy: repeatedly pick the set that covers the most uncovered elements. The result is at most `H_n ≈ ln n` times optimal, where `H_n` is the `n`-th harmonic number. Tight: this is known to be the best polynomial-time approximation possible unless P = NP (Feige 1998) [verify].

These are not provably optimal — they are provably bounded. The distinction matters and is the practitioner-relevant content of approximation algorithms (Chapter 11).

## Decision rules

| Problem signature | Greedy strategy |
| --- | --- |
| Maximum compatible intervals | Earliest finish time |
| Minimum number of resources | Earliest start time + assign to free resource |
| Minimum spanning tree, sparse graph | Kruskal |
| Minimum spanning tree, dense graph | Prim |
| Minimum spanning tree, parallel | Borůvka |
| Optimal prefix-free code | Huffman (least-frequent merge) |
| Maximum-weight independent set in a matroid | Generic greedy |
| Vertex cover (heuristic, 2-approximation) | Take both endpoints of any uncovered edge |
| Set cover (heuristic, `ln n`-approximation) | Take set covering the most uncovered elements |
| Coin change, canonical denominations (US, EUR) | Take largest denomination ≤ remaining |
| Coin change, arbitrary denominations | DP, not greedy (greedy fails) |
| 0/1 knapsack | DP, not greedy (greedy fails) |
| Fractional knapsack | Greedy by value-density |

## Worked example — meeting room scheduling

A 200-person organization runs roughly 800 meetings per day across 40 conference rooms [verify]. Each meeting has a known start and end time when it is booked. The question: given today's bookings, what is the minimum number of rooms needed?

This is interval partitioning. The greedy algorithm is earliest-start-time-with-resource-reuse: sort meetings by start time; for each meeting, assign it to any room whose previous meeting has ended; if none is free, open a new room. The number of rooms used equals the schedule depth — the maximum simultaneous meeting count at any moment.

Implementation. A min-heap of "next free time" per room, sized as needed. For each meeting, peek at the heap's minimum; if `min_free_time ≤ meeting.start`, reuse that room (pop, push meeting's end time). Otherwise add a new room (push meeting's end time). Time: `O(n log n)` for the sort plus `O(n log k)` for the heap operations, where `k` is the room count.

Result: the algorithm returns the optimal room count, and the assignment is constructive — you know which meeting goes in which room. The optimality is provable by the depth lower bound: the room count cannot be smaller than the simultaneous-meeting peak; the greedy algorithm achieves the peak.

Why earliest-start? The schedule is processed in time order; assigning the next meeting to a free room (or opening a new one if none is free) cannot waste rooms. Earliest-finish is the wrong order here: it can leave a room idle when a later-starting meeting could have used it.

The same problem with different constraints drifts toward heuristic territory. *If meetings have fixed room preferences* (the AV-equipped room, the executive boardroom), the problem becomes a constraint-satisfaction problem and greedy can fail. *If meetings can be rescheduled within a window*, the problem becomes a scheduling-with-flexibility problem, often NP-hard. *If meetings have priorities and rooms have capacities*, the problem becomes a matching or flow problem (Chapter 9). The same opening question can have very different correct answers depending on what is fixed and what is flexible.

The lesson: greedy with a clean exchange argument is a precise tool. As constraints grow, the tool's domain shrinks. Be specific about the constraints; the algorithm follows.

## Failure modes — when "greedy is sloppy" misleads

The misconception engaged: "Greedy algorithms are sloppy; DP is the rigorous one."

This is wrong on three counts.

**Greedy is rigorous when the matroid/exchange-argument structure holds.** Interval scheduling, MST, Huffman, fractional knapsack — all are provably optimal greedy algorithms. The proof is mathematical and complete. Calling them sloppy is a category error: they are exactly as rigorous as the DP solutions to problems like 0/1 knapsack or edit distance, and they are cheaper.

**DP is sometimes a hammer where greedy would do.** A practitioner who reaches for DP on every optimization problem will write `O(n²)` or `O(n³)` solutions where `O(n log n)` greedy suffices. The interval scheduling problem solved by DP is a classic over-engineering case.

**Greedy can fail invisibly.** This is the legitimate concern. When the matroid structure is absent, greedy may produce a solution that looks good but is suboptimal. Three concrete cases.

*Coin change with arbitrary denominations.* For US coins (1, 5, 10, 25), greedy works. For arbitrary denominations like (1, 6, 10), greedy fails: making 12 cents greedily uses 10 + 1 + 1 (three coins), but optimal is 6 + 6 (two coins). Greedy is the wrong tool; DP is correct.

*0/1 knapsack.* Pick a subset of items, each with a weight and value, maximizing value subject to a weight bound. Greedy by value-density (value/weight) fails: it can pick a high-density small item that blocks a low-density large item that fits and is more valuable. DP solves it in `O(nW)` time.

*Tour planning (traveling salesman).* Nearest-neighbor heuristic is `O(n²)` and easy to implement, but it can produce tours arbitrarily far from optimal in the worst case. TSP with the metric property has provably-bounded approximations (Chapter 11), but the simple greedy is not one of them.

The corrective heuristic: greedy is right when you can write the exchange argument. Not "feels right." Not "has worked on similar problems." Write the argument. If it goes through, the algorithm is correct and likely the cheapest. If it does not, reach for DP (Chapter 8) or approximation analysis (Chapter 11).

## Cross-references

For Union-Find used by Kruskal, see Chapter 3 §6. For heaps used by Prim, Huffman, and meeting-room scheduling, see Chapter 3 §4. For the contrast with dynamic programming, see Chapter 8. For approximation ratios of vertex cover, set cover, and other NP-hard problems, see Chapter 11. For Dijkstra's algorithm as a greedy shortest-path algorithm, see Chapter 5 §5. For randomized greedy, see Chapter 12.

## Companion-page handoffs

Extended catalog of greedy algorithms with proof patterns; matroid theory primer; implementations of Kruskal, Prim, Borůvka, Huffman, interval scheduling, and meeting-room scheduling in Python; counterexample gallery for non-matroid greedy failures; benchmarks comparing greedy against DP on borderline problems. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-6.

## What this chapter does not enable

This chapter does not give a procedure for inventing greedy algorithms for novel problems. The diagnostic move — does the problem fit a matroid? does an exchange argument go through? — is a craft skill that develops with practice. The chapter also does not cover advanced matroid topics (matroid intersection, matroid union, matroid optimization with side constraints), nor does it carry the full ratio analysis for vertex cover and set cover; those live in Chapter 11.

## Capability statement

You can now identify when a problem admits a provably optimal greedy algorithm via matroid structure or exchange argument; apply the canonical greedy algorithms (interval scheduling, interval partitioning, MST trio, Huffman); recognize when greedy gives an approximation rather than an exact answer; and avoid the trap of using greedy where DP is required. The next time a sequence of independent-looking choices arrives, you can write the exchange argument or know to switch tools.


---

## LLM Exercise — Chapter 6: Greedy Algorithms and the Exchange Argument

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** Canonical greedy algorithms plus an empirical exercise that constructs adversarial inputs to demonstrate when alternative "obvious" greedy strategies fail.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 6 task in the algorithms-by-bear-toolkit. The chapter's
intellectual spine is the exchange argument — proving that the locally
optimal choice extends to a globally optimal solution. The exercise
both implements greedy algorithms and constructs adversarial inputs
that defeat the "obvious wrong" greedy alternatives.

In `chapters/ch06_greedy/`:

1. `implementations.py`:

   - `interval_scheduling_earliest_finish(intervals)` — the provably
     optimal greedy: sort by finish time, take greedy. Plus
     `interval_scheduling_earliest_start(intervals)` and
     `interval_scheduling_shortest_duration(intervals)` — both wrong;
     we keep them to demonstrate failure.
   - `interval_partitioning(intervals)` — minimum-rooms scheduling
     via sort by start time and a min-heap of room-finish-times
     (uses your `MinHeap` from Chapter 3).
   - `huffman_coding(frequencies)` — returns a code-table mapping
     each symbol to its binary code, plus the resulting average
     code length.
   - `kruskal_mst` and `prim_mst` — already in Chapter 5; re-export
     from this chapter as well so the greedy framing is co-located
     with the analysis.
   - `fractional_knapsack(items, capacity)` — provably optimal
     greedy (sort by value/weight ratio).
   - `coin_change_greedy(coins, amount)` — greedy that works on
     canonical denominations and fails on non-canonical
     denominations.

2. `test_implementations.py` — correctness for each. For
   `coin_change_greedy`, include a non-canonical denomination test
   (`coins = [1, 3, 4]`, `amount = 6`) and assert greedy returns 3
   coins while the optimal is 2. The failure is the test.

3. `benchmarks.py` — the exchange-argument study. For interval
   scheduling, generate `N = 1000` adversarial inputs designed to
   break `earliest_start` and `shortest_duration` but not
   `earliest_finish`. Report:

   - The fraction of adversarial inputs on which each wrong greedy
     is suboptimal.
   - The average and worst-case suboptimality gap.

   Bonus: implement a tiny LP-based exact solver for interval
   scheduling (using `scipy.optimize.linprog` with integer rounding)
   to verify ground truth on a sample.

4. `README.md` — decision cards for each algorithm. "Surprising
   findings": the worst-case suboptimality you observed for each
   wrong greedy, and at least one analytic explanation of why
   `earliest_finish` survives the adversarial generator while the
   alternatives don't (this is the exchange argument made empirical).

Commit with `ch06: greedy algorithms with adversarial-input study`.
```

**What this produces:** Eight greedy algorithms (some provably optimal, some demonstrably wrong), adversarial-input generators that show the exchange-argument's truth in numbers, and decision cards. The "wrong" greedy implementations are the pedagogical payoff — failing on purpose is the lesson.

**How to adapt this prompt:**

- *For your own project:* If your work has a real scheduling problem (meeting rooms, machine scheduling, task allocation), replace the adversarial inputs with your historical data and report what each greedy would have done.
- *For ChatGPT / Gemini:* The adversarial-generation logic is the interesting part; ask either model to think about *why* `earliest_start` fails before writing the generator. The reasoning quality varies.
- *For Claude Code:* Native fit. Let it report the suboptimality numbers from its own run.
- *For a Claude Project:* Skip; code-heavy.

**Connection to previous chapters:** Imports `MinHeap` and `UnionFind` from Chapter 3; reuses `kruskal` and `prim` from Chapter 5.

**Preview of next chapter:** Chapter 7 implements divide-and-conquer — closest pair of points, Karatsuba multiplication, merge sort revisited — and verifies the master-theorem predictions against the empirical recurrences.
