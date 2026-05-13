# Approximation Algorithms

## TL;DR

An approximation algorithm runs in polynomial time and returns a solution with a provable bound on quality — typically a multiplicative ratio relative to optimal. Reach for this chapter when a problem is NP-hard (Chapter 10) but you need a polynomial-time algorithm with a guarantee. After consulting it, you can identify the canonical approximation algorithms, distinguish constant-factor / PTAS / FPTAS / inapproximable problems, and correctly interpret what an approximation ratio does and does not promise.

## Recognition pattern

The signal: you have an NP-hard problem (Chapter 10) and you need a polynomial-time algorithm with a quality guarantee. Heuristics give no guarantee. Exact algorithms give the right answer but exponential time. Approximation sits in between — polynomial time, bounded quality.

Three concrete signs an approximation algorithm is the right tool.

*The problem is NP-hard but admits structural relaxation.* Many NP-hard problems become tractable when "find the optimum" is relaxed to "find within a factor of optimum." Vertex cover, set cover, metric TSP, knapsack, bin packing — all admit polynomial-time approximations with known ratios.

*You need a guarantee, not just an answer.* A heuristic might give a great answer on your data; you cannot prove it. An approximation gives a provably-bounded answer on every input. The guarantee matters when the algorithm is making consequential decisions and "usually fine" is not enough.

*You need bounded suboptimality across instance variation.* Real workloads vary. A heuristic that is great on benchmark A and terrible on benchmark B is a deployment risk. An approximation with ratio `α` produces a solution within `α · OPT` on *every* instance — adversarial, average, or yours.

A signal approximation is *not* the right tool: the problem admits a polynomial-time exact algorithm and you missed it. Check Chapter 6 (greedy), Chapter 7 (D&C), Chapter 8 (DP), Chapter 9 (flow), Chapter 13 (LP) before declaring NP-hardness.

The misconception engaged in §8 is the one practitioners actually fall into: misreading what the ratio says about a *specific instance*. The ratio is a worst-case bound over all instances; on your specific instance, the algorithm may do much better, or it may hit the worst case. The chapter's most practitioner-discriminating content is in §6 (when ratios mislead) and §8.

## What you need to know first

This chapter assumes Big O (Chapter 2), greedy structure (Chapter 6), DP (Chapter 8), graph theory (Chapter 5), and an NP-hardness vocabulary (Chapter 10). For LP relaxation as an approximation technique, see Chapter 13. For randomized rounding, see Chapter 12.

## Approximation ratio — the contract

An algorithm `A` is an `α`-approximation for a minimization problem if, for every instance, `A`'s solution costs at most `α · OPT`, where `OPT` is the optimal cost. For a maximization problem, `A`'s solution achieves at least `OPT / α`. Convention: `α ≥ 1`. Smaller is better.

Three classes of approximation guarantees, in increasing strength.

**Constant-factor approximation.** The ratio is a constant independent of input size. Vertex cover admits a 2-approximation. Metric TSP admits a 1.5-approximation (Christofides 1976) [verify]. Steiner tree admits a 1.39-approximation [verify]. Useful when "within a constant factor" is acceptable.

**Polynomial-Time Approximation Scheme (PTAS).** For any `ε > 0`, the algorithm produces a `(1 + ε)`-approximation in time polynomial in `n` (but possibly exponential in `1/ε`). Examples: Euclidean TSP (Arora 1998, Mitchell 1999) [verify], minimum makespan scheduling. The user picks `ε`; smaller `ε` means better solution, more time.

**Fully Polynomial-Time Approximation Scheme (FPTAS).** Same `(1 + ε)` guarantee, but in time polynomial in *both* `n` and `1/ε`. Stronger than PTAS. The 0/1 knapsack admits an FPTAS via DP rounding. Bin packing admits an asymptotic PTAS but not an FPTAS [verify].

**Logarithmic-factor approximation.** Set cover admits an `H_n ≈ ln n`-approximation via greedy (Chapter 6). The ratio grows with input size, but slowly.

**Inapproximable problems.** Some NP-hard problems are provably hard to approximate within any constant factor, unless P = NP. Maximum clique cannot be approximated within `n^(1−ε)` for any `ε > 0` unless P = NP (Håstad 1996) [verify]. The PCP theorem (Arora-Lund-Motwani-Sudan-Szegedy 1998) [verify] is the foundational machinery for inapproximability proofs.

## Design techniques

Four canonical techniques. Each yields approximation algorithms for a class of problems.

**Greedy.** Make locally optimal choices; bound the worst-case suboptimality. Set cover greedy gives `ln n`. Vertex cover via the matched-edge approach gives 2 (described below).

**Local search.** Start with a feasible solution; improve by local moves until no improvement is possible. Many local-search algorithms have provable approximation ratios; many do not. Used heavily in practice when no constant-factor algorithm exists.

**Linear programming relaxation and rounding.** Formulate the problem as integer linear programming (ILP); relax integrality (allow fractional values); solve the LP in polynomial time (Chapter 13); round the fractional solution to an integer solution. The rounding scheme determines the approximation ratio. Vertex cover via LP rounding gives 2 (matching the greedy bound by a different proof). The technique is general — many approximation algorithms are LP-rounding underneath.

**Dynamic programming with rounding.** When a problem has a pseudo-polynomial DP solution (Chapter 8 §6), rounding the input values can produce a polynomial-time algorithm with `(1 + ε)` approximation. The 0/1 knapsack FPTAS works this way: round each value to a precision determined by `ε`, run DP on the rounded values, return the result.

## Canonical approximation algorithms

**Vertex cover, 2-approximation (matched-edge).** Find any maximal matching `M`. Output the set of all endpoints of edges in `M`. Bound: `|M|` is a lower bound on optimum (every vertex cover must touch every matched edge with a distinct vertex), and the algorithm outputs `2|M|` vertices. Ratio: 2. Simple, fast, and tight up to a small constant — vertex cover is hard to approximate below 2 unless P = NP.

The "max-cover greedy" — repeatedly pick the vertex covering the most uncovered edges — is a different algorithm. It also achieves 2-approximation but with more work; the matched-edge algorithm is the cleaner one.

**Set cover, `H_n`-approximation (greedy).** Repeatedly pick the set covering the most uncovered elements. The bound: the algorithm outputs a cover of size at most `H_n · OPT`, where `H_n = 1 + 1/2 + 1/3 + ... + 1/n ≈ ln n`. The ratio is essentially tight: `(1 − o(1)) ln n` is the best polynomial-time ratio possible unless P = NP (Feige 1998) [verify].

**Metric TSP, 1.5-approximation (Christofides).** For TSP on a metric (triangle inequality holds): build the MST, find a minimum-weight perfect matching on the odd-degree vertices, combine into an Eulerian multigraph, shortcut to a Hamiltonian cycle. Ratio: 1.5. Held the best-known ratio for metric TSP for over 40 years until 2020 improvements [verify].

**Knapsack, FPTAS.** Round item values to a precision tied to `ε`, run pseudo-polynomial DP, return the rounded-DP solution. Time: `O(n³/ε)`. For `ε = 0.01` (1% from optimum), the algorithm runs in `O(100 n³)`.

**Bin packing, First-Fit Decreasing (FFD).** Sort items by decreasing size; for each item, place it in the first bin with enough remaining capacity, opening a new bin if none fits. Ratio: roughly 11/9 in the asymptotic sense (Yue 1991) [verify]. The exact constants are subtle; the practical performance is excellent.

**Scheduling, list scheduling on identical machines.** Assign each job in arrival order to the machine with the least current load. Ratio: 2 − 1/m (Graham 1966) [verify]. Improvement to 4/3 + ε via PTAS.

**Vertex cover, LP-rounding 2-approximation.** Solve the LP relaxation (each vertex assigned a fractional value in [0, 1] subject to "for every edge, the two endpoints' values sum to at least 1"). Round: include every vertex with value ≥ 1/2. The constraint guarantees every edge has an included endpoint. Ratio: 2. Same bound as matched-edge, different proof.

**Steiner tree, 2-approximation.** Build the minimum spanning tree on the metric closure of required vertices. Ratio: 2. Improvements to 1.39 [verify] via more sophisticated techniques.

## When the ratio misleads

The chapter's most practitioner-discriminating content. An approximation ratio is a worst-case guarantee; it is not a prediction.

**Worst case may not be your case.** A 2-approximation guarantees no instance produces a solution worse than 2 · OPT. On your specific instance, the algorithm may produce 1.01 · OPT — or 2 · OPT exactly. The ratio bounds the worst; it does not characterize the typical. For most real instances, the algorithm performs better than the bound, sometimes much better.

**The constant in the ratio matters.** A "2-approximation" sounds good. But for an NP-hard problem where exact solutions are 100 units, a 2-approximation might output 200. If a heuristic with no guarantee outputs 105 on the same instance, the heuristic wins on this case. The approximation's value is the *guarantee*, not the constant — but in production, both matter.

**The ratio is the *worst* over all instances.** A `(1 + ε)`-PTAS sounds tight. But the running time may be `n^(1/ε)`, which is `n^100` for `ε = 0.01`. The algorithm is "polynomial" but uselessly slow. The FPTAS distinction (polynomial in `1/ε` too) is what makes the scheme practical.

**Two algorithms with the same ratio can perform very differently.** Two 2-approximations for vertex cover (matched-edge vs LP-rounding) produce different solutions. On your instances one may consistently beat the other in practice. The ratio guarantees the worst; it does not rank the algorithms on average performance.

**The ratio assumes the model.** Metric TSP's 1.5-approximation requires the triangle inequality. If your distances violate it (e.g., asymmetric distances or distances that can game the algorithm), the bound is void. A practitioner who applies Christofides without checking the metric assumption may get arbitrarily bad results.

**Inapproximability is a hard wall.** Some problems cannot be approximated below a threshold unless P = NP. Maximum clique, MAX-3SAT (above 7/8), graph coloring (above `n^(1−ε)`), set cover (below `ln n`) — these have proven lower bounds on polynomial-time approximation. No future algorithm will improve the ratio without first resolving P vs NP.

## Decision rules

| Problem situation | Approach |
| --- | --- |
| NP-hard, want bounded-quality polynomial-time solution | Approximation algorithm |
| Want `(1 + ε)` for any `ε`, time polynomial in `n` | PTAS |
| Want `(1 + ε)` time polynomial in `n` and `1/ε` | FPTAS |
| Constant-factor sufficient | Constant-factor approximation |
| Need to know if better-than-`α` is possible | Check inapproximability literature |
| Greedy gives `ln n`-approximation, set-cover-style | Greedy approximation |
| Problem has natural LP formulation | LP relaxation + rounding |
| Pseudo-polynomial DP exists | DP rounding for FPTAS |
| Worst case rare, want best practical performance | Heuristic; approximation as fallback or sanity check |
| Ratio matters less than absolute performance | Compare algorithms on representative benchmarks |

## Worked example — set cover for sensor placement

A municipal water utility wants to place water-quality sensors in its distribution network. The network has 10,000 junctions [verify]. Each potential sensor location, if installed, can detect contamination at a known set of downstream junctions before the contamination reaches consumers. The utility wants to find the minimum number of sensor locations such that every junction is monitored by at least one sensor.

This is set cover. Universe `U` = junctions. Subsets `S_i` = the set of junctions that sensor location `i` covers. Goal: minimum subset of `S_i` whose union is `U`.

Set cover is NP-hard. Exact solution is infeasible at this scale.

**Greedy approximation.** Repeatedly pick the candidate sensor location covering the most uncovered junctions. Time: `O(|U| · m)` where `m` is the number of candidate locations. For 10,000 junctions and 1,000 candidate locations [verify], this is `10⁷` operations — instant.

**Bound.** The greedy solution uses at most `H_n · OPT ≈ 9.21 · OPT` sensors for `n = 10,000`. So if the optimal placement is 50 sensors, greedy uses at most ~460 sensors. That bound is loose for most realistic instances; in practice, greedy on water-network sensor placement typically runs at 1.1× to 1.3× optimal [verify].

**When the bound is meaningful.** For arbitrary set-cover instances, the `ln n` ratio is essentially tight (Feige 1998). On adversarial inputs designed to make greedy fail, the ratio approaches `ln n`. On your sensor-placement instance, the structure (geographic locality, downstream-only coverage, network topology) usually makes greedy do much better.

**When the bound misleads.** If a few candidate sensors cover most of the network and many candidates cover only a few junctions, greedy's first picks dominate; subsequent picks are nearly optimal. If coverage is highly uniform — every candidate covers about the same number of junctions — greedy's worst-case behavior is closer to the bound. The water utility should benchmark on synthetic and historical contamination scenarios; the ratio is a backstop, not a prediction.

**Alternative: ILP via solver.** Encode as integer linear programming (Chapter 13). Solve with Gurobi, CPLEX, or OR-Tools. For 10,000 junctions, modern solvers handle this size in minutes [verify]. The exact solution beats greedy; the time cost is non-trivial but manageable. In production, the practitioner runs both — solver for the ground truth, greedy for fast iteration during model development.

**Alternative: LP rounding.** Solve the LP relaxation, round fractional variables to integers via deterministic threshold or randomized rounding (Chapter 12). For set cover, LP rounding gives the same `O(log n)` ratio as greedy, with a different constant.

**Other production set-cover instances.**

*Server location.* CDN edge-server placement: candidate cities, demand cities, latency thresholds. Each candidate "covers" the demand cities it can serve under threshold. Find minimum servers covering all demand. NP-hard; greedy and LP-rounding deployed routinely.

*Test-suite reduction.* Software regression testing: candidate tests, code branches each test exercises. Find minimum tests covering all branches. Tools like Coverity and SonarQube use set-cover heuristics for this. [verify].

*Feature selection.* Machine learning: candidate features, training examples each feature distinguishes. Find minimum features distinguishing all examples. The matroid analog admits polynomial algorithms for some special cases (Chapter 6); the general case is set cover.

The lesson: set cover is everywhere once you can see it. The greedy ratio is loose on most real instances. Knowing the bound exists tells you nothing will be catastrophically worse; knowing the bound is loose tells you the typical case is better than the bound suggests. Both readings matter.

## Failure modes — when "the ratio tells you how good your answer is" misleads

The misconception engaged: "An approximation guarantee tells you how good the answer is on this instance."

It does not. The ratio is a worst-case bound over all instances. Three concrete failures.

**Treating the ratio as a per-instance bound.** A 2-approximation guarantees no instance produces a result worse than 2 · OPT. It does not promise your particular instance produces a result close to 2 · OPT, or close to OPT. The bound is over a hostile adversary; your instance may be much better, or might hit the adversary's worst case. To know how the algorithm performs on your data, benchmark.

**Comparing algorithms by ratio rather than performance.** A 1.5-approximation sounds better than a 2-approximation. On a specific instance, the 2-approximation may produce a smaller cover, finish faster, and use less memory. The ratio is one dimension; production performance includes time, memory, implementation complexity, and how well the algorithm handles structured inputs. Choose by what matters for your workload.

**Assuming polynomial-time = practical-time.** A PTAS with running time `n^(1/ε)` is polynomial. For `ε = 0.05` (5% from optimum) and `n = 1000`, this is `1000^20 = 10^60` operations — older than the universe. The FPTAS distinction matters: only fully polynomial schemes are reliably practical for tight `ε`.

**Forgetting the model assumptions.** Christofides' 1.5-approximation requires a metric (triangle inequality). LP rounding assumes the LP has integral optima or a known integrality gap. Greedy set cover assumes you can compute coverage cheaply. Violating an assumption voids the bound silently.

**Chasing tighter ratios where practical performance does not need them.** Set cover greedy is `H_n`. There is no constant-factor polynomial-time algorithm. Spending engineering time to improve from `H_n` to `H_n − 1` is rarely worth it; spending time on instance-aware heuristics that beat `H_n` on your specific data often is. The theoretical literature optimizes the worst case; production optimizes the typical case.

The corrective heuristic: state the ratio and the model assumptions; benchmark on representative instances; report typical-case behavior alongside the worst-case guarantee. The ratio is a contract about the worst; the benchmark is the prediction about the typical.

## Cross-references

For the greedy algorithms underlying many approximation algorithms (vertex cover, set cover, MST), see Chapter 6. For DP-based FPTAS (knapsack), see Chapter 8 §6. For LP relaxation and rounding in detail, see Chapter 13 §3. For randomized rounding and Las Vegas / Monte Carlo distinctions, see Chapter 12. For the NP-hardness reductions that motivate approximation, see Chapter 10.

## Companion-page handoffs

Comparative approximation-algorithm benchmarks across vertex cover, set cover, TSP, knapsack, bin packing, scheduling; PCP theorem reading list; inapproximability catalog; LP rounding tutorials; FFD bin-packing implementation with worked-out worst-case constructions; Christofides' algorithm walkthrough. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-11.

## What this chapter does not enable

This chapter does not give a procedure for designing a new approximation algorithm for an arbitrary problem. The art of approximation — finding an LP relaxation with a small integrality gap, designing a primal-dual algorithm, exploiting problem-specific structure — is research-level material; for that, consult Williamson and Shmoys's *The Design of Approximation Algorithms* or Vazirani's *Approximation Algorithms*. The chapter also does not cover online approximation (decisions made under uncertainty about future input), which lives in online-algorithms literature, or the specific machinery of the PCP theorem and inapproximability hardness proofs.

## Capability statement

You can now identify or apply an approximation algorithm with a known ratio for a given NP-hard problem; distinguish constant-factor / PTAS / FPTAS / inapproximable; correctly interpret what the ratio guarantees and what it does not; choose between greedy, LP-rounding, and DP-rounding design techniques; and avoid the failure modes that come from misreading the ratio as a per-instance bound. The next time an NP-hard problem arrives with "we need a fast solution that we can defend," the path from problem to algorithm and ratio is in your hands.


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
