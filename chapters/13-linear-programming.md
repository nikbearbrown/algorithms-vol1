# Chapter 13 — Linear Programming

*The most powerful tool you are not using is the one that has been running the world's supply chains since 1947.*

---

In 1947, the United States Air Force asked George Dantzig to solve a planning problem. They needed to distribute men, equipment, and supplies across a theater of operations under constraints on what could move where, when, and at what cost. The problem had hundreds of variables and hundreds of constraints. No computational device of the era could enumerate the possibilities. Dantzig invented the simplex algorithm and solved it.

Within a decade, oil refineries were using LP to optimize their production schedules. Within two decades, airline companies were using it to schedule crews. Today the power grid you depend on is re-optimized by an LP solver every few minutes. Every major airline uses LP to plan routes, crew assignments, and maintenance windows. The supply chains of global retailers are integer programs built on LP relaxations. The portfolio weights in many financial instruments are LP solutions. If you have used a logistics service, flown commercially, or turned on a light, you have already benefited from LP running somewhere upstream.

The misconception this chapter engages — "LP is for academics; production systems use heuristics" — is one of the most consequential mismatches between theory and practice in all of engineering. This chapter closes that gap.

---

## What the problem looks like

A linear program asks you to optimize a linear objective subject to linear constraints. The standard form is:

$$\text{maximize} \quad \mathbf{c}^\top \mathbf{x}$$
$$\text{subject to} \quad A\mathbf{x} \leq \mathbf{b}, \quad \mathbf{x} \geq 0$$

where $\mathbf{x} \in \mathbb{R}^n$ is the vector of decision variables, $\mathbf{c} \in \mathbb{R}^n$ holds the objective coefficients, $A \in \mathbb{R}^{m \times n}$ is the constraint matrix, and $\mathbf{b} \in \mathbb{R}^m$ is the right-hand side.

The geometry is clean. Each constraint $\mathbf{a}_i^\top \mathbf{x} \leq b_i$ defines a half-space. Their intersection is a polyhedron — the feasible region. The objective $\mathbf{c}^\top \mathbf{x}$ is linear, so its level sets are parallel hyperplanes. To maximize the objective, you push the hyperplane as far as possible in the direction of $\mathbf{c}$ while staying inside the polyhedron. The maximum, when it exists and is bounded, is achieved at a vertex of the polyhedron.

This geometric fact is what makes LP tractable. The feasible region might have infinitely many points, but only finitely many vertices, and the optimum is at one of them.

Three outcomes are possible. The LP has a unique optimum at a vertex. Or it has multiple optima — an entire face of the polyhedron achieves the same objective value. Or it has no finite optimum, either because no $\mathbf{x}$ satisfies the constraints (infeasible) or because the objective grows without bound in the feasible direction (unbounded).

![2D LP geometry diagram ](images/13-linear-programming-fig-01.png)
*Figure 13.1 — 2D LP geometry diagram *

---

## The recognition pattern

The signal that LP is the right tool is a combination of three things: decisions that are continuous quantities, costs and benefits that scale linearly with the decisions, and constraints that are linear inequalities or equalities.

How much of each product to manufacture, given limited machine time and material. How to route flow across a network, given edge capacities. How to blend ingredients to minimize cost while meeting nutritional requirements. How to allocate a budget across advertising channels with diminishing-returns thresholds that can be linearized. The shape is always: maximize or minimize $\mathbf{c}^\top \mathbf{x}$ subject to $A\mathbf{x} \leq \mathbf{b}$.

A second signal is when you need not just the optimal solution but the marginal value of your constraints. Every LP has a dual LP, and the dual variables quantify exactly this: how much does the optimal objective change per unit of relaxation in each constraint? If you are planning production and want to know which piece of equipment is worth upgrading, the LP dual tells you that, in dollars, without requiring a separate analysis.

A third signal is when you have a problem that is NP-hard as an integer program but tractable as a relaxation. Drop the integrality requirements, solve the LP in polynomial time, and you get a lower bound on the integer optimum. Round the fractional solution to get an approximation. Chapter 11 develops this into a systematic technique; LP is its engine.

What LP is not: when constraints or objectives are non-linear (quadratic, non-convex), when decisions are inherently discrete and the fractional relaxation is meaningless, when stochastic elements cannot be encoded without destroying linearity. Those require LP's relatives — quadratic programming, integer programming, stochastic LP — not LP itself.

---

## Simplex: the algorithm that finds the vertex

Dantzig's simplex algorithm does not search all feasible points. It searches only vertices, and it moves between adjacent vertices in a direction that improves the objective.

A vertex is defined by which constraints are active — binding at equality. A set of $m$ active constraints gives a system of $m$ equations in $n$ unknowns; when the system has a unique solution, that solution is a vertex. The simplex algorithm maintains a *basis* — a set of $m$ variables held at positive values, with the remaining $n - m$ at zero — which defines the current vertex. A *pivot* swaps one basis variable for one non-basis variable, moving to an adjacent vertex.

The stopping condition is elegant. At each vertex, compute the *reduced costs* — the rate at which the objective improves if each non-basic variable is increased from zero. When all reduced costs are non-positive, no move can improve the objective, and the current vertex is optimal. The optimality certificate is local and global simultaneously: if no neighboring vertex is better, no vertex anywhere is better.

Worst-case, simplex can visit exponentially many vertices. Klee and Minty constructed a family of LPs — the Klee-Minty cube — where simplex with Dantzig's pivoting rule visits every vertex. In practice, this never happens on real problems. Spielman and Teng's smoothed analysis (2001) showed that under small random perturbations, simplex runs in expected polynomial time, which explains what practitioners already knew: on real-world instances, the number of pivots is typically proportional to the number of constraints, not exponential in it.

Modern simplex implementations in Gurobi, CPLEX, and FICO Xpress handle LPs with millions of variables and constraints. They exploit sparse linear algebra — most real-world constraint matrices are very sparse — presolve techniques that simplify the problem before solving, and sophisticated pivot rules that exploit problem structure. The gap between textbook simplex and production simplex is decades of engineering.

---

## Interior-point methods: the polynomial-time alternative

In 1984, Narendra Karmarkar published an interior-point algorithm for LP that was both polynomial-time and practically competitive with simplex. The geometric idea is different: instead of walking along the boundary of the feasible polyhedron, move through its interior, approaching the optimal vertex from inside.

The mathematical mechanism is a *barrier function* — a function that is finite inside the feasible region and goes to infinity at the boundary. The barrier function is added to the objective, penalizing proximity to the constraints. The modified problem is solved with Newton's method, a step that finds the locally optimal direction in a single matrix inversion. Then the barrier strength is reduced, the Newton step is repeated, and the process tracks the optimal vertex from the interior.

Interior-point methods are polynomial-time with precise guarantees — roughly $O((m+n)^{3/2} \log(1/\varepsilon))$ iterations to $\varepsilon$-accuracy on appropriate problem classes. In practice, they take a small constant number of iterations — typically 20 to 50 — regardless of problem size, with each iteration requiring a sparse linear system solve. They scale well on very large LPs where simplex's iteration count would be prohibitive.

The practical trade-off between simplex and interior-point is warm-starting. Simplex can restart cheaply from the optimal basis of a previous solve when the problem changes slightly: change a right-hand side by a small amount, and simplex needs only a few more pivots from the old optimal vertex to the new one. Interior-point cannot do this — it starts from the interior every time. For branch-and-bound algorithms, which solve thousands of LP relaxations that differ from each other by small amounts, simplex dominates. For one-shot large LP solves, interior-point often wins.

Modern solvers run both and choose automatically.

![Diagram contrasting simplex (path along the polyhedron boundary,](images/13-linear-programming-fig-02.png)
*Figure 13.2 — Diagram contrasting simplex (path along the polyhedron boundary,*

---

## Duality: the answer inside the answer

Every LP has a dual LP, and the duality relationship is one of the deepest results in optimization. The dual of the standard-form LP

$$\text{maximize} \quad \mathbf{c}^\top \mathbf{x} \quad \text{subject to} \quad A\mathbf{x} \leq \mathbf{b}, \quad \mathbf{x} \geq 0$$

is

$$\text{minimize} \quad \mathbf{b}^\top \mathbf{y} \quad \text{subject to} \quad A^\top \mathbf{y} \geq \mathbf{c}, \quad \mathbf{y} \geq 0$$

The **strong duality theorem** says that if the primal has an optimal solution, so does the dual, and the optimal values are equal: $\mathbf{c}^\top \mathbf{x}^* = \mathbf{b}^\top \mathbf{y}^*$. The primal's maximum equals the dual's minimum.

The dual variables $y_i$ have a direct economic interpretation. They are the *shadow prices* of the primal constraints — the rate of change in the optimal objective per unit increase in the right-hand side $b_i$. If constraint $i$ says "machine capacity is at most 1000 hours" and its shadow price is $\$5$/hour, then relaxing that constraint to 1001 hours increases the optimal profit by approximately $\$5$. Exactly $\$5$ for small changes; the linearity of LP makes this exact in a neighborhood of the optimum.

**Complementary slackness** is the optimality condition that captures this precisely. At the optimum, for each constraint either the constraint is binding (tight, used at full capacity) or its shadow price is zero. A constraint with slack — some capacity left unused — costs nothing to relax further, so its shadow price is zero. A binding constraint has a positive shadow price, measuring what would be gained by relaxing it.

The shadow prices tell you things the primal solution does not. The primal tells you what to do. The dual tells you which constraints are holding you back and by how much. In production planning, shadow prices tell management which capacity constraints are worth expanding. In diet optimization, they tell you which nutritional requirements are binding and what each is costing. In network routing, they relate to the bottleneck edges — and this is exactly where Chapter 9's max-flow min-cut theorem lives: it is a special case of LP duality.

![Sensitivity analysis plot ](images/13-linear-programming-fig-03.png)
*Figure 13.3 — Sensitivity analysis plot *

---

## LP, IP, MIP, QP: where the boundaries are

LP is one member of a family of optimization problems. The boundaries matter because crossing from LP into integer programming changes the computational complexity from polynomial to NP-hard.

**LP** — linear objective, linear constraints, continuous variables — is polynomial-time solvable, as established by Khachiyan's ellipsoid method in 1979 and in practice by simplex and interior-point.

**IP (integer programming)** — the same setup with integer variables — is NP-hard. Adding integrality to a linear program changes the feasible region from a polyhedron to a discrete set of lattice points, and the optimum over lattice points can be far from the vertex of the LP relaxation.

**MIP (mixed-integer programming)** — some variables continuous, others integer — is also NP-hard and is the most common form in practice. Almost any real scheduling, routing, or assignment problem with both quantity decisions (continuous) and binary yes/no decisions (integer) is a MIP.

**0-1 integer programming** — binary variables — is the standard formulation for combinatorial problems: is this edge in the cut (0 or 1), is this item in the knapsack (0 or 1), is this customer assigned to this facility (0 or 1). Chapter 10's NP-complete problems all have natural 0-1 IP formulations.

**QP (quadratic programming)** — quadratic objective, linear constraints — is polynomial-time when convex (used in portfolio optimization and support vector machines) and NP-hard when non-convex.

The transition from LP to MIP is where most practitioners encounter the computational cliff. An LP with 10,000 variables solves in seconds. Adding even a few hundred binary variables creates a MIP where branch-and-bound searches exponentially many LP relaxations. MIP solvers (Gurobi, CPLEX) apply decades of techniques — cutting planes, presolve, heuristic branching — to make large MIPs tractable in practice despite the theoretical complexity.

The LP relaxation — solving the MIP as an LP by dropping integrality constraints — is the core subroutine of branch-and-bound. Every node in the branch-and-bound tree solves an LP. The LP optimum is an upper bound (for maximization) on the true integer optimum. If the LP optimum happens to be integral, it is the optimal integer solution. If it is fractional, the algorithm branches on a fractional variable, creating two subproblems. LP makes MIP solvable; without LP as the backbone, MIP solving would be hopeless.

| problem class | objective type | variable type | complexity | canonical examples |
| --- | --- | --- | --- | --- |
| LP, IP, MIP, 0-1 IP, QP (convex | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. |
| QP (non-convex | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. |
| student sees the full family at a glance and can locate their problem in the right row | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. |

---

## Worked example: production planning at a refinery

Oil refineries have used LP since the 1950s; the early applications drove commercial LP solver development. The problem is canonical.

A refinery processes crude oil into finished products: gasoline, diesel, jet fuel, fuel oil, lubricants. Multiple crude types are available at different prices. Each crude can be processed by different distillation paths, each yielding different product ratios and subject to throughput constraints. Products must meet demand forecasts. The question: how much of each crude to run through each path, to maximize profit?

The LP formulation is direct. Decision variables $x_{ij}$ represent the amount of crude $i$ processed on path $j$. The objective maximizes revenue from products minus cost of crude minus processing costs. Constraints enforce: throughput on each distillation path, availability of each crude type, and satisfaction of demand for each product. Non-negativity ensures no negative quantities.

For a small refinery with 5 crude types, 8 processing paths, and 6 finished products, the LP has 40 variables and roughly 19 constraints — solved in milliseconds. Real refineries have hundreds of variables and constraints; solved in seconds.

Solving gives the optimal production schedule. But the dual gives something additional.

The shadow price on each capacity constraint says: if this distillation tower's capacity increases by one barrel per day, profit increases by $X$. That is the value of capital investment, computed without any additional analysis. The shadow price on each crude constraint says: this is the most we should pay above the market price for additional supply of this crude. The shadow price on each demand constraint says: this is what it costs, in reduced profit, to meet one more unit of demand.

Sensitivity analysis extends this. What if jet fuel demand rises 10%? Solve the LP again with the right-hand side changed, or use the dual to estimate the change linearly. What if a unit goes offline for maintenance, cutting path capacity to zero for a week? Resolve with that constraint tightened. The LP framework supports rapid what-if analysis because the dual is already computed.

The extensions that arise in practice are worth noting. Pooling problems — where intermediate streams are blended in tanks — introduce bilinear terms that make constraints non-linear. This breaks LP's polynomial-time guarantee and requires specialized solvers. Multi-period planning links periods by inventory constraints, creating a sequence of LPs. Integer decisions about starting or shutting down units create MIPs. The basic LP is tractable; the real-world extensions escalate toward MIP and non-linear programming, but LP is always in the backbone.

---

## The misconception about LP and its costs

"LP is for academics; production systems use heuristics." This is factually wrong about industrial practice and carries a practical cost: problems that could be solved optimally in seconds are instead solved by ad-hoc methods with unknown solution quality and no sensitivity information.

The industrial record is not ambiguous. Airline scheduling, refinery production, power-grid dispatch, supply-chain optimization, telecommunications network design, and financial portfolio construction are LP-driven at scale. The claim that production systems use heuristics instead of LP is true in the sense that some heuristics exist — and false in the sense that LP solvers run at industrial scale daily, on problems with millions of variables.

The deeper misconception is about what LP provides that a heuristic does not. A heuristic gives you a solution with no guarantee on its distance from optimal and no information about which constraints are limiting you. An LP gives you the optimal solution (for the LP problem class), the shadow prices quantifying the marginal value of each constraint, and the sensitivity range over which that solution remains optimal. This is not academic; this is the information that capital investment decisions and contract negotiations are built on.

The specific limits of LP are real and worth knowing. Non-linear constraints — pooling, blending, quadratic risk terms — break the LP structure and require QP, SOCP, or general convex programming. Integer requirements — scheduling discrete starts and stops, routing binary assignments — require IP. Uncertainty in the data requires stochastic or robust LP formulations. These are not reasons to avoid LP; they are reasons to know where LP ends and its relatives begin.

The corrective: when a problem looks like quantitative allocation under linear costs and constraints, formulate the LP. If integrality is essential, escalate to MIP. If the objective is quadratic, escalate to QP. Start with LP; escalate when the structure demands it.

---

## Decision rules

| Problem signature | Approach |
|---|---|
| Linear objective, linear constraints, continuous variables | LP: simplex or interior-point |
| Need shadow prices and sensitivity | Solve LP and read the dual |
| Linear constraints, integer variables | IP or MIP solver |
| Mix of continuous and integer | MIP solver |
| Need lower bound on IP optimum | LP relaxation |
| Need approximation from NP-hard IP | LP relaxation + rounding (Chapter 11) |
| Quadratic objective, convex, linear constraints | QP solver |
| Network flow problem | Specialized network simplex — LP-equivalent, faster in practice |
| Very large LP, one-shot | Interior-point |
| Repeated LP solves, small modifications | Simplex (warm-starts well) |
| Don't know if LP-shaped | Try writing constraints as $A\mathbf{x} \leq \mathbf{b}$; if they fit, solve as LP |

---

## What this chapter does not enable

This chapter does not enable implementing simplex or interior-point from scratch. Production solvers represent decades of careful engineering; the right move is to use them, not to reinvent them. The chapter does not cover column generation, Lagrangian relaxation, or Benders decomposition — techniques for very large LPs and MIPs that decompose the problem into tractable subproblems. For those, consult Bertsimas and Tsitsiklis's *Introduction to Linear Optimization* or Wolsey's *Integer Programming*. Stochastic and robust LP formulations are touched here but developed in operations research literature.

---

## Capability statement

You can now formulate a problem with a linear objective and linear constraints as an LP in standard form; choose between simplex and interior-point based on whether the problem benefits from warm-starting; read the dual solution as shadow prices and use them for sensitivity analysis; distinguish LP from IP, MIP, and QP and know which solver each requires; and recognize when LP relaxation provides an approximation bound or serves as the backbone of a branch-and-bound algorithm. The next time a problem arrives that looks like quantitative allocation under linear constraints — refinery, airline, supply chain, portfolio, power grid — the path from problem to LP to optimal solution and its sensitivity is in your hands. This chapter closes the optimization arc the volume opened with.

---

## Exercises

**Warm-up**

1. A company makes two products. Product A requires 3 hours of machine time and 2 kg of material per unit; product B requires 1 hour of machine time and 4 kg of material per unit. The machine is available for 120 hours per week; 200 kg of material is available. Product A sells for $5/unit and product B for $4/unit. Write this as a linear program in standard form — define the decision variables, write the objective, write the constraints. Do not solve it; just formulate it. *(Tests: converting a verbal description to standard LP form.)*

2. The simplex algorithm stops when all reduced costs are non-positive. Explain in geometric terms what a negative reduced cost means (what does it tell you about a neighboring vertex?) and why all non-positive reduced costs certifies global optimality, not just local. *(Tests: understanding the simplex stopping condition as a global certificate, not a local one.)*

3. Strong duality says the primal maximum equals the dual minimum. Complementary slackness says: at optimum, a constraint is either binding or its shadow price is zero. Give an intuitive explanation of why a slack constraint must have shadow price zero. What would it mean economically if a constraint with surplus capacity had a positive shadow price? *(Tests: understanding complementary slackness as an economic statement, not just an algebraic one.)*

**Application**

4. You solve a production-planning LP and find that the binding constraints are machine-hours (shadow price $4.50/hour) and raw material (shadow price $0). The material constraint has 30 units of slack. Management asks: "Should we invest in more machine capacity or more raw material?" Use the shadow prices to answer this question, and explain what the zero shadow price on raw material tells you about the current plan. *(Tests: reading dual variables as managerial information, not just algorithmic output.)*

5. A network flow problem (Chapter 9) can be formulated as an LP: decision variables are flow on each edge, the objective minimizes cost, and constraints enforce capacity and conservation. Write the LP formulation for a small network with 3 nodes and 4 edges (you may invent the numbers). Then identify which LP variables correspond to which edges and what the dual variables would represent physically. *(Tests: translating between a combinatorial problem and its LP formulation; connecting LP duality to Chapter 9.)*

6. The LP relaxation of the minimum vertex-cover integer program allows vertex variables to take fractional values between 0 and 1. If the LP relaxation returns a fractional solution, explain why rounding every value ≥ 0.5 up to 1 produces a valid integer solution. Then argue why this rounded solution is at most twice the size of the optimal integer solution. (This is the 2-approximation guarantee from Chapter 11 derived through LP.) *(Tests: connecting LP relaxation to approximation algorithm analysis.)*

**Synthesis**

7. Simplex has exponential worst-case complexity but is fast in practice. Interior-point has polynomial worst-case complexity but loses to simplex on warm-started re-solves. A branch-and-bound MIP solver solves thousands of LP relaxations, each slightly different from the last. Which algorithm should the MIP solver use for its LP subproblems, and why? What property of simplex makes it the right choice even though it has worse theoretical guarantees? *(Tests: applying the simplex/interior-point trade-off to a concrete algorithmic context.)*

8. A colleague says: "Our scheduling problem has 50 jobs, 5 machines, and binary assignment variables — it's an NP-hard MIP. We should use a heuristic." Walk through the argument for trying LP relaxation first: what does it give you that the heuristic does not, what is the LP relaxation of this specific problem, and under what conditions would the LP relaxation solution be usable directly without branch-and-bound? *(Tests: applying the LP-first principle to a real NP-hard problem; understanding when LP relaxation is sufficient.)*

**Challenge**

9. The chapter says LP duality is the general framework behind max-flow min-cut (Chapter 9). Sketch the argument: write the max-flow LP (objective: maximize total flow; constraints: edge capacities, flow conservation), write its dual, and identify which dual variables correspond to which edges and cuts. You do not need to derive the full duality proof — state the correspondence and explain why the dual of the max-flow LP should give you the min-cut value, given strong duality. *(Tests: connecting LP duality to a specific Chapter 9 result; reasoning across the book's optimization arc.)*

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
