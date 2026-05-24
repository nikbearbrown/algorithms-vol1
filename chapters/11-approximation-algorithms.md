# Chapter 11 — Approximation Algorithms

*What it means to be provably close to the right answer.*

---

Here is a problem that sounds straightforward until you try to solve it. A city water utility wants to place sensors in its distribution network so that every junction is monitored — contamination anywhere should be detectable before it reaches consumers. There are thousands of candidate sensor locations. Each location, if installed, covers a known set of downstream junctions. The utility wants the minimum number of sensors that together cover every junction.

This is the set cover problem. It is NP-hard. Exact algorithms take exponential time. The network has thousands of junctions and the utility needs an answer in minutes, not years.

The classical response to this situation is a heuristic: use engineering judgment, try a few configurations, pick the best one. Heuristics are reasonable, and for many problems they perform well in practice. But they offer no guarantee. The solution they find might be near optimal, or it might be twice as large as necessary, and there is no way to know which without solving the problem exactly — which you just admitted you cannot do.

There is a middle ground. An approximation algorithm runs in polynomial time and returns a solution whose quality is provably bounded relative to optimal. Not "usually good." *Provably bounded*, on every instance. That is the contract this chapter is about.

---

## The Contract

An approximation algorithm comes with a ratio — a number `α ≥ 1` — that defines the worst-case quality guarantee.

For a minimization problem, the guarantee says: for every input, the algorithm's solution costs at most `α · OPT`, where `OPT` is the optimal cost. For a maximization problem, the algorithm achieves at least `OPT / α`. In both cases, `α = 1` would mean exact; larger `α` means a looser guarantee. Smaller is better.

The ratio is a *worst-case* bound over all possible inputs. It tells you that no adversary, however cleverly they construct an input, can make the algorithm do worse than `α · OPT`. What it does not tell you is how the algorithm performs on *your specific* input. Your input might produce a solution much closer to optimal. Or it might hit the worst case exactly. The ratio alone cannot distinguish between these. This distinction — between the worst-case bound and the typical-case performance — is the most important thing to understand about approximation algorithms, and I will return to it.

![Diagram illustrating the approximation contract ](images/11-approximation-algorithms-fig-01.png)
*Figure 11.1 — Diagram illustrating the approximation contract *

---

## A Hierarchy of Guarantees

Not all approximation guarantees are equal. Four classes, in increasing strength.

A **constant-factor approximation** gives a ratio that is a fixed constant independent of input size. Vertex cover admits a 2-approximation. Metric TSP admits a 1.5-approximation (Christofides, 1976). These algorithms are useful when "within a constant factor of optimal" is acceptable — which, for many engineering problems, it is.

A **PTAS** — polynomial-time approximation scheme — goes further. For any `ε > 0` you choose, the algorithm produces a `(1 + ε)`-approximation in time polynomial in the input size `n`. The tradeoff is that the running time may depend badly on `ε`: it might be `n^(1/ε)`, which for `ε = 0.05` and `n = 1000` is `1000^20`. That is an astronomically large number. A PTAS is polynomial for each fixed `ε`, but if `ε` needs to be small, the polynomial may be impractical.

An **FPTAS** — fully polynomial-time approximation scheme — removes the bad dependence on `ε`. It runs in time polynomial in both `n` and `1/ε`. The 0/1 knapsack problem admits an FPTAS via a technique I'll describe. This is the strongest class of approximation guarantee: you can make `ε` as small as you like, say `ε = 0.001` for solutions within 0.1% of optimal, and the running time scales gracefully.

A **logarithmic-factor approximation** has a ratio that grows slowly with input size. Set cover admits an `H_n ≈ ln n` approximation via a simple greedy algorithm, where `H_n = 1 + 1/2 + 1/3 + ... + 1/n`. For `n = 10,000` this is roughly 9.2 — the greedy solution uses at most about 9 times as many sets as the optimal. The ratio grows, but slowly.

Some NP-hard problems cannot be approximated within any constant factor unless P = NP. Maximum clique cannot be approximated within `n^(1−ε)` for any `ε > 0` under standard complexity assumptions. For these problems, no polynomial-time algorithm with a useful guarantee exists. The boundary between approximable and inapproximable is itself a deep mathematical structure — the PCP theorem is the machinery that produces most inapproximability results — but for a practitioner, the relevant fact is simpler: check the literature before investing in an approximation algorithm that cannot exist.

| guarantee form | running time dependence on ε | canonical example | what makes it stronger than the row above |
| --- | --- | --- | --- |
| constant-factor, PTAS, FPTAS, logarithmic-factor, inapproximable | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | A specific, evidence-linked version that readers can verify. |
| columns: guarantee form, running time dependence on ε, canonical example, what makes it stronger than the row above | Use the chapter example as the concrete test case. | Use the chapter example as the concrete test case. | A specific, evidence-linked version that readers can verify. |
| student should be able to classify a new guarantee they encounter by reading down the "guarantee form" column | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | A specific, evidence-linked version that readers can verify. |

---

## Three Design Techniques

Most approximation algorithms are built from one of three templates. Recognizing the template tells you both how the algorithm works and why the ratio holds.

**Greedy with a charging argument.** Make locally optimal choices and bound the ratio by arguing that every unit of "extra" cost in the greedy solution can be "charged" to a unit of cost in the optimal solution. Set cover via greedy is the canonical example. Vertex cover via maximal matching is another. The greedy algorithm runs fast; the analysis establishes the ratio by a careful accounting argument.

**LP relaxation and rounding.** Formulate the optimization problem as an integer program — a linear program where variables must take integer values. Integer programs are NP-hard in general. Relax the integrality constraint, allowing variables to take fractional values. Now the problem is a linear program, solvable in polynomial time (Chapter 13). Take the fractional solution and round it to integers by some scheme. The rounding scheme determines the approximation ratio, and the LP optimum provides the lower bound you compare against. Many approximation algorithms are LP-rounding underneath, even when they're described in combinatorial terms.

**DP rounding for FPTAS.** When a problem has a pseudo-polynomial DP solution — one whose running time is polynomial in the numeric values of the inputs — rounding the input values to a coarser precision produces a polynomial-time algorithm with a `(1 + ε)` guarantee. The trick: round each value down to the nearest multiple of some threshold Δ that depends on `ε` and the optimal value. The rounded problem has fewer distinct values, the DP table shrinks, and the error introduced by rounding is bounded by a function of `ε`. The 0/1 knapsack FPTAS works exactly this way.

![Three-template diagram for approximation algorithm design ](images/11-approximation-algorithms-fig-02.png)
*Figure 11.2 — Three-template diagram for approximation algorithm design *

---

## The Four Canonical Problems

**Vertex cover.** A vertex cover of a graph is a set of vertices such that every edge has at least one endpoint in the set. Finding the minimum vertex cover is NP-hard.

The 2-approximation: find any maximal matching — a set of edges such that no two share an endpoint, and no more edges can be added without violating this. Take all the endpoints of the matched edges as your cover. This is valid: every unmatched edge has both endpoints in the matching, so every edge is covered. And the size is at most 2 · OPT: the optimal cover must include at least one endpoint of every matched edge, and the matched edges don't share endpoints, so the optimal needs at least `|M|` vertices — but the algorithm outputs `2|M|`. Ratio: 2.

The algorithm is fast, clean, and the bound is believed to be tight — vertex cover is hard to approximate below 2 under a complexity assumption stronger than P ≠ NP. The 2-approximation is essentially the best polynomial-time algorithm known.

**Set cover.** A universe of elements, a collection of subsets. Find the minimum number of subsets whose union covers the universe. Finding the minimum set cover is NP-hard.

The greedy approximation: repeatedly pick the subset covering the most uncovered elements, until everything is covered. Time: `O(|U| · m)` where `m` is the number of candidate subsets.

Why does this give an `H_n`-approximation? Here is the argument. Label each element with the cost of the iteration in which it got covered: if the greedy algorithm covers `k` new elements in a step where it picks a set of size `s`, charge each new element `1/k` of the cost of that set. The element that gets covered when there are `t` elements remaining is charged at most `1/t` (since the greedy set covers at least one element, and we could have covered this one by picking its optimal set). Summing over all elements: the total charge is at most `H_n · OPT`. This argument has the flavor of amortized analysis — distributing a cost across the elements it benefits.

The ratio `H_n ≈ ln n` is essentially optimal: Feige showed in 1998 that set cover cannot be approximated within `(1 − o(1)) ln n` in polynomial time unless NP ⊆ DTIME(`n^(log log n)`), which is believed to be false.

**Metric TSP.** The traveling salesman problem asks for the minimum-cost tour visiting every city exactly once. General TSP is NP-hard to approximate within any constant unless P = NP — the problem is in the same inapproximability class as clique. But when the distances satisfy the triangle inequality (the metric condition: going directly from A to C is never more expensive than going A → B → C), a 1.5-approximation exists.

Christofides' algorithm (1976) works as follows. Compute the minimum spanning tree `T` on the cities. Identify the set `O` of vertices with odd degree in `T` — there are an even number of them. Find a minimum-weight perfect matching on `O`. Add the matching edges to `T` to get a multigraph where every vertex has even degree. Find an Eulerian circuit (a tour that traverses every edge exactly once). Shortcut the Eulerian circuit: when revisiting a city already visited, skip it and go directly to the next unvisited city. The shortcut is valid because the triangle inequality ensures skipping a city doesn't increase the cost.

Why 1.5? The MST costs at most `OPT` (the optimal tour is a Hamiltonian cycle, which is a spanning tree plus one edge). The minimum matching on `O` costs at most `OPT/2` (this takes more work to prove, using the fact that the odd-degree vertices can be paired in two ways by traversing the optimal tour, and one of those ways costs at most half the tour). Total: at most `1.5 · OPT`.

![Step-by-step Christofides walkthrough on a small 6-city metric](images/11-approximation-algorithms-fig-03.png)
*Figure 11.3 — Step-by-step Christofides walkthrough on a small 6-city metric*

Christofides' algorithm held the best-known ratio for metric TSP for over forty years. A 2020 result by Karlin, Klein, and Oveis Gharan broke this for general metrics (to something slightly below 1.5), but the improvement is not yet practical.

**Knapsack FPTAS.** You have `n` items with weights and values; pick a subset of maximum value subject to a weight constraint. The exact pseudo-polynomial DP (Chapter 8) runs in `O(nW)` time — fine for moderate `W`, slow for large `W`.

The FPTAS: scale and round each item's value down to the nearest multiple of Δ = `ε · v_max / n`, where `v_max` is the maximum item value and `ε` is the desired precision. Run the DP on the rounded values. The number of distinct rounded values is at most `n / ε`, so the DP table has `O(n² / ε)` entries and the algorithm runs in `O(n³ / ε)` — polynomial in both `n` and `1/ε`.

The rounding introduces error. Each item's value is reduced by at most Δ. The total error is at most `n · Δ = ε · v_max`. The optimal value is at most `n · v_max`. So the solution value is at least `OPT − ε · v_max ≥ (1 − ε) · OPT`. Ratio: `1/(1 − ε)`, which for small `ε` is approximately `1 + ε`.

This construction — round to reduce the DP's state space, bound the rounding error — is the general recipe for turning pseudo-polynomial DP into FPTAS whenever the structure permits.

![Before-and-after diagram for knapsack FPTAS value rounding ](images/11-approximation-algorithms-fig-04.png)
*Figure 11.4 — Before-and-after diagram for knapsack FPTAS value rounding *

---

## When the Ratio Misleads

This is the content that matters most for applying these algorithms in practice. Four misreadings, each common.

**Misreading 1: "The ratio tells me how good my answer is."** It does not. The ratio tells you the worst case over all possible inputs. Your specific input might produce a solution much closer to optimal — empirically, the vertex cover 2-approximation typically outputs solutions between 1.05× and 1.3× optimal on random graphs, nowhere near 2×. Or your input might be adversarially constructed to hit the ratio exactly. The ratio is a guarantee about the contract, not a prediction about the outcome. To know how the algorithm performs on your data, you have to benchmark on your data.

**Misreading 2: "A smaller ratio means a better algorithm in practice."** A 1.5-approximation for TSP sounds better than a 2-approximation. On your specific instances, the 2-approximation might produce smaller tours, run ten times faster, and require one-tenth the memory. The ratio ranks algorithms on the worst case; it says nothing about average-case performance, constant factors, or implementation complexity. Two algorithms with the same ratio can perform very differently in practice.

**Misreading 3: "A PTAS is always practical."** A PTAS runs in time polynomial in `n` for any fixed `ε`. For `ε = 0.01` and a running time of `n^(1/ε) = n^100`, the algorithm is technically polynomial and practically useless. This is why the FPTAS distinction matters — it is the class of schemes where the running time is polynomial in both `n` and `1/ε`, making tight approximation genuinely achievable.

**Misreading 4: "The bound applies without checking the assumptions."** Christofides' 1.5-approximation requires the triangle inequality. If your distances violate it — as they might with asymmetric costs, or with time-dependent travel times, or with any non-metric structure — the algorithm has no ratio guarantee. LP-rounding bounds depend on the integrality gap of the relaxation. Greedy set-cover bounds depend on being able to compute coverage cheaply. Each algorithm's ratio comes attached to a model assumption. Violating the assumption voids the contract silently, without the algorithm noticing.

The corrective practice: when applying an approximation algorithm, check its assumptions against your problem instance. Benchmark on representative data. Report the typical-case ratio alongside the theoretical worst-case bound. The bound is the floor of the guarantee; the benchmark is the prediction about your specific workload.

![Empirical ratio distribution for vertex cover 2-approximation on](images/11-approximation-algorithms-fig-05.png)
*Figure 11.5 — Empirical ratio distribution for vertex cover 2-approximation on*

---

## Inapproximability: Where the Wall Is

Some NP-hard problems cannot be approximated within any useful factor in polynomial time, assuming standard complexity conjectures. Knowing where this wall is matters for deciding whether to look for an approximation algorithm or give up on guarantees entirely.

Maximum clique — finding the largest complete subgraph — is NP-hard to approximate within `n^(1−ε)` for any `ε > 0`. For a graph on 1000 vertices, this means no polynomial-time algorithm is known to give a solution within a factor of 1000^0.99 ≈ 900 of optimal. The problem is essentially as hard to approximate as to solve.

Graph coloring — finding the minimum number of colors to color vertices so no adjacent pair shares a color — is NP-hard to approximate within `n^(1−ε)` as well. MAX-3SAT — maximizing the fraction of satisfied clauses in a 3-CNF formula — cannot be approximated above 7/8 in polynomial time unless P = NP (and 7/8 is achievable by a trivial random assignment).

The machinery behind these results is the PCP theorem, which characterizes NP in terms of probabilistically checkable proofs. The theorem implies that the gap between satisfiable and unsatisfiable instances is preserved under polynomial reductions in a strong sense, making hardness of approximation proofs possible. For a practitioner, the relevant implication is: before designing an approximation algorithm, check whether the problem is inapproximable. If it is, the effort should go into heuristics, special cases, or exact algorithms for restricted inputs.

---

## Back to the Sensor Network

The water utility's sensor placement problem is set cover. The greedy algorithm runs in seconds on 10,000 junctions. The ratio bound is `H_{10,000} ≈ 9.2`. If optimal uses 50 sensors, greedy guarantees at most ~460 — a bound that sounds alarming.

But that bound is the worst case over all possible set-cover instances, including adversarial ones. Water networks are geographic: sensors cover downstream junctions, coverage regions have spatial structure, and realistic contamination scenarios tend to cluster. On instances with this structure, greedy typically runs at 1.1× to 1.3× optimal in practice. The utility should verify this on its specific network and scenario set — but the reasonable expectation is that the 9.2× worst case will never materialize.

The utility could also run an ILP solver for the exact optimum. Modern solvers handle set cover instances of this size in minutes. The standard production approach: run the ILP for the exact solution on the real network, run greedy during model development when you need fast iteration, and know that greedy's solution, whatever it is, is within `9.2 · OPT` in the worst case. Both algorithms serve a purpose; neither is universally better.

---

## What You Can Do Now

Given an NP-hard optimization problem, you can identify whether a polynomial-time approximation algorithm exists with a known ratio. You can distinguish constant-factor, PTAS, FPTAS, and inapproximable cases. You can recognize the three design templates — greedy with a charging argument, LP relaxation and rounding, DP rounding for FPTAS — and apply the canonical algorithms for vertex cover, set cover, metric TSP, and knapsack. And you can interpret the ratio correctly: as a worst-case contract over all inputs, not a prediction about your specific instance.

The boundary between approximable and inapproximable, between PTAS and FPTAS, is not arbitrary. It reflects deep structure in the problem class. Chapter 10 establishes the NP-hardness of the problems here. Chapter 13 develops LP relaxation as a general tool. Chapter 12 shows how randomization — particularly randomized rounding — extends the LP-rounding technique to problems where deterministic rounding loses too much.

---

## Exercises

### Warm-Up

**1.** For each of the following guarantees, classify it as constant-factor approximation, PTAS, FPTAS, logarithmic-factor approximation, or inapproximable (under standard assumptions). Justify each answer in one sentence.

- An algorithm for vertex cover that always returns a solution of size at most `2 · OPT`
- An algorithm for set cover that runs in `O(|U| · m)` and returns a solution of size at most `(ln n + 1) · OPT`
- An algorithm for knapsack that, given any `ε > 0`, returns a solution of value at least `(1 − ε) · OPT` in time `O(n³ / ε)`
- A claimed algorithm for maximum clique that returns a clique of size at least `OPT / 5` in polynomial time
- An algorithm for Euclidean TSP that, for any `ε > 0`, returns a tour of length at most `(1 + ε) · OPT` in time `n^(O(1/ε))`

*Tests: classifying approximation guarantees; recognizing the FPTAS vs. PTAS distinction; identifying a claim that violates inapproximability.*

**2.** State the approximation ratio for the vertex cover maximal-matching algorithm and prove it is correct. Your proof must: (a) argue the output is a valid vertex cover, (b) identify a lower bound on OPT derived from the matching, and (c) show the algorithm's output size is at most twice that lower bound.

*Tests: executing the charging/lower-bound style of approximation ratio proof; understanding the matched-edge lower bound argument.*

**3.** An algorithm for bin packing (First-Fit Decreasing) guarantees a solution using at most `(11/9) · OPT + 6/9` bins. A colleague has implemented the algorithm and benchmarked it on their data: it typically uses `1.02 · OPT` bins.

(a) What does the ratio guarantee in the event of a truly adversarial input?

(b) The colleague wants to use the algorithm in production and says "the ratio is 11/9, so we're always at most 22% above optimal." Is this interpretation correct? Why or why not?

(c) What is the minimum information you would want before trusting the algorithm for a specific production workload?

*Tests: interpreting an approximation ratio correctly; distinguishing worst-case guarantee from typical-case performance; specifying the benchmarking requirement.*

---

### Application

**4.** Apply the set cover greedy algorithm to the following instance. Universe `U = {1, 2, 3, 4, 5, 6}`. Candidate sets: `S₁ = {1, 2, 3, 4}`, `S₂ = {2, 4, 5}`, `S₃ = {1, 3, 6}`, `S₄ = {5, 6}`.

(a) Trace the greedy algorithm step by step. At each step, name the set chosen and the elements newly covered.

(b) What is the greedy solution size? What is the optimal solution size? Compute the empirical ratio `greedy / OPT` for this instance.

(c) The ratio bound is `H_6 ≈ 2.45`. The empirical ratio you computed is probably smaller. What does this tell you about the tightness of the `H_n` bound?

*Tests: executing the greedy set cover algorithm; comparing empirical and theoretical ratio; reasoning about bound tightness.*

**5.** The knapsack FPTAS scales item values by Δ = `ε · v_max / n`. Consider a knapsack instance with `n = 4` items and values `{10, 6, 5, 3}`, and `ε = 0.5`.

(a) Compute Δ. Round each item value down to the nearest multiple of Δ to get the rounded values.

(b) The exact DP on the rounded values gives an optimal rounded solution. Suppose that solution has value `V_rounded`. What is the worst-case gap between `V_rounded` and the true optimal `OPT`? Express it in terms of the original values.

(c) For `ε = 0.1` on the same instance, recompute Δ and the rounded values. How does the table size (number of distinct values) change? This is the FPTAS tradeoff in action: name it explicitly.

*Tests: executing the FPTAS rounding step; bounding the rounding error; seeing the ε–table size tradeoff concretely.*

**6.** An approximation algorithm for metric TSP (not Christofides — a simpler 2-approximation via MST doubling) works as follows: compute the MST, double every edge to make all vertices even-degree, find an Eulerian circuit, shortcut. Argue this gives a 2-approximation.

Your argument should: (a) bound the MST cost in terms of OPT, (b) bound the doubled-edge cost in terms of OPT, (c) argue the shortcutting step doesn't increase cost (name the property required), and (d) add the bounds to get the ratio.

Then explain what Christofides improves and why the matching step produces a tighter bound than simply doubling all edges.

*Tests: constructing a ratio proof from first principles; understanding why Christofides improves on MST-doubling; applying the triangle inequality to shortcutting.*

---

### Synthesis

**7.** A network operator wants to deploy monitoring agents across a data center network. Nodes are servers; edges are network links. An agent on a server monitors all links incident to it (a vertex cover). The operator wants the minimum number of agents such that every link is monitored.

(a) Map this to vertex cover. What is the universe? What is a feasible solution?

(b) Apply the 2-approximation. The network has 1,000 servers and 4,000 links. What is the worst-case guarantee on number of agents?

(c) A security argument requires that the solution be within 10% of optimal. Can the 2-approximation satisfy this requirement? If not, is there any polynomial-time algorithm that can? Justify with reference to inapproximability.

(d) The operator's actual network is a near-planar graph (data centers are physically structured). Does the inapproximability result apply to planar graphs? What algorithmic options improve when graph structure is restricted?

*Tests: reducing an application problem to vertex cover; applying the approximation in context; reasoning about inapproximability vs. special-case tractability.*

**8.** Compare the greedy set cover algorithm and LP-rounding for set cover. Both achieve an `O(log n)` approximation ratio.

(a) Describe the LP-rounding approach for set cover in enough detail to explain where the `O(log n)` ratio comes from. (Hint: the LP assigns fractional values to sets; the rounding scheme samples each set with probability proportional to its fractional value and repeats until covered.)

(b) What is the advantage of LP-rounding over greedy? What is the disadvantage?

(c) Both algorithms produce different solutions on the same instance. For the water utility's sensor placement (10,000 junctions, 1,000 candidate sensors), which would you choose for production and why? Your answer should address running time, quality guarantee, and implementation complexity.

*Tests: comparing two approximation algorithms with the same ratio but different designs; making a practical engineering choice with explicit tradeoffs.*

---

### Challenge

**9.** The PCP theorem states (informally) that NP = PCP(log n, 1) — every NP language has a probabilistically checkable proof that can be verified using O(log n) random bits and reading a constant number of bits from the proof. This seemingly technical characterization is the foundation of most inapproximability results.

(a) Explain informally how the PCP theorem implies that MAX-3SAT cannot be approximated above 7/8 in polynomial time unless P = NP. (You do not need to prove the theorem — explain the reduction idea at the conceptual level.)

(b) MAX-3SAT has a trivial 7/8-approximation: assign each variable uniformly at random. Each clause is satisfied with probability 7/8. Explain why this matches the inapproximability bound and what it implies about the hardness landscape of MAX-3SAT compared to, say, vertex cover.

(c) Set cover has an inapproximability threshold of `(1 − o(1)) ln n` and the greedy algorithm achieves `H_n ≈ ln n`. This means greedy is essentially optimal for polynomial-time set cover. Explain why this is both a positive result (greedy is near-optimal) and a negative result (no polynomial algorithm can do significantly better), and what the combination tells you about investing in sophisticated set-cover algorithms.

*Tests: connecting the PCP theorem to specific inapproximability results; reasoning about tight inapproximability thresholds; evaluating when algorithm improvement is and isn't worth pursuing.*

**10.** The FPTAS for knapsack relies on the existence of a pseudo-polynomial DP. Not every NP-hard problem has one.

(a) State precisely what "pseudo-polynomial" means. Why does knapsack have a pseudo-polynomial algorithm while, say, graph coloring does not?

(b) The bin packing problem has an asymptotic PTAS but not an FPTAS (under standard assumptions). What structural property of bin packing prevents the DP-rounding FPTAS construction from applying?

(c) A colleague proposes an FPTAS for the partition problem (given a set of integers, can they be split into two equal-sum subsets?). Is this plausible? Partition is NP-complete. Sketch the DP-rounding argument if you believe it's achievable, or explain the obstacle if you don't.

*Tests: reasoning about when pseudo-polynomial DPs exist and when they don't; understanding what structural property enables the FPTAS construction; applying the FPTAS template to a new problem.*

---

## LLM Exercise — Chapter 11: Approximation Ratios Empirically

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** Approximation algorithms with known ratios, plus an empirical study answering: does the worst-case ratio actually show up in practice, or is real-world performance much better?

**Tool:** Claude Code.

**The prompt:**

```
Chapter 11 task in the algorithms-by-bear-toolkit. Bear's most
practitioner-discriminating content is the section on when ratios
mislead — the worst-case bound holds for every input, but on your
specific input the algorithm may do much better or hit the worst case
exactly. The exercise measures the gap between worst-case and typical
performance.

In `chapters/ch11_approximation/`:

1. `implementations.py`:

   - `vertex_cover_2_approx(graph)` — the 2-approximation via
     maximal matching: repeatedly pick any edge, add both endpoints
     to the cover, remove both from the graph. Returns a cover
     guaranteed within 2× optimal.
   - `set_cover_greedy(universe, sets)` — the canonical greedy:
     repeatedly pick the set covering the most uncovered elements.
     Returns a cover within `H(n) ≈ ln(n) + 1` of optimal.
   - `tsp_christofides(distance_matrix)` — the 1.5-approximation
     for metric TSP via minimum spanning tree, perfect matching on
     odd-degree vertices, and Euler-tour shortcutting. (This is
     ambitious; if the perfect-matching step is too heavy,
     substitute the simpler MST-based 2-approximation and note the
     trade.)
   - `knapsack_fptas(weights, values, capacity, epsilon)` — the
     Fully Polynomial-Time Approximation Scheme via value rescaling.
     Returns a solution within (1-epsilon) of optimal.

2. `test_implementations.py` — each algorithm returns a valid
   solution; for small instances, compute the optimal via brute
   force and confirm the ratio bound holds.

3. `benchmarks.py` — the ratio-tightness study. For each algorithm,
   generate two kinds of input:

   - **Worst-case adversarial inputs** designed to make the
     approximation hit (or approach) its ratio bound.
   - **Random inputs** sampled from a realistic distribution.

   For each input: run the approximation, then run an exact
   solver (use `pulp` ILP for vertex cover and set cover; use
   `networkx` for held-karp TSP up to n = 15; use the DP exact for
   knapsack up to capacity 1000). Report the empirical ratio
   `approx / optimal`.

   Plot empirical ratio distributions for adversarial vs. random
   inputs. Report: (a) the worst empirical ratio observed; (b) the
   median empirical ratio on random inputs; (c) the gap between
   them — this is the "how misleading is the worst-case ratio in
   practice" answer.

4. `README.md` — decision cards. "Surprising findings": for each
   algorithm, the median random-input ratio compared to the
   worst-case bound. If your set cover hit close to ln(n) on random
   instances, that's notable (it usually doesn't); if vertex cover
   beat 1.5× on average, also notable.

Commit with `ch11: approximation algorithms with empirical ratio
study`.
```

**What this produces:** Four approximation algorithms, a worst-case/random ratio comparison, decision cards, and a strong "the bound is loose in practice" finding (or the rarer and more interesting "the bound is tight" finding).

**How to adapt this prompt:**

- *For your own project:* Replace the synthetic instances with your domain's data (real delivery routes for TSP, real ad-placement instances for set cover).
- *For ChatGPT / Gemini:* Christofides is ambitious; if either model produces buggy perfect-matching code, fall back to the MST-based 2-approximation cleanly.
- *For Claude Code:* Native fit. Have it install `pulp` to get exact ILP solutions on small instances.
- *For a Claude Project:* Skip.

**Connection to previous chapters:** Uses `Graph` from Chapter 5; the greedy set-cover algorithm is a Chapter 6 callback; the LP-rounding pattern previews Chapter 13.

**Preview of next chapter:** Chapter 12 introduces randomization — Karger's min-cut algorithm, randomized quickselect, locality-sensitive hashing — and asks you to *measure* the concentration of randomized algorithms, which is the property that makes "expected behavior" trustworthy.

## Prompts

Use these prompts with Claude to generate interactive D3 v7 versions of the
figures in this chapter. Each produces a standalone HTML file you can open
in a browser and modify freely.

**Prerequisites:** Load `brutalist/CLAUDE.md` and `brutalist/DESIGN.md` into
your Claude project context before using these prompts. They define the stack,
naming conventions, color system, and typography the figures use.

---

### Figure 11.1 — Diagram illustrating the approximation contract 

Create a standalone D3 v7 HTML file for Figure Diagram illustrating the approximation contract . Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: two-panel diagram illustrating the approximation contract — left panel: number line from OPT to α·OPT, with "algorithm's solution lands somewhere in here" shaded and a question mark indicating the algorithm's actual output position within the range; right panel: two sample instances, one where the algorithm hits near OPT, one where it hits near α·OPT — student should see that the ratio defines the ceiling, not the landing spot. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/11-approximation-algorithms-fig-01.html`

---

### Figure 11.2 — Three-template diagram for approximation algorithm design 

Create a standalone D3 v7 HTML file for Figure Three-template diagram for approximation algorithm design . Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: three-template diagram for approximation algorithm design — three horizontal lanes labeled "Greedy + charging", "LP relax + round", "DP round for FPTAS"; each lane contains: (1) the key idea in one sentence, (2) what provides the lower bound for the ratio proof, (3) canonical example problem; arrows connect the canonical example to its ratio; student should be able to match a new algorithm they read about to its design template. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/11-approximation-algorithms-fig-02.html`

---

### Figure 11.3 — Step-by-step Christofides walkthrough on a small 6-city metric

Create a standalone D3 v7 HTML file for Figure Step-by-step Christofides walkthrough on a small 6-city metric. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: step-by-step Christofides walkthrough on a small 6-city metric graph — four panels: (1) the 6-city graph with edge weights; (2) MST highlighted, odd-degree vertices circled; (3) minimum perfect matching on odd-degree vertices added as dashed edges; (4) Eulerian circuit shortcut to Hamiltonian tour with the shortcut edges annotated; annotate the MST cost ≤ OPT and matching cost ≤ OPT/2 arguments at the relevant panels; student should see the algorithm as a physical construction, not just a proof. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/11-approximation-algorithms-fig-03.html`

---

### Figure 11.4 — Before-and-after diagram for knapsack FPTAS value rounding 

Create a standalone D3 v7 HTML file for Figure Before-and-after diagram for knapsack FPTAS value rounding . Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: before-and-after diagram for knapsack FPTAS value rounding — left: bar chart of original item values (varying heights); right: same items with values rounded down to nearest multiple of Δ (stepped heights); annotate Δ = ε·v_max/n below; label the gap between original and rounded value on one bar as "≤ Δ per item"; label total error as "≤ n·Δ = ε·v_max"; student should see geometrically why the rounding error is bounded and how it connects to the ε guarantee. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/11-approximation-algorithms-fig-04.html`

---

### Figure 11.5 — Empirical ratio distribution for vertex cover 2-approximation on

Create a standalone D3 v7 HTML file for Figure Empirical ratio distribution for vertex cover 2-approximation on. Use the CDN https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js, inline CSS, ResizeObserver redraw, SVG role="img", aria-labelledby, title, and desc. Build the figure from this structural brief: empirical ratio distribution for vertex cover 2-approximation on random Erdős-Rényi graphs — x-axis is empirical ratio (approx/OPT), y-axis is frequency; the distribution should be centered around 1.1–1.3 with no observations near 2.0; annotate the theoretical worst-case bound as a vertical dashed line at x=2; student should see viscerally that the worst-case bound is rarely approached on random instances, grounding Misreading 1 in data rather than assertion. Use the described data shape and labels; when exact values are not supplied, use plausible illustrative values that preserve the relationships in the brief. Use a zero baseline for bars or areas, direct labels where possible, and annotations named in the brief. Use only DESIGN.md color variables and the required serif/mono font split.

> Reference implementation: `d3/11-approximation-algorithms-fig-05.html`
