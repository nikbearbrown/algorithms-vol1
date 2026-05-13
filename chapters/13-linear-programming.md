# Linear Programming

## TL;DR

Linear programming (LP) is the optimization of a linear objective subject to linear constraints. Reach for this chapter when a problem can be modeled with continuous decision variables, linear costs, and linear feasibility — and many real problems can be. After consulting it, you can formulate an LP, choose between simplex and interior-point solvers, interpret the dual, distinguish LP from IP / MIP / QP, and recognize when LP relaxation provides a useful approximation to a harder integer problem.

## Recognition pattern

The signal: a problem with quantitative decisions, costs that scale linearly with the decisions, and constraints that are inequalities or equalities in those decisions. How much of each product to make, given limited capacity. How to allocate flow across a network, given edge capacities. How to blend ingredients to minimize cost while meeting nutritional requirements. How to schedule resources to maximize value within budgets. The shape: maximize or minimize `cᵀx` subject to `Ax ≤ b`, `x ≥ 0`.

A second signal: the problem has *duality structure*. Every LP has a dual LP whose optimal value matches the primal's. The dual variables have economic meaning — shadow prices, marginal values of constraints. When the optimal solution and its dual are both available, you understand not just the answer but its sensitivity.

A third signal: a problem that is NP-hard as an *integer* program may be tractable as a *linear* program. The LP relaxation drops integrality constraints; the resulting LP is polynomial-time solvable; the LP optimum is a bound on the IP optimum, and rounding the fractional LP solution gives an approximation algorithm (Chapter 11).

A signal LP is *not* the right tool: the objective or constraints are non-linear (quadratic programming, second-order cone programming, semidefinite programming, general convex optimization). Or: the decisions are discrete and integrality matters fundamentally (integer programming, NP-hard in general). Or: the problem has stochastic elements that LP cannot encode without losing the linear structure.

The misconception engaged in §10 is the one that keeps practitioners from reaching for LP at all: "LP is for academics; production systems use heuristics." This is wrong. LP solvers are foundational to operations research, supply chain, finance, energy, transportation, and increasingly to machine learning. The capability is industrial; the misconception is provincial.

## What you need to know first

This chapter assumes Big O (Chapter 2) and basic linear algebra (vector dot products, matrix multiplication, systems of linear equations). For LP duality's relationship to max-flow min-cut, see Chapter 9 §3. For LP relaxation as approximation technique, see Chapter 11 §4. For the matroid-LP intersection that makes greedy optimal on certain problems, see Chapter 6 §5.

## The standard form

A linear program in standard form:

```
maximize    cᵀx
subject to  Ax ≤ b
            x ≥ 0
```

where `x ∈ ℝⁿ` is the vector of decision variables, `c ∈ ℝⁿ` the objective coefficients, `A ∈ ℝᵐˣⁿ` the constraint matrix, `b ∈ ℝᵐ` the right-hand side.

Variants:

- **Minimization:** `min cᵀx` is equivalent to `max -cᵀx`.
- **Equality constraints:** `Ax = b` can be encoded as `Ax ≤ b` and `Ax ≥ b`.
- **Free variables:** `x ∈ ℝ` (no sign constraint) can be encoded as `x = x⁺ - x⁻` with `x⁺, x⁻ ≥ 0`.
- **Bounded variables:** `l ≤ x ≤ u` with explicit bounds is supported directly by most solvers.

LP solvers accept all these variants natively; the standard form is for analysis, not for input.

The **feasible region** — the set of `x` satisfying all constraints — is a polyhedron (intersection of half-spaces). The objective `cᵀx` is linear, so its level sets are hyperplanes. The optimum, when it exists and is bounded, is achieved at a vertex of the polyhedron. This geometric fact is what makes the simplex algorithm work.

Three possible outcomes for an LP: a unique optimum (vertex), multiple optima (a whole face of the polyhedron achieves the optimum), or no finite optimum (infeasible: no `x` satisfies the constraints; or unbounded: the objective grows without bound).

## The simplex algorithm

Dantzig 1947 [verify]. The original LP algorithm and still the production workhorse for many problem classes.

**Geometric idea.** The optimum lies at a vertex. Start at any feasible vertex; move along an edge of the polyhedron in a direction that improves the objective; arrive at a neighboring vertex. Repeat until no improving direction exists. The vertex you stop at is optimal.

**Algorithmic idea.** Maintain a *basis* — a set of `m` of the `n` variables held at non-zero values, with the rest at their bounds. The basis defines a vertex. A *pivot operation* swaps one basic variable for one non-basic variable, moving to a neighboring vertex.

**Worst case is exponential.** Klee-Minty 1972 [verify] constructed instances where simplex visits exponentially many vertices. In practice, simplex is fast: on real-world instances, the number of pivots is typically `O(m + n)` or so. Smoothed analysis (Spielman and Teng 2001) [verify] explains this gap — under small random perturbations, simplex runs in expected polynomial time.

**Pivoting rules.** Several strategies for choosing the next pivot: Dantzig's rule (most-negative reduced cost), Bland's rule (lexicographically smallest index, prevents cycling), steepest-edge (best objective improvement per unit step). Modern solvers use sophisticated rules tuned to instance type.

**Production performance.** Commercial simplex implementations (Gurobi, CPLEX, FICO Xpress) handle LPs with millions of variables and constraints [verify]. The combination of decades of pivoting rule research, sparse linear algebra, and presolve techniques makes simplex effective on a vast range of practical problems.

## Interior-point methods

Karmarkar 1984 [verify] gave the first polynomial-time interior-point algorithm that was competitive with simplex in practice. Earlier polynomial-time algorithm (the ellipsoid method, Khachiyan 1979) was a theoretical breakthrough but practically slow.

**Geometric idea.** Move through the interior of the feasible region toward the optimum, rather than along the boundary as simplex does. Each iteration takes a Newton-method-like step on a perturbed objective.

**Algorithmic idea.** Replace the constraints with a barrier function that is finite inside the feasible region and goes to infinity at the boundary. Solve the barrier problem with Newton's method. Decrease the barrier strength to zero, tracking the optimum.

**Performance properties.** Polynomial-time worst case: `O((m + n)^(3/2) · log(1/ε))` iterations to ε-accuracy with appropriate analysis [verify]. In practice, interior-point methods take a relatively small constant number of iterations (~20–50) regardless of problem size, with each iteration costing a sparse linear system solve. They scale better than simplex on very large LPs (millions of variables); simplex retains an edge on warm-starting (re-solving after a small change).

**Production rule of thumb.** For one-shot solves of large LPs, prefer interior-point. For repeated solves where each is a small modification of the last (branch-and-bound, sensitivity analysis, parametric programming), prefer simplex. Modern solvers run both and pick automatically.

## Duality

Every LP has a *dual* LP. The dual of

```
max cᵀx subject to Ax ≤ b, x ≥ 0
```

is

```
min bᵀy subject to Aᵀy ≥ c, y ≥ 0.
```

**Strong duality theorem.** If the primal has an optimal solution, so does the dual, and the optimal values are equal: `cᵀx* = bᵀy*`. The dual variables `y_i` have an economic interpretation — they are the *shadow prices* or *marginal values* of the primal constraints. `y_i` is the rate of increase in the optimal objective per unit of relaxation in the `i`-th constraint.

**Complementary slackness.** At optimum, for each constraint either the constraint is tight (binding) or its dual variable is zero. Equivalently: a slack constraint costs nothing to relax further; a binding constraint has a positive shadow price.

**Why duality matters.**

*Sensitivity analysis.* Want to know how much the optimum changes if a constraint's right-hand side changes by ε? The dual variable tells you the linear-order change. Critical for "what if" analysis on planning models.

*Bounds on integer programs.* The LP relaxation gives a bound on the IP optimum. The dual of the LP relaxation gives a *certificate* of that bound. Branch-and-bound algorithms use this duality every iteration.

*Reformulation.* Sometimes the dual is easier to solve than the primal. Network flow's max-flow min-cut (Chapter 9) is the canonical example: the max-flow primal and min-cut dual are linked by strong duality.

*Economic interpretation.* In production planning, the shadow prices tell management which capacities to expand and which are wasteful. The dual is not an algorithmic trick; it is the structural answer to "what is the marginal value of this resource?"

## LP, IP, MIP, QP — the family

LP is one member of a family of optimization problems. Knowing where each lives is part of the recognition pattern.

**LP — Linear Programming.** Linear objective, linear constraints, continuous variables. Polynomial-time solvable.

**IP — Integer Programming.** Linear objective, linear constraints, integer variables. NP-hard in general.

**MIP — Mixed Integer Programming.** Some variables continuous, others integer. NP-hard. Solved by branch-and-bound combining LP relaxations with integer branching.

**0-1 IP / Binary IP.** Variables restricted to 0 or 1. NP-hard but a standard formulation for combinatorial problems (set cover, vertex cover, knapsack, scheduling).

**QP — Quadratic Programming.** Quadratic objective, linear constraints. Convex QP is polynomial; non-convex QP is NP-hard. Used in finance (portfolio optimization with risk), control, machine learning (SVMs).

**SOCP — Second-Order Cone Programming.** Generalizes LP and convex QP. Polynomial-time. Used in robust optimization, antenna design, control.

**SDP — Semidefinite Programming.** Generalizes SOCP. Polynomial-time. Used in approximation algorithms (Goemans-Williamson MAX-CUT 0.878-approximation, 1995) [verify].

**General convex programming.** Convex objective, convex feasible set. Polynomial-time given oracle access. The umbrella that contains LP, QP, SOCP, SDP, and others.

The transition from LP to IP is where the practitioner most often gets stuck. Many real problems are MIP, not LP, and the gap matters: MIP solvers can take orders of magnitude longer than LP solvers on the same number of variables.

## Decision rules

| Problem signature | Approach |
| --- | --- |
| Linear objective, linear constraints, continuous variables | LP via simplex or interior-point |
| Linear objective, linear constraints, integer variables | IP via branch-and-bound (NP-hard) |
| Mix of continuous and integer | MIP solver (Gurobi, CPLEX, OR-Tools) |
| Want shadow prices and sensitivity analysis | Solve LP and read the dual |
| Need lower bound on IP optimum | LP relaxation |
| Need approximation algorithm for NP-hard problem | LP relaxation + rounding (Chapter 11) |
| Quadratic objective, linear constraints, convex | QP solver |
| Network flow problem | Reduce to LP (Chapter 9) — but specialized solvers are faster |
| Very large LP, one-shot | Interior-point |
| Repeated LP solves with small modifications | Simplex (warm-starts well) |
| Robust to data uncertainty needed | SOCP or robust LP formulation |
| Approximation ratio via SDP | Specialized SDP solver |
| Don't know if problem is LP-shaped | Try to write the constraints as `Ax ≤ b` |

## Worked example — production planning at a refinery

A refinery processes crude oil into multiple finished products: gasoline, diesel, jet fuel, fuel oil, lubricants. Crude is purchased from suppliers at varying prices. Each unit of crude can be processed by several distillation paths producing different yield ratios. Each path has a capacity constraint (throughput, time, equipment availability). Demand for each product is forecast and must be met. The question: how much of each path to run on each crude type, to maximize profit subject to capacity and demand?

This is one of the canonical LP applications. Refineries have used LP since the 1950s [verify]; the original applications drove much of the early commercial LP solver development.

**Formulation.** Decision variables `x_{ij}` = amount of crude `i` processed by path `j`. Objective: `max Σ (revenue from products − cost of crude × quantity − processing costs)`. Constraints:

- Capacity: total throughput on each path `≤ path's capacity`
- Crude availability: total of each crude type `≤ supplier's offering`
- Demand: total of each product produced `≥ demand forecast`
- Non-negativity: `x_{ij} ≥ 0`

For a refinery with 5 crude types, 8 processing paths, and 6 finished products, the LP has 40 decision variables and roughly 19 constraints — solvable in milliseconds by any modern solver. Real refineries have hundreds of variables and constraints; still solvable in seconds.

**The optimum and the dual.** Solving gives the optimal production plan. The dual gives the shadow prices.

- Shadow price on a capacity constraint: how much profit increases per unit of additional capacity. If a distillation tower's shadow price is $0.50 per barrel, expanding throughput by 10,000 barrels per day adds $5,000/day to profit (linear extrapolation, locally valid).
- Shadow price on a crude constraint: the marginal value of one more barrel of that crude type. If the shadow price is $2 above the purchase price, the refinery would pay up to $2 more per barrel for incremental supply.
- Shadow price on a demand constraint: the marginal cost of meeting one more unit of demand. If the price is high, the demand is expensive to meet — possibly an opportunity to negotiate price or shed demand.

**Sensitivity analysis.** What if jet fuel demand jumps 10%? The LP can be re-solved or, for small changes, the dual gives a linear-order estimate. What if a refinery undergoes maintenance and one path's capacity drops to zero for a week? Re-solve with that constraint binding. The LP framework supports rapid scenario analysis.

**Real refinery LPs include extensions.**

- *Pooling problems* — when intermediate streams are blended in tanks, the constraints become non-linear (bilinear). These are non-convex and require specialized algorithms or approximation. Major source of difficulty in practice.
- *Multi-period planning* — sequence of LPs across time, with inventory linking periods.
- *Stochastic LP* — uncertainty in demand or prices, optimized over scenarios. Two-stage stochastic LPs are LPs themselves; multi-stage become harder.
- *Integer constraints* — scheduling decisions (start a unit, shut down a unit, transition between modes) are 0/1 integers; the resulting MIP is harder than the LP.

**Lessons.** The basic refinery LP is straightforward; the LP framework scales to real production planning at industrial scale. The dual gives information not present in the primal — sensitivity, marginal values, what-if analysis. The practitioner who can formulate the LP and read the dual unlocks both the optimum and its sensitivity in one solve.

## Failure modes — when "LP is for academics" misleads

The misconception engaged: "LP is for academics; production systems use heuristics."

This is wrong about industrial reality, wrong about LP capability, and wrong about the trade-off heuristics actually make.

**LP is industrial.** Airline scheduling, refinery production, power-grid dispatch, supply-chain optimization, telecommunications network design, financial portfolio optimization — all of these are LP-driven (or MIP-driven, with LP as the relaxation backbone) at scale. Major airlines solve LPs with millions of variables daily. The supply chain of a global retailer is an MIP. The power grid is an LP solved every few minutes. "LP is for academics" describes an audience the literature serves; it does not describe where LP runs.

**Heuristics ≠ better-than-LP.** A heuristic without a guarantee may be faster on small instances. At scale, LP solvers benefit from decades of solver engineering — sparse linear algebra, presolve, cutting planes, dual simplex, parallel branch-and-bound. Heuristics that look good in textbooks may lose to LP-based methods on real workloads.

**LP relaxation is the workhorse for NP-hard problems.** Approximation algorithms (Chapter 11) almost always use LP somewhere — relaxation, rounding, primal-dual, dual-fitting. Modern combinatorial optimization is unrecognizable without LP.

**Solver progress is real and ongoing.** LP solvers have improved by orders of magnitude over the past 30 years, combining algorithmic advances (interior-point methods, presolve, primal-dual algorithms) with hardware (cache-friendly linear algebra, parallelism, vectorization). Problems considered infeasible a decade ago are now routine.

**Degeneracy and numerical stability are real but managed.** LP can struggle with degeneracy (multiple optimal vertices), unbounded problems, numerical conditioning. Production solvers handle these via lexicographic perturbations, scaling, and careful arithmetic. The practitioner should know about these issues but does not usually need to engineer around them.

**The genuine cases where LP is wrong.** Non-linear constraints (pooling, blending). Discrete decisions where integrality is essential (scheduling on/off, routing). Stochastic problems with deep recursion. These require LP's relatives (QP, MIP, stochastic programming), not LP itself. The misconception is "LP is rare"; the corrective is "LP is everywhere, and its limits are specific."

The corrective heuristic: when a problem looks like quantitative resource allocation under linear costs and constraints, formulate the LP. Solve it. Read the dual. If integrality is essential, escalate to MIP. The LP-first move is the production move.

## Cross-references

For LP duality applied to network flow (max-flow min-cut), see Chapter 9 §3. For LP relaxation and rounding as approximation, see Chapter 11 §4. For matroid LPs and greedy optimality, see Chapter 6 §5. For randomized rounding in LP-based approximation, see Chapter 12. For NP-hardness of integer programming, see Chapter 10.

## Companion-page handoffs

Solver comparison (Gurobi, CPLEX, FICO Xpress, COIN-OR CBC, OR-Tools, SciPy linprog) on benchmark instances; modeling tutorials in Python (PuLP, CVXPY, Pyomo); refinery-LP case study with code; airline-scheduling MIP example; power-grid dispatch demonstration; sensitivity-analysis walkthrough; introduction to QP and SDP. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-13.

## What this chapter does not enable

This chapter does not enable implementing simplex or interior-point methods from scratch. Production solvers are decades of careful engineering; reimplementing them is not the right call for almost any practical problem. The chapter also does not cover advanced LP topics — column generation, Lagrangian relaxation, decomposition methods, parametric programming at depth. For those, consult Bertsimas and Tsitsiklis's *Introduction to Linear Optimization* or Vanderbei's *Linear Programming: Foundations and Extensions*. Stochastic and robust LP variants live in operations research literature; the LP relaxation chapter touches them but does not develop them.

## Capability statement

You can now formulate a problem with linear constraints and a linear objective as an LP; choose between simplex (warm-starts well, classical workhorse) and interior-point (better on large one-shot solves); interpret the dual as shadow prices and use it for sensitivity analysis; distinguish LP from IP / MIP / QP and know which solver each requires; recognize when LP relaxation provides an approximation bound for an NP-hard problem; and avoid the misconception that LP is academic rather than industrial. The next time a problem arrives that looks like quantitative allocation under linear constraints — refinery, airline, supply chain, portfolio, network — the path from problem to LP to solution is in your hands. This chapter closes the optimization arc the volume opened with.


---

## LLM Exercise — Chapter 13: Linear Programming and the Capstone Study

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** A real LP formulation — production planning, with multiple products, capacity constraints, and demand forecasts — plus dual interpretation, sensitivity analysis, and an LP-relaxation approximation that ties back to Chapter 11.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 13 task in the algorithms-by-bear-toolkit. This is the
capstone — LP closes the optimization arc the volume opened with. The
exercise formulates a real planning problem as an LP, solves it,
interprets the dual as shadow prices, runs a sensitivity analysis,
and demonstrates LP relaxation as an approximation tool that connects
back to Chapter 11.

In `chapters/ch13_linear_programming/`:

1. `implementations.py`:

   - `production_planning_lp(products, resources, demand_forecast)`
     — formulates a multi-product production planning LP using
     `pulp`. Decision variables: how much of each product to make.
     Objective: maximize profit. Constraints: resource capacity,
     demand cap per product. Returns the optimal production plan,
     the optimal objective value, and the dual variables.
   - `interpret_dual(lp_result)` — translates dual variables into
     plain-English shadow prices: "increasing capacity of resource
     X by 1 unit would increase optimal profit by Y."
   - `sensitivity_analysis(lp_problem, parameter, range)` — sweep
     a parameter (demand for product A, capacity of resource B)
     across a range; return how the optimal objective changes.
   - `vertex_cover_lp_relaxation(graph)` — LP relaxation of the
     ILP for minimum vertex cover. Returns the fractional optimum
     and a rounded integer solution. (Each fractional value ≥ 0.5
     becomes 1.)
   - `solve_min_cost_flow_via_lp(flow_network)` — show that
     min-cost-flow is an LP. Solve via `pulp` and compare to a
     specialized solver from Chapter 9.

2. `test_implementations.py`:

   - Production planning: on a tiny 2-product, 2-resource case with
     known optimal, verify the LP returns the known answer.
   - Dual interpretation: on a case where one constraint is binding
     and one is not, verify the non-binding constraint has zero
     dual.
   - Vertex cover LP relaxation: on small graphs, confirm the
     rounded integer solution is at most 2× the true ILP optimum
     (this is the 2-approximation that links back to Chapter 11).

3. `benchmarks.py` — three studies:

   **Study A: simplex vs. interior-point.** For varying LP sizes
   (10, 100, 1000, 10000 variables), solve with both methods and
   compare runtimes. `pulp` and `scipy.optimize.linprog` both
   expose method choices.

   **Study B: LP relaxation vs. ILP exact.** On 100 random
   instances of vertex cover and set cover, compute (a) the LP
   relaxation rounded; (b) the exact ILP optimum (via `pulp` with
   integer variables); (c) the Chapter 11 combinatorial
   approximation. Compare ratios.

   **Study C: shadow prices in action.** On the production
   planning LP, perturb the most-binding constraint by ±10% and
   plot how optimal profit moves. Show that the slope equals the
   shadow price.

4. `README.md` — decision cards for LP / IP / MIP / QP framing,
   production planning template, dual interpretation, LP
   relaxation, simplex vs. interior-point. "Surprising findings":
   the ratio gap between LP relaxation and combinatorial
   approximation on your random instances, and any case where
   simplex beat interior-point or vice versa unexpectedly.

   **Capstone section** in the README: a one-paragraph reflection
   on what the toolkit can now do that it couldn't at Chapter 1.
   Name three real problems in your work or domain that the
   toolkit could now address. This is the "what was built" summary
   of the whole repo.

Commit with `ch13: linear programming with dual interpretation and
LP-relaxation study`.

Then create a final commit `repo: top-level README updated with
capstone summary` that updates the top-level README to summarize
all 13 chapters' deliverables and the cross-references between them.
```

**What this produces:** A real LP, dual interpretation, sensitivity analysis, LP-relaxation comparison to Chapter 11's combinatorial approximations, and a capstone README summary tying the whole repo together. This is the final commit of the project.

**How to adapt this prompt:**

- *For your own project:* Replace the synthetic production-planning numbers with a real planning problem from your domain — refinery, airline scheduling, ad budget allocation, portfolio rebalancing, server provisioning.
- *For ChatGPT / Gemini:* LP modeling syntax varies by library; settle on `pulp` first and verify a tiny known-optimum case before scaling.
- *For Claude Code:* Native fit. Let it run the sensitivity study and produce the shadow-price slope plot — the visual is the chapter's most teaching moment.
- *For a Claude Project:* Now is the time. After Chapter 13 closes, create a Claude Project loaded with the repo's top-level README and the 13 chapter READMEs as project files. You now have a personal *Algorithms by Bear* consultant that knows what you built.

**Connection to previous chapters:** Ties back to Chapter 9 (min-cost flow as LP), Chapter 11 (LP relaxation as approximation technique), Chapter 6 (matroid intersection LPs), Chapter 10 (LP relaxation of NP-hard ILPs). The capstone README references the full chapter arc.

**Preview of next chapter:** None — this is the volume's final chapter. The companion repository is now a portfolio-grade artifact. Open issues for what *Vol. 2* (decision-making algorithms: Bayesian methods, optimal stopping, game theory) would add and you have a continuation plan.
