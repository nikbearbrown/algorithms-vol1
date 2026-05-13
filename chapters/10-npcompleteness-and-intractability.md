# NP-Completeness and Intractability

## TL;DR

NP-completeness is a classification of problems for which no polynomial-time algorithm is known and none is believed to exist. Reach for this chapter when you suspect a problem is computationally hard and want to know what to do about it. After consulting it, you can recognize NP-hardness via reduction, correctly interpret what NP-hardness means in practice (it is not "impossible"), and choose between exact-exponential, approximation, heuristic, and special-case strategies for hard problems.

## Recognition pattern

The signal: a problem looks combinatorial. It involves choosing among exponentially many configurations — assignments, subsets, orderings, partitions — and the obvious algorithm enumerates them. You are not confident a polynomial-time algorithm exists. You suspect there is some classical hard problem hiding inside yours.

Three concrete signs to check.

*The problem matches a canonical NP-complete pattern.* SAT, 3-SAT, vertex cover, independent set, clique, graph coloring, Hamiltonian cycle, traveling salesman, subset sum, set cover, bin packing, integer programming. If your problem rephrases as one of these, NP-hardness is almost certain. The classical list (Garey and Johnson 1979) [verify] catalogs hundreds.

*A polynomial reduction from a known NP-hard problem maps to your problem.* If you can transform 3-SAT (or any NP-hard problem) into your problem in polynomial time, your problem is at least as hard as 3-SAT — therefore NP-hard.

*The problem space grows exponentially with input size, and there is no obvious structure to exploit.* Brute force is `2^n` or `n!`; greedy gives wrong answers; DP either gives `O(2^n)` solutions (which is exact-exponential, not polynomial) or fails because the subproblem space is exponential.

A signal NP-completeness is *not* the right framing: the problem is solvable in polynomial time and you have not noticed. Worth checking before declaring intractability — many problems that look hard have polynomial-time algorithms via flow (Chapter 9), DP (Chapter 8), or matroid theory (Chapter 6).

The misconception engaged in §9 is the bigger trap: assuming NP-complete means impossible. It does not, and the chapter's most actionable content — §6 — is what to do instead.

## What you need to know first

This chapter assumes Big O (Chapter 2), basic graph theory (Chapter 5), and DP (Chapter 8) for the exact-exponential algorithms section. For approximation algorithms with provable ratios on NP-hard problems, see Chapter 11. For randomized algorithms applied to hard problems, see Chapter 12. For LP-based exact and approximate solutions, see Chapter 13.

## P, NP, NP-complete, NP-hard

Four formal classes. The definitions matter because they are routinely conflated in practice.

**P.** Problems solvable in polynomial time by a deterministic algorithm. "Polynomial time" means the running time is `O(n^k)` for some constant `k`, where `n` is the input size in bits. P is the class of problems considered tractable. Sorting is in P. Shortest path is in P. Linear programming is in P (Khachiyan 1979) [verify].

**NP.** Problems whose *yes-instances* have polynomial-time *verifiable* certificates. A solution can be checked quickly even if finding one is hard. SAT is in NP: given a candidate assignment, you can check satisfaction in polynomial time. Hamiltonian cycle is in NP: given a candidate cycle, you can verify it visits every vertex once in polynomial time. P ⊆ NP (every polynomial-time algorithm is also a polynomial-time verifier on its own output).

**NP-hard.** A problem is NP-hard if every problem in NP can be reduced to it in polynomial time. NP-hard problems are at least as hard as the hardest problems in NP. NP-hard problems need not themselves be in NP — the halting problem is NP-hard but undecidable, far beyond NP.

**NP-complete.** A problem that is both in NP and NP-hard. The "hardest" problems in NP, in the precise sense that all of NP reduces to them. The first proven NP-complete problem was SAT (Cook 1971; independently, Levin 1973) [verify]. Karp (1972) [verify] gave 21 more, opening the floodgates; the modern catalog has thousands.

The big open question: **does P = NP?** No proof either way exists. The consensus in theoretical CS is that P ≠ NP, but the consensus is unproven. If P = NP, every problem in this chapter has a polynomial-time algorithm we just have not found; if P ≠ NP (the believed case), no such algorithm exists for any NP-complete problem.

For practical purposes, treat P ≠ NP as the working assumption. If you find a polynomial-time algorithm for an NP-complete problem, you have either made a mistake or solved a Millennium Prize Problem.

## Reductions — the proof technique

A polynomial-time reduction from problem `A` to problem `B` is a polynomial-time algorithm that transforms instances of `A` into instances of `B` such that the answer to `A` can be read from the answer to `B`. If `B` has a polynomial-time algorithm, so does `A` (apply the reduction, then solve `B`). Equivalently: if `A` is hard, so is `B`.

To prove a problem `X` is NP-hard, reduce a known NP-hard problem `Y` to `X`. The reduction certifies that `X` is at least as hard as `Y`.

**Canonical reductions.** Knowing a few standard reductions covers many practical cases.

- **3-SAT to vertex cover.** Standard textbook reduction; clauses become triangles, variables become edges between literal-vertices.
- **3-SAT to independent set.** Same construction read differently — vertex cover and independent set are complementary.
- **3-SAT to clique.** Variant construction.
- **3-SAT to graph coloring.** Each variable's literals are connected; clause gadgets enforce satisfaction; color classes encode truth values.
- **Vertex cover to set cover.** Standard reduction; each vertex becomes a set covering its incident edges.
- **Hamiltonian path to TSP.** Add edges with appropriate weights.
- **Subset sum to partition.** Standard.
- **3-SAT to integer linear programming.** Each variable becomes a 0/1 integer; clauses become linear constraints.

When in doubt, reduce from 3-SAT. It is the workhorse starting point for NP-hardness proofs.

**Reductions also show problems are NOT NP-hard.** If you reduce your problem to bipartite matching (polynomial), or shortest path (polynomial), or LP (polynomial), it is in P. The reduction is the proof in either direction.

## Common NP-complete problems

A short catalog. The full Garey-Johnson catalog has more than 300 problems [verify]; the practitioner relevant subset is maybe two dozen.

**SAT and 3-SAT.** Boolean satisfiability. The Cook-Levin theorem proves these NP-complete; they are the universal "starting point" for reductions.

**Vertex cover.** Minimum set of vertices touching every edge.

**Independent set.** Maximum set of pairwise non-adjacent vertices.

**Clique.** Maximum complete subgraph.

**Graph coloring (chromatic number).** Minimum colors needed so adjacent vertices have different colors. NP-hard for `k ≥ 3`. Worked example below.

**Hamiltonian cycle / path.** Cycle / path visiting every vertex exactly once.

**Traveling salesman (TSP).** Minimum-cost cycle visiting every vertex. NP-hard in general; admits a 1.5-approximation in the metric case (Christofides 1976) [verify].

**Subset sum / partition.** Find a subset summing to a target. Pseudo-polynomial DP exists (Chapter 8); NP-hard in the bit-length sense.

**Knapsack.** 0/1 knapsack. Pseudo-polynomial DP; FPTAS exists (Chapter 11).

**Set cover / hitting set.** Greedy `ln n`-approximation (Chapter 6, 11); inapproximable below `ln n` unless P = NP (Feige 1998) [verify].

**Bin packing.** Pack items into minimum number of fixed-capacity bins.

**Integer linear programming.** General ILP is NP-hard; certain special cases (totally unimodular constraint matrices) are in P (Chapter 13).

**Steiner tree.** Minimum-weight tree connecting a specified subset of vertices. NP-hard; constant-factor approximation exists.

## What to do when a problem is NP-complete

This section is the chapter's most actionable content. NP-completeness is not a verdict of impossibility — it is a notice that polynomial-time exact algorithms are not available, and you must choose among alternatives.

**1. Solve exactly with an exponential algorithm if `n` is small.**

Many NP-hard problems have exact algorithms substantially faster than brute-force `n!`. Held-Karp solves TSP in `O(n² · 2^n)` (Chapter 8) — feasible for `n ≤ 25` or so [verify]. Branch-and-bound and SAT solvers handle problems with thousands of variables on real-world instances. Modern SAT solvers (Glucose, MiniSAT, CaDiCaL, Kissat) [verify] routinely handle industrial instances with millions of clauses despite the worst-case exponential bound.

The phenomenon: real-world instances often have structure that exact algorithms exploit. The worst case is exponential; the typical case is polynomial. This is the "easy hard problems" observation that drives modern combinatorial optimization.

**2. Approximate with a guaranteed ratio.**

For many NP-hard problems, polynomial-time algorithms exist with provable approximation ratios. Vertex cover admits a 2-approximation. Set cover admits a `ln n`-approximation. Metric TSP admits a 1.5-approximation. Knapsack admits an FPTAS — a polynomial-time scheme with `(1+ε)` approximation for any `ε > 0`. See Chapter 11 for the full treatment.

The trade-off: you give up optimality but gain a polynomial-time algorithm with a provable bound on solution quality.

**3. Heuristic with no guarantee.**

Local search, simulated annealing, genetic algorithms, tabu search, and similar techniques produce good solutions on real-world instances without theoretical guarantees. Used heavily in practice when the instances are too large for exact and the structure is too irregular for guaranteed approximation.

The trade-off: you give up theoretical bounds but gain practical performance on workloads that resist formal analysis.

**4. Solve a special case.**

Many NP-hard problems become polynomial on restricted inputs. Vertex cover is polynomial on bipartite graphs. Graph coloring is polynomial on planar graphs for `k = 4` (Four Color Theorem) [verify] but NP-hard for `k = 3` even on planar graphs. 2-SAT is polynomial; 3-SAT is NP-hard. Tree-decomposable graphs of bounded treewidth admit polynomial-time algorithms for many otherwise hard problems.

Look for the structure your real instances actually have. A general NP-hard problem may be tractable on the slice you care about.

**5. Fixed-parameter tractable algorithms.**

If a problem is hard in general but admits an algorithm running in `O(f(k) · n^c)` for some parameter `k` (independent of `n`), the problem is fixed-parameter tractable (FPT). Vertex cover is FPT in the cover size: `O(2^k · n)` is feasible for `k` up to roughly 30 on real instances [verify]. Many parameterized problems are FPT.

**6. SAT or ILP encoding.**

Modern SAT solvers and ILP solvers (Gurobi, CPLEX, OR-Tools) handle problem sizes that would have been infeasible 20 years ago [verify]. If you can encode your problem as SAT or ILP, the solver does the rest. This is the engineering default for many practical NP-hard problems.

**7. Phase-transition awareness.**

Many NP-hard problems show a phase transition — instances near a critical density are exponentially hard, instances far from it are easy. 3-SAT instances with clause-to-variable ratio near 4.26 are notoriously hard; ratios well above or below are easy [verify]. If your real instances cluster on the easy side, the worst-case bound does not predict your wall-clock time.

## Decision rules

| Situation | Approach |
| --- | --- |
| Problem looks hard; want to confirm | Reduce a known NP-hard problem to it |
| Confirmed NP-hard, n small (≤ 30) | Exact exponential (Held-Karp, branch-and-bound) |
| Confirmed NP-hard, structured instance | SAT solver or ILP solver |
| Confirmed NP-hard, want bounded-quality solution | Approximation (Chapter 11) |
| Confirmed NP-hard, want best practical solution, no guarantee | Heuristic (local search, simulated annealing) |
| Confirmed NP-hard, real instances on a special class | Special-case algorithm |
| Confirmed NP-hard, parameterized in some `k` | FPT algorithm |
| Suspected NP-hard but not yet proven | Look for known reduction; if none, attempt one from 3-SAT |
| Looks NP-hard but might not be | Check for matching, flow, DP, LP structure first |

## Worked example — course scheduling reduced to graph coloring

A university must assign courses to time slots such that no student is enrolled in two courses scheduled at the same time. Each course has a list of enrolled students. The question: what is the minimum number of time slots needed?

**Reduction to graph coloring.**

Build a graph `G`. Each vertex is a course. Add an edge between two courses if they share at least one student. The minimum number of time slots equals the chromatic number of `G` — the minimum number of colors needed to color vertices such that adjacent vertices have different colors.

Why? A valid schedule assigns each course to a time slot. Courses sharing a student cannot share a time slot, so adjacent vertices (in `G`) must get different colors (time slots). Conversely, any valid coloring of `G` is a valid schedule.

**Hardness.** Graph coloring with `k ≥ 3` colors is NP-hard. Course scheduling inherits the hardness via the reduction. There is no known polynomial-time algorithm for the general problem.

**What real universities actually do.**

*Heuristic-driven.* Saturation degree (DSATUR) and recursive largest-first (RLF) heuristics produce schedules that are usually feasible and near-optimal on real registration data [verify]. The "real" structure — student enrollments cluster by department, by year, by program — produces graphs with low chromatic number relative to vertex count, far from worst-case.

*Partial-instance.* Universities do not solve the global problem from scratch each term. They start from last term's schedule, modify for new courses and changed enrollments, and re-solve only the affected subproblem. Incrementality reduces effective instance size by an order of magnitude or more.

*Iteration.* The first generated schedule has conflicts (a faculty member assigned to two simultaneous courses, a room used twice, accessibility constraint violated). The schedule is shown to a human scheduler, who edits and re-runs. The algorithm is a partner, not a black box. After a few iterations, an acceptable schedule emerges.

*ILP encoding for hard cases.* When constraints are complex (faculty preferences, room features, accessibility, exam window constraints), the problem is encoded as an integer linear program and solved by Gurobi or CPLEX. Modern ILP solvers handle real university scheduling instances in minutes to hours [verify].

*Soft constraints.* Real schedules tolerate some conflicts as soft penalties rather than hard constraints. The optimization minimizes a weighted penalty rather than enforcing zero conflicts.

The lesson: the reduction proves the worst case is hard. The practice is far from worst case. Real NP-hard problems are routinely solved by combining structure exploitation, incrementality, heuristics, and modern solvers. "NP-hard" is a starting point for engineering, not a stopping point.

## Failure modes — when "NP-complete means impossible" misleads

The misconception engaged: "NP-complete means impossible."

This is the most consequential misconception in this chapter. Three specific failures.

**Refusing to attempt an NP-hard problem.** A practitioner sees "NP-complete" and abandons the problem. Real NP-hard problems are routinely solved on real instances. The chapter's §6 is what to do; the misconception's response is "give up." The corrective: NP-hardness is a worst-case statement; the typical-case complexity may be polynomial in disguise.

**Believing exponential algorithms are useless.** Modern SAT solvers process millions of clauses; ILP solvers handle problems that would have been infeasible decades ago; specialized exponential algorithms (Held-Karp, color-coding, kernelization) push the boundary. "Exponential" describes growth rate, not absolute slowness.

**Conflating NP-hard with undecidable.** NP-hard problems are decidable (you can solve them with finite computation, just slowly). Undecidable problems (halting problem, first-order logic validity) cannot be solved at all by any algorithm. Missing the distinction leads to despair where engineering would suffice.

**Overestimating worst-case relevance.** The worst-case bound describes the hardest instance. Real instances often cluster far from worst case. 3-SAT clauses generated uniformly at random are easy at most ratios and hard only near phase transitions. Real-world SAT instances from hardware verification, planning, or theorem proving have structure that solvers exploit. The worst case is a bound; it is not the prediction.

**Underestimating reduction's power as a tool.** Reduction is a positive technique, not just a negative one. Modeling your problem as SAT, ILP, MAX-SAT, or constraint satisfaction lets you use industrial-grade solvers for a problem you cannot solve by hand. The reduction is the engineering move, not just a hardness proof.

The corrective heuristic: when a problem is NP-hard, ask which of the seven options in §6 fits. Almost always at least one does. The action is to engage, not to retreat.

## Cross-references

For approximation algorithms with provable ratios on the problems named here, see Chapter 11. For randomized algorithms applied to hard problems, see Chapter 12. For LP relaxation and rounding as exact and approximate solution techniques, see Chapter 13. For DP-based exact-exponential algorithms (Held-Karp), see Chapter 8.

## Companion-page handoffs

Reduction catalog with full constructions for the canonical problems; SAT solver demo (MiniSAT or Glucose); phase-transition visualizations for 3-SAT; ILP encodings for course scheduling, vertex cover, TSP; FPT algorithm walkthroughs; special-case algorithm catalog. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-10.

## What this chapter does not enable

This chapter does not enable proving P ≠ NP. The Millennium Prize Problem remains open; if you find a proof, the remainder of this chapter becomes obsolete. The chapter also does not cover the polynomial hierarchy (PH), the relationship between NP and counting problems (#P), or the modern interactive proofs and PCP machinery; those live in computational complexity texts. Concrete-complexity questions (lower bounds for specific algorithm classes) are also out of scope.

## Capability statement

You can now determine whether a problem is likely NP-hard via reduction, correctly distinguish NP-hard from NP-complete from in-P, choose among the seven strategies for NP-hard problems (exact exponential, approximation, heuristic, special case, FPT, SAT/ILP encoding, phase-transition awareness), and recognize when an "intractable" problem is actually solvable on the instances you care about. The next time a problem looks hard, you can identify whether the hardness is real and what to do about it.


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
