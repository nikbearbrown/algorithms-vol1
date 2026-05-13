# Network Flow

## TL;DR

Network flow models problems as flow through a directed graph with edge capacities, source, and sink. Reach for this chapter when a problem can be reduced to "how much can I push from A to B given these limits?" — and many surprising problems can be. After consulting it, you can model a problem as a flow network, choose between Ford-Fulkerson, Edmonds-Karp, and push-relabel solvers, recognize the canonical reductions (bipartite matching, image segmentation, project selection), and use max-flow min-cut as both an algorithm and a proof technique.

## Recognition pattern

The signal: your problem has *capacities* on transit between stages, *demand* on the output, and you want the maximum throughput. Pipes carrying water. Roads carrying cars. Bandwidth on links. Items being assigned to workers. Pixels being labeled foreground or background. Any problem with the shape "stuff flows from sources to sinks through a graph with limits."

The deeper signal — the one this chapter argues makes flow non-niche: many problems that do not initially look like flow are *reductions* to flow. Bipartite matching is max-flow on a derived graph. Image segmentation is min-cut on a pixel graph with energy-based weights. Project selection (which projects to fund given prerequisites and budgets) is min-cut. Edge-disjoint paths is max-flow. The flow framework is a *modeling vocabulary* — once you can see your problem in it, the algorithm is in your hands.

A signal flow is *not* the right tool: the model is fundamentally non-linear (capacities depend on choices), or has constraints flow cannot express (this assignment forbids that one), or scales to billions of edges where solver constants dominate. In those cases, LP (Chapter 13) or specialized algorithms apply.

## What you need to know first

This chapter assumes graph fundamentals (Chapter 5) — vertices, edges, BFS for path-finding. Heaps (Chapter 3 §4) appear in some flow algorithms. For LP duality, which generalizes max-flow min-cut, see Chapter 13. For approximation via LP relaxation that can be rounded to integer flows, see Chapter 11.

## Flow networks — the model

A **flow network** is a directed graph `G = (V, E)` with:

- a **source** `s` and a **sink** `t`,
- a **capacity** `c(u, v) ≥ 0` for every edge `(u, v)`.

A **flow** `f` is a function on edges satisfying:

- *Capacity constraint:* `0 ≤ f(u, v) ≤ c(u, v)` for every edge.
- *Conservation:* for every vertex `v` other than `s` and `t`, the flow entering `v` equals the flow leaving `v`.

The **value** of a flow is the total flow leaving the source (equivalently, entering the sink). The **maximum flow** problem asks for the flow with greatest value.

A **cut** is a partition of `V` into `S` (containing `s`) and `T` (containing `t`). Its **capacity** is the total capacity of edges from `S` to `T`. The **minimum cut** is the cut with smallest capacity.

The central theorem of the field:

**Max-flow min-cut theorem (Ford-Fulkerson 1956)** [verify]: In any flow network, the maximum flow value equals the minimum cut capacity.

The theorem is the bridge between the algorithm and the proof. To prove a flow is maximum, exhibit a cut of equal capacity. To prove a cut is minimum, exhibit a flow of equal value. Every max-flow algorithm doubles as a min-cut algorithm: when the algorithm terminates, the residual graph's reachability from `s` defines the minimum cut directly.

## Ford-Fulkerson and its variants

**Ford-Fulkerson (1956).** Initialize flow to zero. Repeatedly find an augmenting path from `s` to `t` in the *residual graph* (the graph of edges with remaining capacity, plus reverse edges representing flow that can be cancelled). Augment along the path by the path's bottleneck capacity. Stop when no augmenting path exists.

The residual graph is what makes the algorithm work. Forward edges represent unused capacity; reverse edges represent flow that can be redirected. Without reverse edges, the algorithm could commit to a bad routing decision and never recover.

Termination and complexity depend on how augmenting paths are chosen. In the worst case with arbitrary path choice and integer capacities, Ford-Fulkerson runs in `O(E · F)` time where `F` is the max-flow value. With irrational capacities, it can fail to terminate at all [verify Zwick irrational example].

**Edmonds-Karp (1972) [verify].** Ford-Fulkerson with augmenting paths chosen by BFS (shortest path in edges). This eliminates the worst-case dependence on capacities: time is `O(V · E²)` regardless of capacity values. The improvement comes from a structural argument — the BFS-shortest path increases monotonically across iterations, bounded by `V`.

**Dinic's algorithm (1970) [verify].** Builds layered graphs and pushes blocking flows. Time: `O(V² E)`. On unit-capacity graphs, `O(E √V)`. The standard high-performance choice for general networks.

**Push-relabel (Goldberg-Tarjan 1988) [verify].** Maintains a *preflow* (excess at vertices is allowed) and pushes excess toward the sink while raising vertex labels. Time: `O(V² E)` or `O(V³)` with global relabeling. In practice, often the fastest algorithm on dense graphs.

In production, library solvers — NetworkX, OR-Tools, LEMON, Boost Graph Library — choose among these based on graph properties. Practitioners rarely implement flow from scratch; the production choice is which solver to call.

## Canonical reductions to flow

The reductions are the chapter's most practically useful content. When you can map your problem onto a flow network, the solver does the rest.

**Bipartite matching.** Given two sets `L` and `R` and edges between them, find the maximum matching. Reduction: add a source `s` connected to each vertex in `L` with capacity 1; add a sink `t` connected from each vertex in `R` with capacity 1; original edges from `L` to `R` have capacity 1. Max flow equals max matching. Time: `O(E √V)` with Dinic's.

**Edge-disjoint paths.** Given source `s` and sink `t`, find the maximum number of edge-disjoint paths from `s` to `t`. All edges have capacity 1; max flow equals max edge-disjoint paths. Menger's theorem (1927) [verify] is the graph-theoretic version of max-flow min-cut for this setting.

**Vertex-disjoint paths.** Same as edge-disjoint, but on a graph where each vertex is split into two — an "in" vertex and an "out" vertex connected by a unit-capacity edge. Max flow equals max vertex-disjoint paths.

**Image segmentation.** Worked example below.

**Project selection.** A set of projects, each with a profit (positive or negative) and prerequisites. Choose a subset maximizing profit subject to "if project A is in, all its prerequisites must be in." Reduction: source connected to positive-profit projects with capacity equal to profit; negative-profit projects connected to sink with capacity equal to absolute value; prerequisite edges have infinite capacity. The min-cut partitions projects into "selected" and "rejected"; the max profit is the sum of positive profits minus the min-cut value. Min-cut is computed by max-flow.

**Baseball elimination.** Determine whether a team can still win the division given remaining games. Reduces to a flow network where source feeds remaining games, games feed the teams that play them, and teams have capacity equal to remaining wins they need. Eliminated iff max flow saturates the source.

The pattern: build a graph where the structure of the problem maps to vertices, edges, and capacities; compute max-flow or min-cut; read the answer.

## Variants — min-cost flow and multi-commodity

**Min-cost flow.** Each edge has both a capacity and a cost per unit of flow. Find the maximum flow of minimum total cost. Algorithms: cycle-canceling, successive shortest paths, network simplex. Time depends on algorithm and instance; production solvers handle networks with millions of edges. Applications: transportation logistics, assignment problems with weighted preferences, supply-chain optimization.

**Multi-commodity flow.** Multiple source-sink pairs sharing edge capacities. NP-hard in the integer case (each commodity must take an integer amount on each edge); polynomial in the fractional case via LP (Chapter 13). The integer-vs-fractional distinction matters in practice: routing IP packets often requires integer flow per packet but allows fractional aggregate flow per commodity.

**Dynamic flow.** Time-dependent flows where capacities and demands change over time. Used in evacuation planning, traffic management. Beyond the scope of this chapter.

## Decision rules

| Problem signature | Algorithm or reduction |
| --- | --- |
| Maximum throughput, single source, single sink | Max flow (Edmonds-Karp, Dinic, push-relabel) |
| Min-cost throughput | Min-cost flow |
| Max bipartite matching | Reduce to max flow with unit capacities |
| Max edge-disjoint paths | Reduce to max flow, unit-capacity edges |
| Min number of vertex-disjoint paths | Reduce to max flow on split-vertex graph |
| Image segmentation (foreground/background) | Min-cut on pixel graph with energy weights |
| Project selection with prerequisites | Min-cut on project graph |
| Network reliability (k-edge connectivity) | Max-flow with k=connectivity |
| Multi-commodity, integer | NP-hard; use LP relaxation (Chapter 13) or heuristics |
| Multi-commodity, fractional | LP (Chapter 13) |
| Very large graph, want bound only | LP relaxation with rounding (Chapter 11) |
| Want exact and provably optimal on moderate graph | Library solver (OR-Tools, NetworkX) |

## Worked example — image segmentation via min-cut

The problem: segment an image into foreground and background. Each pixel must be labeled F or B. The labeling should respect *unary energies* (how likely each pixel is foreground or background based on its color) and *binary energies* (penalize boundaries between similarly-colored adjacent pixels — boundaries should fall along color edges, not in flat regions).

This is a *binary labeling* problem with submodular pairwise energies. Greig, Porteous, and Seheult (1989) [verify] showed it reduces to min-cut on a pixel graph; Boykov and Kolmogorov (2004) [verify] gave the standard fast algorithm used in computer vision today.

**Graph construction.** Vertices: every pixel, plus a source `s` (representing foreground) and a sink `t` (representing background).

- For each pixel `p`, add an edge from `s` to `p` with capacity equal to the negative log-likelihood that `p` is *background* (high capacity = strongly believes `p` is foreground).
- For each pixel `p`, add an edge from `p` to `t` with capacity equal to the negative log-likelihood that `p` is *foreground*.
- For each pair of adjacent pixels `(p, q)`, add edges `p → q` and `q → p` with capacity equal to a similarity score (high if `p` and `q` are similar in color, low if dissimilar).

**Min-cut.** Compute the minimum `s`-`t` cut. The cut partitions pixels into the side reachable from `s` (foreground) and the side reachable from `t` (background). Each pixel falls on exactly one side.

**Why this works.** The cut's capacity equals the sum of:
- Source edges going to background pixels (cost of labeling each background pixel as background — actually saved by labeling correctly).
- Sink edges coming from foreground pixels (cost of labeling each foreground pixel as foreground).
- Pixel-to-pixel edges crossing the F/B boundary (cost of the segmentation boundary, low when boundary follows color edges).

Minimizing the cut minimizes total energy. The min-cut algorithm finds the labeling jointly — every pixel's label is decided simultaneously based on the global structure, not pixel-by-pixel.

**Practical sizes.** A 1-megapixel image has 10⁶ vertices and roughly 4 × 10⁶ edges (4-connected) or 8 × 10⁶ (8-connected). The Boykov-Kolmogorov algorithm handles this in seconds on consumer hardware [verify]. Multi-megapixel medical images (CT, MRI volumes) push into 10⁸ vertices; specialized GPU implementations and graph reductions handle them.

**Production applications.**

*Medical imaging.* Tumor segmentation in CT/MRI. The energy weights encode radiologist priors and image features; the min-cut produces a segmentation a clinician can edit. [verify] specific clinical pipelines.

*Computer graphics.* Background removal in photo editing (Adobe Photoshop's "Select Subject" used graph-cut techniques in earlier versions before the deep-learning shift) [verify].

*Computer vision pipelines.* Pre-deep-learning segmentation (until roughly 2015) was dominated by graph-cut methods. Modern pipelines often combine deep-learning unary terms with graph-cut refinement.

The lesson: a problem that does not look like a flow problem — assigning each pixel a label, where neighborhood interactions matter — reduces cleanly to min-cut. The reduction is the algorithm. Once the graph is built, the solver does the rest.

## Failure modes — when "network flow is a niche topic" misleads

The misconception engaged: "Network flow is a niche topic."

This is a teaching artifact, not a reflection of practice. Flow models permeate optimization. Three concrete responses to the misconception.

**Bipartite matching is everywhere.** Job scheduling (workers to tasks), course registration (students to courses), kidney exchange (donors to recipients), advertising allocation (ads to slots). All are matching problems; matching reduces to max-flow. A practitioner who cannot see "this is a matching" misses an entire toolbox.

**Min-cut is a modeling primitive, not a niche algorithm.** Image segmentation, project selection, network reliability, MRF inference with submodular pairwise terms — all reduce to min-cut. The min-cut algorithm itself is a few hundred lines of code in a solver; the work is the modeling.

**Flow as proof technique.** Even when not used algorithmically, max-flow min-cut is a proof technique. König's theorem (max bipartite matching = min vertex cover in bipartite graphs) [verify], Menger's theorem (max edge-disjoint paths = min edge cut), Hall's marriage theorem — all are corollaries of max-flow min-cut on appropriate graphs. The duality propagates beyond the algorithm.

**Flow algorithms are well-supported.** OR-Tools, NetworkX, LEMON, Boost Graph Library — flow is solved. The practitioner does not implement; they call. The barrier is not algorithmic depth; it is recognizing that the problem in front of them is a flow problem.

**Negative cases — when flow is genuinely the wrong tool.** Routing with quality-of-service constraints (latency bounds along paths) is not flow, because path-level constraints do not factor into edge-level constraints. Game-theoretic equilibrium routing (selfish agents) is congestion games, not flow. Time-varying capacities sometimes fit dynamic flow but often need ad hoc methods. The misconception is "flow is rare"; the corrective is "flow is a frame, not a niche."

## Cross-references

For graph fundamentals (representations, BFS, DFS), see Chapter 5. For LP duality as the generalization of max-flow min-cut, see Chapter 13 §4. For LP relaxation as approximation technique that often involves flow, see Chapter 11. For randomized algorithms in graph problems (Karger's min-cut), see Chapter 12.

## Companion-page handoffs

Solver comparisons (NetworkX, OR-Tools, LEMON, Boost) on benchmark instances; image segmentation demo with code (Boykov-Kolmogorov implementation); bipartite matching examples; project selection walkthrough; baseball elimination computation; multi-commodity flow LP formulation. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-9.

## What this chapter does not enable

This chapter does not enable implementing high-performance flow solvers from scratch. The Boykov-Kolmogorov, push-relabel, and network-simplex codes in production solvers are the result of decades of optimization and benchmarking; reimplementing them is not the right call for almost any practical problem. The chapter also does not cover advanced flow topics — generalized flows (with edge gains), parametric flow, submodular flow, fractional packing — which live in operations research literature.

## Capability statement

You can now model a problem as a flow network when one applies, choose between Ford-Fulkerson, Edmonds-Karp, Dinic's, and push-relabel solvers, recognize the canonical reductions (bipartite matching, image segmentation, project selection, edge-disjoint paths), and apply max-flow min-cut as both an algorithm and a proof technique. The next time a problem looks like "stuff has to flow under limits," or "two sets need to be matched," or "labels must be assigned with neighborhood interactions," the path from problem to solver is in your hands.


---

## LLM Exercise — Chapter 9: Network Flow and the Bipartite Matching Reduction

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** Max-flow algorithms (Ford-Fulkerson, Edmonds-Karp), the bipartite-matching reduction, and an image-segmentation demo that turns foreground/background separation into min-cut.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 9 task in the algorithms-by-bear-toolkit. Bear's argument is
that network flow is non-niche because *many problems reduce to it* —
the chapter's most useful section is the reduction patterns. The
exercise implements the algorithms and then proves the reduction
claim by solving two non-flow problems through flow.

In `chapters/ch09_network_flow/`:

1. `implementations.py`:

   - `FlowNetwork` class — directed graph with `add_edge(u, v,
     capacity)`. Maintains a residual graph internally. Methods:
     `bfs_augmenting_path(source, sink)`, `dfs_augmenting_path`.
   - `ford_fulkerson(network, source, sink)` — uses DFS augmenting
     paths.
   - `edmonds_karp(network, source, sink)` — uses BFS augmenting
     paths (the polynomial-time version).
   - `min_cut(network, source, sink)` — returns the cut edges after
     a max-flow computation.
   - `bipartite_matching(left_nodes, right_nodes, edges)` — reduces
     to max-flow on a derived network and returns the matching.
   - `image_segmentation(image, foreground_seeds, background_seeds,
     similarity_fn)` — reduces foreground/background labeling to
     min-cut on a pixel graph.

2. `test_implementations.py`:

   - Correctness of `ford_fulkerson` and `edmonds_karp` against a
     known-result `networkx.maximum_flow` reference on random
     graphs.
   - Bipartite matching against `networkx.bipartite.maximum_matching`.
   - Image segmentation: a small 16×16 synthetic image where the
     foreground is a circle on a contrasting background; verify the
     segmentation recovers the circle.

3. `benchmarks.py` — two studies:

   **Study A: FF vs. EK pathology.** Construct the textbook
   pathological case where Ford-Fulkerson with DFS does O(maxflow)
   augmentations and Edmonds-Karp with BFS does O(VE) — the case
   with two capacity-1000000 edges and one capacity-1 edge that
   forces zigzag augmentation. Time both. Confirm the asymptotic
   difference shows up.

   **Study B: bipartite matching at scale.** Generate random
   bipartite graphs at varying sizes and densities. Time
   `bipartite_matching` (flow-based) against a Hopcroft-Karp
   reference (use `networkx`). Plot relative performance.

4. `README.md` — decision cards for max-flow algorithms, bipartite
   matching, min-cut, image segmentation. "Surprising findings":
   the magnitude of the FF/EK gap on the pathological case (it
   should be dramatic — capture the actual numbers).

Commit with `ch09: network flow with bipartite-matching and image-
segmentation reductions`.
```

**What this produces:** Three flow algorithms, two reduction-based applications, and a synthetic image-segmentation demo. The image segmentation is the structural payoff — showing that a "vision problem" is in fact a flow problem.

**How to adapt this prompt:**

- *For your own project:* If you work in resource allocation, replace the synthetic image with a real assignment problem from your domain (drivers to routes, ads to slots, applicants to slots).
- *For ChatGPT / Gemini:* The residual-graph bookkeeping is error-prone; insist on a `networkx` cross-check for every test case.
- *For Claude Code:* Native fit. The image-segmentation demo is the most fun; let it generate visualizations.
- *For a Claude Project:* Skip.

**Connection to previous chapters:** Imports `Graph` from Chapter 5 (or composes from it); imports `harness` from Chapter 2.

**Preview of next chapter:** Chapter 10 confronts NP-completeness — building reductions, implementing SAT-style encoders, and demonstrating what to actually *do* when a problem you face is NP-hard.
