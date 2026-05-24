# Chapter 10 — NP-Completeness and Intractability

*The hardest problems are not impossible. They are hard in a precise sense, and the precise sense matters.*

---

In the summer of 1971, Stephen Cook published a paper proving that one particular problem — Boolean satisfiability, the question of whether a logical formula with variables and AND/OR/NOT connectives can be made true — was in a specific sense the hardest problem of its type. Not just hard. The hardest, in a way that could be made mathematically precise.

The argument was this: every problem whose solutions can be verified quickly can be transformed, in polynomial time, into a SAT instance. Solve SAT efficiently and you solve all of them. SAT was not one hard problem among many. It was a representative of an entire class of problems that were all, in a formal sense, equivalent in difficulty.

This discovery, independently made by Leonid Levin in the Soviet Union around the same time, opened a floodgate. Richard Karp published 21 more problems in the same equivalence class in 1972. By 1979, Garey and Johnson's catalog had hundreds. Today there are thousands. Graph coloring, traveling salesman, protein folding, circuit layout, scheduling, packing, covering, partitioning — problems from every domain of engineering and science — all in the same class.

The class is called NP-complete, and this chapter is about what it means, how to recognize it, and — most importantly — what to do about it.

---

## Four classes, and why the distinctions matter

The theory of computational complexity partitions problems by how the resources needed to solve them scale with input size. Four classes are essential to understand, because they are routinely conflated and the conflation leads to genuine engineering mistakes.

**P** is the class of problems solvable in polynomial time by a deterministic algorithm. Polynomial time means the running time is `O(n^k)` for some constant `k`, where `n` is the input size measured in bits. Sorting is in P. Shortest path is in P. Linear programming is in P — Khachiyan proved this in 1979. Problems in P are considered tractable: as inputs grow, the computation time grows in a manageable way.

**NP** is the class of problems whose yes-instances have solutions that can be verified in polynomial time. The "NP" stands for nondeterministic polynomial — the original definition involved a nondeterministic machine that could guess the right answer, but the cleaner intuition is verification: given a candidate solution, you can check it quickly. SAT is in NP: given a truth assignment to all variables, you can check whether every clause is satisfied in polynomial time. Hamiltonian cycle is in NP: given a candidate cycle, you can verify that it visits every vertex exactly once in polynomial time. P is a subset of NP — if you can solve a problem in polynomial time, you can certainly verify a solution in polynomial time, by solving it yourself and checking the claim.

**NP-hard** means something is at least as hard as the hardest problems in NP, in the following precise sense: every problem in NP can be transformed into it in polynomial time. If you had a polynomial-time algorithm for an NP-hard problem, you could use it — together with the transformation — to solve every problem in NP in polynomial time. NP-hard problems need not themselves be in NP. The halting problem is NP-hard but undecidable, which means it is not solvable by any algorithm at all, let alone a polynomial-time one.

**NP-complete** means both: in NP (solutions are verifiable quickly) and NP-hard (everything in NP reduces to it). NP-complete problems are the hardest problems in NP. Cook's theorem proved SAT is NP-complete. Karp's 21 problems are NP-complete. The thousands in Garey and Johnson are NP-complete.

The big open question is whether P equals NP. No proof exists either way. If P = NP, then every NP-complete problem has a polynomial-time algorithm that nobody has found yet — solutions could be found as quickly as they can be verified. If P ≠ NP, which is what essentially everyone in computer science believes, then NP-complete problems have no polynomial-time algorithms, ever. The Clay Mathematics Institute has offered a million-dollar prize for a proof in either direction.

For working engineers, the practical stance is: assume P ≠ NP. If you find a polynomial-time algorithm for an NP-complete problem, you have either made an error or solved the most famous open problem in mathematics.

![Venn diagram of P, NP, NP-complete, and NP-hard](images/10-npcompleteness-and-intractability-fig-01.png)
*Figure 10.1 — Venn diagram of P, NP, NP-complete, and NP-hard*

---

## How to recognize NP-completeness

The recognition question has two directions: proving a problem is NP-hard, and recognizing the patterns that suggest it.

The proof technique is **reduction**. A polynomial-time reduction from problem A to problem B is an algorithm that transforms any instance of A into an instance of B in polynomial time, such that the answer to B on the transformed instance gives the answer to A on the original. If A is NP-hard and you have a polynomial-time reduction from A to B, then B is NP-hard — any efficient algorithm for B would imply an efficient algorithm for A, which would imply efficient algorithms for everything in NP.

To prove a new problem X is NP-hard, find a known NP-hard problem Y and build a polynomial-time reduction from Y to X. The reduction certifies that X is at least as hard as Y.

3-SAT is the standard starting point for NP-hardness proofs. It is a version of SAT where every clause has exactly three literals. It is NP-complete. The canonical reductions — 3-SAT to vertex cover, to independent set, to clique, to graph coloring, to subset sum — are the scaffolding from which thousands of other hardness proofs are built. Knowing a handful of them is enough to handle most new cases: you find the one whose structure most resembles your problem, adapt the reduction, and the hardness follows.

The practical recognition patterns are three. The first: your problem is an instance of a known NP-complete problem rephrased. Vertex cover, independent set, graph coloring, Hamiltonian cycle, traveling salesman, subset sum, set cover, bin packing — if your problem is secretly one of these, its hardness is inherited directly. The catalog is large and well-indexed.

The second: the obvious algorithm for your problem enumerates an exponentially large search space. You are choosing among subsets of n items, or orderings of n cities, or assignments of n variables to binary values. Greedy gives wrong answers. Dynamic programming requires a state space that is itself exponential. The problem looks like it wants to be solved by trying everything.

The third: there is no obvious structure to exploit. Chapter 9 showed that network flow solves many problems that look hard by coincidence of phrasing — bipartite matching, maximum cut in certain graph classes, assignment problems. Chapter 8 showed that dynamic programming solves problems that look exponential because they have overlapping subproblems with compact structure. Chapter 6 showed that greedy algorithms solve problems with matroid structure optimally. If none of these frameworks fit, the problem may genuinely be hard.

Worth checking before concluding NP-hardness: many problems that look hard are in P because a non-obvious polynomial algorithm exists. 2-SAT is solvable in linear time; 3-SAT is NP-hard. Maximum weight matching in bipartite graphs is polynomial; in general graphs it is harder but still polynomial. Graph coloring with 2 colors is polynomial (bipartiteness); with 3 or more it is NP-hard. The difference of one variable in a problem's structure can move it from P to NP-complete.

![Reduction graph showing 3-SAT at the top with](images/10-npcompleteness-and-intractability-fig-02.png)
*Figure 10.2 — Reduction graph showing 3-SAT at the top with*

---

## The catalog of hard problems

The full catalog has thousands of entries. The working engineer needs perhaps two dozen. Here they are.

**SAT and 3-SAT.** Boolean satisfiability: given a formula in conjunctive normal form, is there an assignment of variables making it true? Cook's theorem makes SAT the universal starting point. 3-SAT restricts clause size to exactly three literals.

**Vertex cover.** Find the minimum set of vertices such that every edge touches at least one of them. Every graph has a vertex cover of size n (all vertices); the question is the minimum.

**Independent set.** Find the maximum set of vertices with no two adjacent. Vertex cover and independent set are complementary: a set S is a vertex cover if and only if its complement is an independent set.

**Clique.** Find the maximum complete subgraph — a set of vertices all pairwise adjacent.

**Graph coloring.** Assign colors to vertices so no two adjacent vertices share a color, using the minimum number of colors. NP-hard for 3 or more colors. The chromatic number of a graph is the answer; computing it is hard.

**Hamiltonian cycle and path.** Does a cycle (or path) exist that visits every vertex exactly once? Eulerian circuits — visiting every edge exactly once — are polynomial by Euler's theorem. The one-word difference (vertex vs. edge) puts one in P and the other in NP-complete.

**Traveling salesman.** Find the minimum-cost cycle visiting every city exactly once. NP-hard in general. Admits a 1.5-approximation in the metric case (Christofides 1976), recently improved slightly. Exact algorithms (Held-Karp) handle instances to a few thousand cities with modern hardware.

**Subset sum.** Given a set of integers and a target, does any subset sum to the target? NP-hard in the bit-length sense. Has a pseudo-polynomial dynamic programming algorithm — polynomial in the numeric value of the target, exponential in the number of bits encoding it. This distinction matters; Chapter 8 covers it.

**Knapsack.** Given items with weights and values and a weight capacity, maximize total value. Same pseudo-polynomial DP as subset sum. Admits an FPTAS — a polynomial-time approximation scheme with any desired accuracy.

**Set cover.** Given a universe and a collection of subsets, find the minimum number of subsets whose union covers the universe. Greedy achieves a `ln n` approximation; this is essentially optimal unless P = NP.

**Integer linear programming.** Optimize a linear objective over linear constraints where variables must take integer values. NP-hard in general. Special structure (totally unimodular matrices) makes certain ILP instances polynomial — Chapter 13 covers this.

| problem | decision version | best known exact bound | best approximation ratio | special tractable cases — one row per problem listed above |
| --- | --- | --- | --- | --- |
| cases | one row per problem listed above | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| reader sees all eleven at once for comparison and lookup | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## What to do about it

This is the chapter's most important section. NP-completeness is not a verdict. It is a classification that says: polynomial-time exact algorithms are not available, so choose among the alternatives.

**Solve exactly if n is small.** Many NP-hard problems have exact algorithms substantially faster than brute-force enumeration. Held-Karp solves TSP in `O(n² · 2^n)` — better than `n!` by an enormous factor, and feasible for n up to roughly 25. Branch-and-bound algorithms with intelligent pruning handle much larger instances on problems with good bounding functions. Modern SAT solvers — Glucose, MiniSAT, CaDiCaL, Kissat — routinely handle industrial instances with millions of clauses, exploiting unit propagation, clause learning, and restart strategies that make typical-case complexity far better than worst case. The worst case is exponential; the typical case on structured real-world instances is often effectively polynomial. "Exact algorithm with exponential worst case" does not mean "slow in practice."

**Approximate with a guaranteed ratio.** For many NP-hard problems, polynomial-time algorithms exist with provable approximation ratios. Vertex cover admits a 2-approximation — always within a factor of 2 of optimal — via a simple maximal matching. Set cover admits a `ln n`-approximation via greedy, and this is the best possible ratio unless P = NP. Metric TSP admits a 1.5-approximation. Knapsack admits an FPTAS: for any ε > 0, a polynomial-time algorithm returns a solution with value at least `(1 − ε)` of optimal. The trade-off is optimality for a polynomial-time algorithm with a bound on how far the solution can stray. Chapter 11 covers approximation algorithms in full.

**Apply heuristics when guarantees are not required.** Local search, simulated annealing, genetic algorithms, and tabu search produce good solutions on real instances without theoretical bounds. Used heavily in practice — in logistics, circuit layout, scheduling, bioinformatics — when instances are too large for exact algorithms and the structure is too irregular for guaranteed approximation. The trade-off: you give up formal guarantees in exchange for practical performance on workloads that resist analytical treatment.

**Exploit special-case structure.** Many NP-hard problems become polynomial on restricted input classes. Vertex cover is polynomial on bipartite graphs — König's theorem gives an equivalence to maximum matching. Graph coloring is polynomial for 4 colors on planar graphs (Four Color Theorem) but NP-hard for 3 colors even on planar graphs. 2-SAT is linear time; 3-SAT is NP-hard. Problems on graphs of bounded treewidth admit polynomial-time algorithms via tree decomposition for many otherwise hard problems. Look for the structure your real instances actually have. A general NP-hard problem may be tractable on the specific slice that matters to you.

**Use parameterized complexity.** If an NP-hard problem has a natural parameter k that is small in practice, an algorithm running in `O(f(k) · n^c)` is feasible when k is small even if n is large. Vertex cover is fixed-parameter tractable in the cover size: a `O(2^k · n)` algorithm exists. For k ≤ 30, this is practical regardless of n. Many problems that are hopeless as general instances are tractable with the right parameterization.

**Encode as SAT or ILP and let a solver handle it.** Modern SAT solvers and ILP solvers — Gurobi, CPLEX, OR-Tools — handle problem sizes that would have been infeasible a decade ago. If you can formulate your problem as SAT or integer linear programming, you can hand it to a solver without implementing a custom algorithm. This is the default engineering approach for many NP-hard problems in industrial practice. The encoding is the skill; the solver does the search.

**Be aware of phase transitions.** Many NP-hard problems show a phase transition — a critical density or parameter value below which instances are easy (almost certainly satisfiable or solvable) and above which they are also easy (almost certainly unsatisfiable or infeasible), with a narrow hard region near the transition. For 3-SAT, the hard region is near a clause-to-variable ratio of approximately 4.26. Random instances near this ratio are notoriously difficult; instances far from it are not. If your real instances cluster away from the phase transition, the worst-case bound is a poor predictor of your wall-clock time.

![3-SAT phase transition ](images/10-npcompleteness-and-intractability-fig-03.png)
*Figure 10.3 — 3-SAT phase transition *

---

## Worked example: course scheduling as graph coloring

A university must assign courses to time slots such that no student is enrolled in two courses in the same slot. Given a set of courses and the student enrollment in each, what is the minimum number of time slots needed?

The reduction to graph coloring is direct. Build a graph with one vertex per course. Add an edge between two courses if they share at least one enrolled student. Courses sharing a student cannot share a time slot; adjacent vertices in the graph cannot share a color. The minimum number of time slots equals the chromatic number of the graph.

Why the chromatic number? Any valid schedule assigns each course a time slot — a coloring of the vertex. Any two courses sharing a student get different slots — adjacent vertices get different colors. Conversely, any proper coloring of the graph corresponds to a valid schedule. The problems are equivalent.

Graph coloring with three or more colors is NP-hard. The reduction inherits the hardness: scheduling a university's courses in the minimum number of time slots is, in general, as hard as graph coloring.

What real universities actually do is instructive. They do not solve the global optimization from scratch each term. They start from last term's schedule and modify it incrementally for new courses and changed enrollments, which reduces the effective instance size by orders of magnitude. The algorithms used are heuristics — DSATUR (saturation-degree greedy) and RLF (recursive largest first) — that produce schedules close to optimal on instances with the structure real enrollment data has, which is far from worst case. When the constraints are complex enough — faculty preferences, room accessibility, examination windows, accreditation requirements — the problem is encoded as an integer linear program and handed to a solver. The solution emerges in minutes to hours.

The resulting schedule has conflicts. A faculty member assigned to two simultaneous sections. A classroom used twice. An accessibility constraint violated. A human scheduler reviews the output, edits, and re-runs. After several iterations, an acceptable schedule emerges. The algorithm is a partner in the process, not a black box delivering a final answer.

Three lessons from this example. First, NP-hard problems are routinely solved on the instances that actually arise in practice. Second, the engineering approach is almost never pure optimization from scratch — it is incremental, heuristic, iterated, and problem-specific. Third, the hardness is not a reason to stop. It is a reason to choose carefully among the seven options listed above.

![Course-scheduling conflict graph ](images/10-npcompleteness-and-intractability-fig-04.png)
*Figure 10.4 — Course-scheduling conflict graph *

---

## What "NP-complete means impossible" gets wrong

The most consequential misconception this chapter addresses is the one where an engineer sees "NP-hard" and stops working on the problem. The classification does not support that response.

NP-hardness is a worst-case statement. It says: on the hardest instances of this problem, no polynomial-time algorithm exists (assuming P ≠ NP). It does not say that your instances are the hardest. It does not say that exponential algorithms are useless in practice. It does not say that approximations do not exist. It does not say anything about undecidability — NP-hard problems are decidable with finite computation; the halting problem is not, and these are categorically different.

Modern SAT solvers processing millions of clauses are exponential-worst-case algorithms performing polynomial-time in practice. ILP solvers handling real scheduling, logistics, and chip-design instances are solving NP-hard problems. Approximation algorithms with 1.5-approximation ratios for metric TSP are running on delivery routing problems every day. The theory tells you what is not available — a guaranteed polynomial-time exact algorithm — and leaves open an enormous space of alternatives.

The corrective: when a problem is NP-hard, work through the seven alternatives. Almost always, at least one fits. The action is to engage, not to retreat.

---

## Decision rules

| Situation | Approach |
|---|---|
| Problem looks hard; want to confirm | Reduce from a known NP-hard problem to yours |
| Confirmed NP-hard, n small (≤ 25–30) | Exact exponential (Held-Karp, branch-and-bound) |
| Confirmed NP-hard, structured instance | SAT solver or ILP solver |
| Confirmed NP-hard, want bounded-quality solution | Approximation algorithm (Chapter 11) |
| Confirmed NP-hard, want best practical solution | Heuristic (local search, simulated annealing) |
| Confirmed NP-hard, real instances on a special class | Special-case algorithm |
| Confirmed NP-hard, small natural parameter k | FPT algorithm |
| Looks NP-hard but not yet proven | Attempt reduction from 3-SAT; check flow/DP/LP first |

---

## What this chapter does not enable

This chapter does not enable proving P ≠ NP — the Millennium Prize Problem remains open. It does not cover the polynomial hierarchy, counting complexity (#P), interactive proofs, probabilistically checkable proofs, or the PCP theorem; those belong in a computational complexity textbook. It also does not cover concrete lower bounds for specific algorithm models — circuit complexity, communication complexity — which are research-frontier questions independent of the P vs. NP question.

---

## Capability statement

You can now determine whether a problem is likely NP-hard via reduction, correctly distinguish P from NP from NP-complete from NP-hard, choose among the seven strategies for NP-hard problems in practice, and recognize when a problem that looks intractable is actually solvable on the instances you care about. The next time a problem looks hard, you can identify whether the hardness is real, what kind of hardness it is, and what to do about it.

---

## Exercises

**Warm-up**

1. Define each of the four complexity classes — P, NP, NP-hard, NP-complete — in your own words, using the verification intuition rather than the nondeterministic machine definition. For each class, give one canonical example and explain why it belongs there. *(Tests: understanding the four classes as distinct concepts with different membership criteria.)*

2. Vertex cover and independent set are described as complementary. Prove this formally: given a graph G with n vertices, show that a set S is a vertex cover if and only if its complement V \\ S is an independent set. What does this complementarity imply about the hardness of independent set, given that vertex cover is NP-hard? *(Tests: working through a concrete hardness-by-complementarity argument.)*

3. The chapter states that 2-SAT is solvable in linear time but 3-SAT is NP-hard, and that graph 2-coloring is polynomial but graph 3-coloring is NP-hard. What structural property do the polynomial-time versions have that the NP-hard versions lack? Describe in one paragraph what changes when you add the third option (third color, third literal per clause) that makes the problem hard. *(Tests: understanding tractability boundaries as structural differences, not just arbitrary facts.)*

**Application**

4. A company wants to assign employees to project teams such that no two employees with a known conflict of interest appear on the same team. Formalize this as a graph problem, identify which canonical NP-hard problem it reduces to, and state the reduction explicitly. Then identify at least two conditions on the input that would make the problem polynomial-time solvable. *(Tests: constructing a reduction from a real-world problem to a canonical NP-hard problem; identifying tractable special cases.)*

5. You are given the following NP-hard problem: schedule n jobs on m identical machines to minimize the makespan (the time at which all jobs finish). For each of the seven strategies in the "What to do about it" section, state whether it applies to this problem and if so, what it would produce. You do not need to implement anything — reason about which strategies are viable given what you know about scheduling. *(Tests: applying the seven-strategy framework to a new NP-hard problem.)*

6. The chapter says that 3-SAT near a clause-to-variable ratio of approximately 4.26 is the hard region, while instances far from this ratio are easy. Explain why an instance with ratio well below 4.26 is easy (hint: think about what happens to satisfiability probability) and why an instance with ratio well above 4.26 is also easy (hint: think about what a solver can do quickly). What does this imply for the relevance of worst-case NP-hardness to real-world 3-SAT instances used in circuit verification or planning? *(Tests: understanding the phase transition as a distribution over instances, not a universal statement about difficulty.)*

**Synthesis**

7. Cook's theorem proves SAT is NP-complete by showing every problem in NP reduces to SAT in polynomial time. Informally, what does the reduction do? Without reproducing the full technical proof, explain the key idea: what structure does a polynomial-time algorithm have that can be encoded as a Boolean formula? Why does this encoding always produce a formula of polynomial size? *(Tests: understanding the conceptual argument of Cook's theorem, not just its conclusion.)*

8. The chapter distinguishes NP-hard (at least as hard as NP) from undecidable (not solvable at all). Give an example of an NP-hard problem and an undecidable problem. For each, describe what happens when you run an algorithm on it for a long time — what does the algorithm eventually do in the NP-hard case versus the undecidable case? Why does this distinction matter for engineering practice? *(Tests: understanding decidability vs. polynomial-time tractability as genuinely different concepts.)*

**Challenge**

9. Suppose you have a new problem Q that you suspect is NP-hard. You try to reduce 3-SAT to Q but cannot find a valid reduction after several attempts. Does this mean Q is in P? Does it mean Q is not NP-hard? What are the two possible explanations for failing to find a reduction, and how would you distinguish between them? Design a research strategy — a sequence of things to try — that would help you determine whether Q is in P or NP-hard when the standard reduction approach has failed. *(Tests: understanding reduction as an investigative tool with failure modes; reasoning about the boundary between P and NP-complete when the answer is unclear.)*

---

## LLM Exercise — Chapter 10: Reductions and the NP-Hard Toolkit

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** A reduction lab — encode a real NP-hard problem in three ways (brute force, SAT encoding, heuristic) and compare how each scales. Bear's lesson is that NP-hard does not mean impossible; the chapter teaches you which strategy to deploy when.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 10 task in the algorithms-by-bear-toolkit. The chapter's
"what to do" section is the actionable content — exact-exponential,
approximation, heuristic, special-case, FPT, SAT/ILP encoding,
phase-transition awareness. The exercise builds three of these
strategies for one problem and compares them.

Pick course scheduling (Bear's worked example) reduced to graph
coloring. Implement and compare four approaches.

In `chapters/ch10_np_completeness/`:

1. `implementations.py`:

   - `graph_coloring_brute_force(graph, k)` — try every k-coloring
     until one works. Useful only for tiny graphs; report timeout
     if n > ~15.
   - `graph_coloring_backtracking(graph, k)` — DPLL-style
     backtracking with constraint propagation. Should handle
     n ≈ 50 comfortably.
   - `graph_coloring_via_sat(graph, k)` — encode as SAT and call
     a real solver (`python-sat` package's `Glucose3` or
     `Cadical`). Decode the SAT solution back to a coloring.
   - `graph_coloring_greedy(graph)` — first-fit greedy heuristic.
     Returns *some* coloring, not necessarily optimal. Report the
     number of colors used.
   - `is_valid_coloring(graph, coloring)` — verification.

   Plus a `reduce_course_scheduling_to_coloring(courses, conflicts)`
   that takes a list of courses and a list of pairs that cannot
   share a time slot, and returns the graph whose chromatic number
   is the minimum number of slots needed.

2. `test_implementations.py` — correctness on hand-checked small
   graphs. All four methods agree on optimal `k` for graphs small
   enough that brute force terminates.

3. `benchmarks.py` — the strategy-comparison study. Generate
   random graphs at varying sizes and edge densities. For each:

   - Brute force (cap at n = 15).
   - Backtracking (cap at n = 60).
   - SAT encoding (cap at n = 200).
   - Greedy (no cap — it always returns a coloring).

   Measure wall time for each. For each (n, density) combination,
   report the smallest `k` each method found and the time taken.
   Plot the strategy frontier — for each graph size, which
   strategy returns the best answer fastest.

   Bonus: investigate the *phase transition* — at what edge
   density does graph coloring become hard? Plot solver time
   against density at fixed n.

4. `README.md` — decision card per strategy. "Surprising findings":
   (a) the strategy frontier in your data; (b) the phase transition
   density you observed (the empirical hard region for k-coloring
   is around density 0.5 for moderate k — does your data match?).

Commit with `ch10: NP-hard strategy comparison on graph coloring`.
```

**What this produces:** Four strategies for the same NP-hard problem, a strategy frontier plot, and (if the phase-transition study runs cleanly) one of the most visually compelling findings in the repo — the empirical hardness peak.

**How to adapt this prompt:**

- *For your own project:* Replace course scheduling with whatever NP-hard problem you actually meet (TSP for delivery routing, knapsack for budget allocation, scheduling with side constraints).
- *For ChatGPT / Gemini:* The SAT encoding is the subtle part — ask each model to encode "at least one color per vertex" and "no two adjacent vertices share a color" as CNF, then verify by inspection.
- *For Claude Code:* Native fit. Install `python-sat` and have it call a real solver; do not let it implement DPLL by hand for this exercise.
- *For a Claude Project:* Skip.

**Connection to previous chapters:** Uses `Graph` from Chapter 5; the greedy first-fit reuses ideas from Chapter 6; harness from Chapter 2.

**Preview of next chapter:** Chapter 11 implements approximation algorithms with known ratios — set cover (greedy ln(n) bound), vertex cover (2-approximation) — and asks: does the worst-case ratio actually show up on real instances?
