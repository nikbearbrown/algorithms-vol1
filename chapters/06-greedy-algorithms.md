# Chapter 6 — Greedy Algorithms

*The locally best choice, taken at every step, sometimes wins everything. The question is when.*

---

Here is a puzzle that looks almost too simple to be interesting.

You have a list of meetings. Each meeting has a start time and an end time. You want to attend as many meetings as possible, but you can only be in one place at a time — no two meetings you attend can overlap. Which meetings do you pick?

Think about it for a moment before reading on. The meetings are not weighted. You don't care which meetings you attend, only how many. You want the count to be as large as it can be.

Most people's first instinct is: start as early as possible, or pick the shortest meetings. Both instincts are wrong, and in a specific way that tells you something deep about what greedy algorithms actually are.

---

## What a greedy algorithm actually is

A greedy algorithm makes the locally best choice at each step and never looks back. That's the whole definition. No recalculation, no revisiting, no comparing alternatives. Commit and move on.

The question — the only question that matters — is whether making the locally best choice at each step guarantees the globally best outcome at the end. Sometimes it does. When it does, the greedy algorithm is provably correct, and it is usually the fastest correct algorithm available. When it doesn't, the greedy algorithm is a heuristic: it may produce a good solution, or a poor one, or the worst possible one, and without additional analysis you cannot tell which.

This chapter is about the difference.

---

## The meeting puzzle, resolved

Back to the meetings. What's the right greedy choice?

Not earliest start. Here is the counterexample. Suppose there is one meeting from 9 AM to 6 PM, and eight short meetings: 9-10, 10-11, 11-12, 12-1, 1-2, 2-3, 3-4, 4-5. Earliest start picks the 9-to-6 meeting and stops. Count: 1. But you could attend all eight short ones. Count: 8. The early-start heuristic fails badly because it commits to a long meeting that blocks everything else.

Not shortest duration. Three meetings: 8-10, 9-11, 10-12. Shortest duration picks 9-11, which overlaps with both others, leaving count 1. But 8-10 and 10-12 don't overlap: count 2. Picking the shortest meeting picked the wrong meeting.

<!-- → IMAGE: three-panel timeline diagram — panel 1 shows earliest-start picking the long 9–6 block and stopping (8 short meetings grayed out, count = 1); panel 2 shows shortest-duration picking the middle 9–11 block and blocking both neighbors (count = 1); panel 3 shows earliest-finish selecting all 8 short non-overlapping meetings (count = 8); each panel labels the strategy and the final count so the failure modes are visually immediate -->

The right choice is **earliest finish time**. Sort the meetings by when they end. Scan through them in that order. Take each meeting that doesn't conflict with the one you most recently selected. That's it.

Why does this work when the others don't? The answer is an exchange argument, and the exchange argument is the central idea in all of greedy algorithm analysis.

---

## The exchange argument

Here is how to prove that earliest-finish-time is optimal. Suppose there is an optimal solution — some set of meetings that achieves the maximum possible count. Call it *O*. If the greedy solution *G* equals *O*, we're done. If not, there must be some first point where they differ.

Let *g₁* be the first meeting the greedy algorithm selects — the one with the earliest finish time overall. Now look at *O*. Let *o₁* be the first meeting in *O* (sorted by start time). If *o₁* is not *g₁*, ask: what happens if we replace *o₁* with *g₁* in *O*?

Because *g₁* has the earliest finish time of any meeting, it finishes no later than *o₁*. And since the second meeting in *O* starts after *o₁* ends, it also starts after *g₁* ends. So the replacement doesn't create any new conflicts. The modified solution *O'* has the same number of meetings as *O* — it's still optimal — and it now agrees with the greedy solution on the first choice.

Apply this argument again to the second choice, then the third, and so on. Each exchange preserves optimality. After at most *n* exchanges, *O* has been transformed into *G* without ever reducing the count. Therefore *G* is optimal.

This is the template. Every provably correct greedy algorithm has an argument with this shape: any optimal solution can be transformed, one exchange at a time, into the greedy solution, without getting worse. If you can write that argument, the algorithm is correct. If you cannot, you don't yet know whether it's correct — and you should assume it isn't until you have a proof or a counterexample.

<!-- → INFOGRAPHIC: step-by-step exchange argument diagram for interval scheduling — row 1 shows optimal solution O with o₁ as its first interval; row 2 shows O' with o₁ replaced by g₁ (g₁ highlighted, same total count); row 3 shows the process repeating for o₂→g₂; final row shows G = O after all exchanges; the visual should make the "one swap at a time, count never decreases" logic scannable without reading the prose -->

---

## The lower-bound template

The exchange argument is one proof template. There's a second one, complementary to it: the lower bound argument.

Consider a variant of the meeting problem. Now you don't pick which meetings to attend — you have to schedule *all* of them, and you want to know the minimum number of rooms you need. Each room can only host one meeting at a time.

The greedy algorithm: sort all meetings by start time. For each meeting, assign it to any room that's currently free. If no room is free, open a new room. Time: *O(n log n)*.

How do you know this is optimal? There's a structural lower bound that makes it obvious. At any moment in time, if *k* meetings are happening simultaneously, you cannot possibly use fewer than *k* rooms — no matter what you do, no matter how clever the assignment. The peak simultaneous count is a hard lower bound on the room count.

Now ask: does the greedy algorithm ever do worse than this lower bound? No. The greedy algorithm opens a new room only when every existing room is occupied, which means the current meeting overlaps with every meeting in progress, which means the depth has just increased by one. The greedy algorithm adds exactly one room per unit of increase in depth, and never more. So its room count exactly equals the depth.

The proof structure: identify a structural lower bound, design a greedy algorithm that meets it, and prove the match. The greedy solution achieves what no algorithm can improve on.

<!-- → CHART: timeline chart of interval partitioning — horizontal bars showing meetings assigned to rooms (rows), with a vertical "depth" indicator at the peak-overlap moment; caption should highlight that the number of rows equals the peak depth, making the lower-bound argument visible as a geometric fact rather than an algebraic one -->

---

## Minimum spanning trees — three algorithms, one principle

Connect a set of cities by roads. You want to connect them all, building the minimum total road length. This is the minimum spanning tree problem.

Three greedy algorithms solve it, all optimally, all provably.

**Kruskal's algorithm.** Sort all edges by weight. Scan through them in ascending order. Add each edge to the growing tree unless it would create a cycle. (Use Union-Find from Chapter 3 to detect cycles efficiently.) The algorithm adds edges greedily by cost, stopping when the graph is connected.

**Prim's algorithm.** Start from any vertex. Maintain a priority queue of edges from the growing tree to vertices not yet in it. Repeatedly extract the cheapest such edge and add its endpoint. The tree grows outward from its current boundary, always taking the cheapest available extension.

**Borůvka's algorithm.** Look at each component independently. Every component finds its cheapest outgoing edge — the cheapest edge that crosses to another component. Merge all components along those edges simultaneously. Repeat until one component remains. The oldest of the three, due to Borůvka in 1926, and the most naturally parallelizable.

All three are correct, and their shared correctness rests on one property: the **cut property**. Take any cut of the graph — any partition of the vertices into two sets. The cheapest edge crossing the cut belongs to some minimum spanning tree.

The proof is an exchange argument. Suppose a minimum spanning tree *T* doesn't include the cheapest crossing edge *e*. Then *T* must cross the cut using some other edge *f*, which is heavier. Swap *f* for *e*: remove *f*, add *e*. The result is still a spanning tree (it still connects all vertices), and its total weight decreased by the weight of *f* minus the weight of *e*, which is positive. That contradicts *T* being minimal. So *T* must include *e*.

Each of the three algorithms is a different way of applying the cut property repeatedly. Kruskal applies it to the cuts defined by connected components. Prim applies it to the cut between the current tree and everything outside it. Borůvka applies it to every component simultaneously. The algebra is the same; the traversal order is different.

<!-- → IMAGE: three small graph diagrams side by side — Kruskal: edges sorted by weight with the next-cheapest non-cycle edge highlighted; Prim: current tree shaded, cheapest frontier edge highlighted; Borůvka: each component with its cheapest outgoing edge highlighted simultaneously; captions name the cut each algorithm is implicitly applying -->

---

## Huffman coding — greed at the bit level

Suppose you are transmitting a text over a channel. The text uses a small alphabet — say, six letters: A, B, C, D, E, F — with known frequencies. A encodes 45% of the text, B about 13%, C about 12%, D about 16%, E about 9%, F about 5%. A naïve encoding assigns each letter a 3-bit code (there are 6 symbols, 3 bits gives 8 possibilities). But why give the rare symbols the same code length as the frequent ones? Assign shorter codes to more frequent symbols and you transmit fewer bits overall.

The constraint: no code can be a prefix of another. If A is 0 and B is 01, then when you see 0 in the stream you can't tell whether it's A or the start of B. Prefix-free codes are those where no codeword is a prefix of any other.

Huffman's greedy algorithm builds the optimal prefix-free code. Represent each symbol as a leaf node with weight equal to its frequency. Repeatedly take the two nodes with the smallest weights, create a new internal node whose weight is their sum, and attach them as its children. Repeat until one tree remains. The codeword for each symbol is the path from root to that leaf, with left branches as 0 and right branches as 1.

Huffman's greedy algorithm builds the optimal prefix-free code. Represent each symbol as a leaf node with weight equal to its frequency. Repeatedly take the two nodes with the smallest weights, create a new internal node whose weight is their sum, and attach them as its children. Repeat until one tree remains. The codeword for each symbol is the path from root to that leaf, with left branches as 0 and right branches as 1.

<!-- → IMAGE: step-by-step Huffman tree construction for the six-symbol example (A=45, B=13, C=12, D=16, E=9, F=5) — four panels showing the state of the priority queue and partial tree after each merge step; final panel shows the complete tree with codewords labeled on edges; student should see how F and E (lowest frequencies) end up deepest and share a parent -->

Why does this work? The proof is another exchange argument, but it needs a preliminary observation: in any optimal prefix tree, the two symbols with the lowest frequencies have the longest codewords — they live at the greatest depth. If they didn't, you could swap them with whatever is deepest, decreasing the total weighted path length, contradicting optimality.

Given that observation: the two least-frequent symbols should be siblings (children of the same parent), as deep as possible. The greedy algorithm ensures this by combining them first. After combining, the internal node takes their place with their combined frequency, and the problem reduces to the same problem on a smaller alphabet. The exchange argument applies at each step: the greedy combination is always includable in an optimal tree.

Huffman coding is the foundation of real compression systems. DEFLATE, used in gzip and PNG, builds Huffman codes on top of Lempel-Ziv dictionary compression. JPEG uses Huffman coding in its entropy-coding stage. The greedy core is exactly what Huffman described; everything else is engineering around it.

---

## The matroid abstraction

There is a pattern running through all of these problems. Interval scheduling, MST, Huffman coding, fractional knapsack — they all share a structure that makes greedy provably optimal. The name for that structure is a **matroid**.

A matroid is a set system with an independence property. You have a ground set of elements (meetings, edges, symbols) and a collection of "independent sets." The independent sets satisfy three axioms: the empty set is independent; every subset of an independent set is independent; and if two independent sets have different sizes, some element of the larger one can be added to the smaller one and the result is still independent.

The punchline, which took mathematicians decades to formalize but once stated seems almost inevitable: **a greedy algorithm solves the maximum-weight independent set problem optimally if and only if the structure is a matroid**.

You don't need to know matroid theory to apply greedy correctly. But you do need to know that the structure matters. The question "is greedy optimal here?" is really the question "is this problem a matroid?" — or equivalently, "does the exchange argument go through?" Both are asking the same thing from different directions.

---

## When greedy is wrong

The same instinct that works so well in the problems above fails completely in others, and the failures are not subtle.

**Coin change with arbitrary denominations.** For US coins — 1¢, 5¢, 10¢, 25¢ — the greedy algorithm works: always take the largest coin that fits in the remaining amount. This is optimal. But change the denominations to 1¢, 6¢, and 10¢, and ask for 12¢. Greedy takes the 10¢ coin, then needs 2¢, but there's no 2¢ coin, so it takes two pennies: three coins total. The optimal solution is two 6¢ coins. Greedy produced the wrong answer, and it's not even close to right.

Why does greedy work for US coins and fail for this denomination set? The US coin system has the matroid property; the 1-6-10 system doesn't. There is no exchange argument for the arbitrary-denomination case. The correct tool is dynamic programming, which we'll reach in Chapter 8.

**The 0/1 knapsack.** You have a bag with a weight limit and a set of items, each with a weight and a value. Pick items to maximize total value without exceeding the weight limit. The greedy temptation: sort by value-per-unit-weight (value density) and take the highest-density items first.

This fails. Suppose the bag holds 50 pounds and you have three items: 10 lb for \$60 (density 6), 20 lb for \$100 (density 5), and 30 lb for \$120 (density 4). Greedy takes the 10-lb item, then the 20-lb item, filling 30 lb for \$160. But taking the 20-lb and 30-lb items fills 50 lb for \$220. Greedy missed the best answer.

Notice: if you could take fractions of items, greedy by value density is optimal. You'd take 10 lb at density 6, 20 lb at density 5, then 20 lb out of the 30-lb item at density 4, total value \$60 + \$100 + \$80 = \$240. The fractional version has the matroid structure. The 0/1 version (take it or leave it, no fractions) does not, and requires dynamic programming.

**Nearest-neighbor tour.** Find the shortest route that visits all cities exactly once and returns home. The greedy heuristic: from your current city, go to the nearest unvisited city. This is fast and often produces decent tours. But it can also produce tours that are far from optimal — on adversarially constructed inputs, the nearest-neighbor tour can be arbitrarily worse than optimal. There is no exchange argument. Nearest-neighbor is a heuristic, not an algorithm, in the technical sense.

---

## The approximation middle ground

Some problems sit between "greedy is provably optimal" and "greedy has no guarantees." They are NP-hard — no polynomial-time algorithm is known that always finds the exact optimum — but greedy gives a provably bounded approximation.

**Vertex cover.** Find the smallest set of vertices such that every edge in the graph has at least one endpoint in the set. Greedy strategy: repeatedly pick any uncovered edge and add *both* its endpoints to the cover. The resulting cover is at most twice the size of the optimal cover.

Why twice? The edges picked by the greedy (those that trigger each selection) form a matching — no two of them share an endpoint. Any vertex cover must include at least one endpoint of each matched edge. Since the matched edges are disjoint, the minimum vertex cover has size at least the number of matched edges. The greedy cover has exactly twice that many vertices. Therefore the greedy result is within a factor of 2 of optimal.

This is a proof of approximation ratio, not of optimality. It is rigorous. It guarantees something. But what it guarantees is a bound on how bad the answer can be, not a promise of optimality.

**Set cover.** Cover a universe of *n* elements using the minimum number of sets from a given collection. Greedy: repeatedly pick the set that covers the most uncovered elements. The resulting collection is at most *ln n* times the size of the minimum cover, where *ln n* is the natural logarithm.

This bound is known to be tight: no polynomial-time algorithm can do better than ln *n* approximation on set cover unless P = NP. The greedy algorithm is, in a precise mathematical sense, the best you can do efficiently.

The full treatment of approximation algorithms lives in Chapter 11. The point here is that greedy is not a binary choice between "provably optimal" and "useless." There is a middle register: provably bounded, which is often exactly what practice requires.

<!-- → INFOGRAPHIC: horizontal spectrum from left ("Provably optimal — exchange argument holds") through middle ("Provably bounded — approximation ratio known") to right ("Heuristic — no guarantee"); place labeled examples at each position: interval scheduling / MST / Huffman at left; vertex cover (2×) and set cover (ln n ×) in middle; nearest-neighbor TSP at right; makes the three-way taxonomy scannable -->

---

## The corrective heuristic

Here is the actual decision process, reduced to its core.

When you look at a new optimization problem and wonder whether greedy will work, ask one question: *can I write the exchange argument?* Not "does it feel right." Not "has it worked on similar problems." Not "this looks like the meeting scheduling problem." Write the argument. Suppose an optimal solution exists that doesn't follow the greedy strategy. Show that it can be modified to follow the greedy strategy without becoming worse. If that argument goes through, the algorithm is correct.

If the argument doesn't go through — if there's a step where you can't make the swap without potentially losing value — the algorithm is not provably correct. Find a small counterexample to confirm, then reach for dynamic programming.

The reverse mistake is also real: using dynamic programming on a problem that has a clean exchange argument and could be solved in *O(n log n)* instead of *O(n²)* or *O(n³)*. The interval scheduling problem solved by DP instead of greedy is the canonical example of a practitioner reaching for a heavier tool than necessary. DP is not more rigorous than greedy on problems where both are provably optimal — it is just slower.

Greedy is rigorous when the structure supports it. The structure is not a feeling; it is a proof. If you have the proof, the algorithm is exact. If you don't, you have a heuristic, and you should know which one you have.

---

## What you can do now that you couldn't before

You can look at an optimization problem and ask whether its structure supports a greedy algorithm. You can write the exchange argument for interval scheduling and MST, and you understand why the same argument fails for 0/1 knapsack. You can apply the lower-bound template to interval partitioning. You can implement Kruskal, Prim, Huffman, and earliest-finish-time interval scheduling, and you know why each one is correct rather than just knowing that it is.

The next chapter turns to dynamic programming — the tool for when optimal substructure is present but the greedy choice property is not, when the choices aren't independent and the decision at one step depends on what you'll do at the next. Where greedy commits and never looks back, dynamic programming looks everywhere before committing.

---

## Exercises

### Warm-up

**1.** Apply the earliest-finish-time algorithm to this set of intervals: [1,4], [3,5], [0,6], [5,7], [3,8], [5,9], [6,10], [8,11], [8,12], [2,13], [12,14]. List the intervals selected in order, and state the total count. Then verify by checking that no two selected intervals overlap.
*(Tests: mechanical application of the earliest-finish-time greedy.)*

**2.** Build the Huffman tree for these four symbols and frequencies: A=50, B=25, C=15, D=10. Show the merge steps. Then write out the codeword for each symbol and calculate the average code length in bits (weighted by frequency). Compare it to the naïve 2-bit fixed-length encoding.
*(Tests: Huffman construction and average-length calculation.)*

**3.** You have the following weighted edges in a graph: (A–B, 4), (A–C, 2), (B–C, 1), (B–D, 5), (C–D, 8), (C–E, 10), (D–E, 2). Run Kruskal's algorithm. List the edges added in order and state the total MST weight.
*(Tests: Kruskal's algorithm with cycle detection.)*

---

### Application

**4.** The exchange argument for earliest-finish-time relies on the fact that *g₁* finishes no later than *o₁*. Explain precisely why this property ensures the replacement *O'* (with *o₁* swapped for *g₁*) has no new conflicts with the remaining meetings in *O*. Then explain why the same argument would *fail* if you tried to run it for earliest-start-time — what breaks at the step where you attempt the replacement?
*(Tests: understanding the exchange argument as a logical structure, not just a result.)*

**5.** You have six meetings to schedule across as few rooms as possible: [9,10], [9,11], [10,12], [11,13], [12,14], [13,15].

   - (a) What is the schedule depth (maximum simultaneous meetings at any point)?
   - (b) Run the earliest-start-time greedy to assign rooms. Show the assignment.
   - (c) How many rooms does the greedy use? Is this optimal? Explain using the lower-bound argument.

*(Tests: interval partitioning, depth as lower bound, greedy achieves the bound.)*

**6.** For coin denominations {1, 5, 10, 25}, the greedy algorithm (take the largest coin that fits) is optimal. For denominations {1, 6, 10}, it is not.

   - (a) Show that greedy fails on {1, 6, 10} for amount 12 by exhibiting the greedy solution and the optimal solution.
   - (b) The greedy works for US denominations because each coin denomination "covers" the gap left by not using the next larger one efficiently. Explain informally why this structural property holds for {1, 5, 10, 25} but fails for {1, 6, 10}.
   - (c) What algorithm solves coin change correctly for arbitrary denominations?

*(Tests: recognizing where greedy fails, the matroid/non-matroid distinction in concrete terms.)*

---

### Synthesis

**7.** Prim's algorithm and Dijkstra's algorithm (from Chapter 5) look nearly identical in structure: both maintain a priority queue, both repeatedly extract the cheapest element, both grow a tree from a starting vertex. Yet Prim's finds the minimum spanning tree and Dijkstra's finds shortest paths, and they are solving different problems.

   Identify the precise difference in what each algorithm puts in the priority queue and what each "cheapest" means. Then explain why swapping the priority definitions — using Dijkstra's priority in Prim's setting, or vice versa — produces incorrect results.

*(Tests: understanding both algorithms deeply enough to distinguish them, not just recall their steps.)*

**8.** The vertex-cover greedy produces a 2-approximation. Construct a small graph (8 vertices or fewer) on which the greedy vertex cover is exactly twice the size of the optimal vertex cover. State the optimal cover and the greedy cover explicitly, and verify the factor of 2.

*(Tests: approximation ratio analysis made concrete; understanding when the bound is tight.)*

---

### Challenge

**9.** The chapter states: "a greedy algorithm solves the maximum-weight independent set problem optimally if and only if the structure is a matroid." The "if" direction (matroid → greedy is optimal) is the direction this chapter proves by exchange argument. The "only if" direction is the converse: if greedy is always optimal, the structure must be a matroid.

   Explain what it would mean for the "only if" direction to fail — describe a hypothetical structure that is *not* a matroid but on which greedy is nevertheless always optimal. Then explain why matroid theorists believe no such structure exists (you do not need to reproduce the full proof — the argument by counterexample construction is sufficient).

*(Tests: understanding the biconditional as a logical claim, not just memorizing the result; distinguishing the two proof directions.)*

**10.** Design a greedy algorithm for the following problem and either prove it is optimal via an exchange argument, or construct a small counterexample showing it fails.

   *Task scheduling with deadlines.* You have *n* unit-length tasks, each with a deadline (the latest time slot in which it can run) and a penalty (paid if the task misses its deadline). You want to minimize total penalty. You can run one task per time slot. Propose a greedy strategy, trace it on the instance: Task A (deadline 2, penalty 100), Task B (deadline 1, penalty 19), Task C (deadline 2, penalty 27), Task D (deadline 1, penalty 25), Task E (deadline 3, penalty 15). State whether your strategy is optimal and sketch why.

*(Tests: applying the exchange-argument framework to a novel problem; distinguishing between "feels right" and "provably right.")*

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
