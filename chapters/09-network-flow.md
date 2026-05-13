# Chapter 9 — Network Flow

*The same mathematics that moves water through pipes moves workers to jobs, pixels to labels, and money through constraints. Once you can see the shape, you have the whole toolbox.*

---

Here is a question that sounds like it belongs in a civil engineering textbook.

You have a network of pipes. Water enters at a source and exits at a sink. Each pipe has a maximum flow rate — a capacity. How much water can you push from source to sink?

That question has a clean mathematical answer, an efficient algorithm, and a theorem connecting the two that is one of the most elegant results in all of combinatorics. None of that would justify a chapter in an algorithms book except for one thing: an enormous number of problems that have nothing to do with pipes or water have exactly the same mathematical structure. Once you recognize the structure, the algorithm is already in your hands.

This chapter is about learning to see the structure.

---

## The model

A flow network is a directed graph with a source, a sink, and a capacity on every edge. A flow assigns a value to each edge — how much is flowing on it — subject to two constraints.

The first constraint is obvious: you cannot push more flow through an edge than its capacity. The second constraint is the one that makes the model interesting: at every vertex other than the source and sink, flow in equals flow out. Nothing is created or destroyed in the middle of the network. The source produces; the sink absorbs; everything in between conserves.

The value of a flow is the total amount leaving the source, which equals the total amount entering the sink. The maximum flow problem asks: given this network, what is the largest possible flow value?

A cut is a partition of the vertices into two sets, one containing the source and one containing the sink. The capacity of a cut is the total capacity of edges going from the source-side to the sink-side. (Edges going the other direction don't count — they go against the flow.) The minimum cut is the cut whose capacity is as small as possible.

These two things — maximum flow and minimum cut — are the same number. This is the max-flow min-cut theorem.

<!-- → IMAGE: small labeled flow network (5–6 vertices, source s, sink t, labeled capacities) with two panels — left panel shows a valid flow annotated on each edge (e.g. "3/5" for 3 units on a capacity-5 edge), right panel highlights one minimum cut as a dashed line with its crossing edges labeled; caption should state the max-flow value and the min-cut capacity as equal numbers, making the theorem concrete before the proof -->

---

## Why max-flow equals min-cut

The equality is not obvious. Maximum flow is an optimization over flows — infinitely many possible assignments of values to edges. Minimum cut is an optimization over partitions — finitely many but exponentially large set of possible divisions of the vertices. That these two different optimizations over two different objects give the same number is the kind of result that makes you stop and think.

Here is the intuition. A cut is a bottleneck. If every path from source to sink must cross the cut, then no flow can exceed the cut's capacity — you cannot push more water through a bottleneck than the bottleneck allows. So the minimum cut is an upper bound on the maximum flow. The min-cut value is at least as large as the max-flow value.

The other direction is the surprising one: the maximum flow is always large enough to equal the minimum cut. Put differently: you can always find a flow that completely saturates some cut. There is no slack, no wasted capacity in the bottleneck, no way to get more flow by rerouting cleverly.

The proof is constructive — the algorithm that finds the maximum flow also, when it terminates, identifies the minimum cut. At termination, some vertices are reachable from the source in the residual graph and some are not. The reachable set is the source side of the minimum cut. The capacity of the edges crossing from reachable to unreachable exactly equals the flow value. QED.

---

## The residual graph

The algorithms for maximum flow all work by augmentation: find a path from source to sink, push as much flow as possible along it, repeat until no more augmenting paths exist. The subtlety is what "path" means.

The residual graph is the key idea. Given a current flow, the residual graph has two kinds of edges. Forward edges represent unused capacity: if edge $(u, v)$ has capacity $c$ and current flow $f < c$, the residual graph has a forward edge from $u$ to $v$ with residual capacity $c - f$. Reverse edges represent cancellable flow: the same edge contributes a reverse edge from $v$ to $u$ with capacity $f$. Pushing flow along a reverse edge means "undo some of the flow you committed to this edge earlier."

The reverse edges are what make the algorithms correct. Without them, an algorithm might commit to a routing early on that blocks the optimal routing, with no way to recover. The reverse edges allow any decision to be partially or completely undone. The algorithm is free to explore without fear of irreversible mistakes.

<!-- → IMAGE: side-by-side diagram of original graph and residual graph for the same partial flow — original graph shows edges with capacities and current flow values; residual graph shows the same edges split into forward residuals (unused capacity, same direction) and reverse residuals (cancellable flow, opposite direction); one edge should be fully saturated so its forward residual is 0 and only the reverse residual exists — student should see why the reverse edge is the "undo" mechanism -->

An augmenting path is a path from source to sink in the residual graph. Augmenting along it means pushing flow equal to the minimum residual capacity along the path — the bottleneck — and updating the residual graph accordingly.

When no augmenting path exists, the algorithm is done. The set of vertices reachable from the source in the residual graph, together with the rest, is the minimum cut.

---

## Three algorithms, one framework

The augmentation framework is the same across all max-flow algorithms. What differs is how augmenting paths are chosen.

**Ford-Fulkerson** uses any augmenting path — typically found by depth-first search. It terminates when no path exists. With integer capacities, it terminates in at most $F$ augmentations where $F$ is the max-flow value: each augmentation increases the flow by at least 1. So the worst-case complexity is $O(E \cdot F)$, which can be terrible if $F$ is large. With irrational capacities, Ford-Fulkerson can fail to terminate at all — it can cycle, making smaller and smaller improvements, converging to the wrong answer. The algorithm is the conceptual foundation, not the production implementation.

**Edmonds-Karp** fixes the termination problem by choosing augmenting paths via BFS — shortest paths in terms of number of edges. A structural argument shows that BFS paths cause the length of augmenting paths to never decrease across iterations, and this bounds the number of augmentations at $O(VE)$ regardless of capacity values. Total complexity: $O(VE^2)$. This is polynomial in the graph size, independent of flow values, and the correct algorithm to reach for when you need a general-purpose implementation without special structure to exploit.

**Dinic's algorithm** improves on Edmonds-Karp by building a layered graph — a BFS shortest-path DAG — and pushing a *blocking flow* through it before rebuilding the layers. A blocking flow saturates at least one edge on every source-to-sink path in the layered graph. Total complexity: $O(V^2 E)$. On unit-capacity graphs — bipartite matching, edge-disjoint paths — it achieves $O(E \sqrt{V})$, which is the asymptotically best known for that special case.

**Push-relabel** abandons the augmenting-path framework entirely. Instead of pushing flow along complete source-to-sink paths, it maintains a preflow — excess is allowed at intermediate vertices — and pushes excess locally from higher-labeled vertices to lower-labeled ones, like water flowing downhill. It is the algorithm of choice on dense graphs in practice, with complexity $O(V^2 E)$ or $O(V^3)$ depending on implementation.

In practice, you call a library — NetworkX, OR-Tools, LEMON, Boost Graph Library. These implement Dinic's or push-relabel with decades of engineering. The reason to understand the algorithms is to reason about their behavior, not to reimplement them.

---

## The reduction toolkit

This is the section that justifies everything above.

A reduction takes a problem you want to solve and maps it to a flow network whose max-flow or min-cut encodes the answer. The solver does the rest. Once you can see your problem as flow, you have the combined power of decades of algorithmic engineering working for you.

**Bipartite matching.** You have two sets of vertices — workers and jobs, students and courses, donors and recipients — and edges between them indicating compatibility. You want the maximum matching: the largest set of edges with no shared endpoint.

The reduction: add a source with an edge of capacity 1 to every vertex on the left. Add edges of capacity 1 from every vertex on the right to a sink. Keep the original edges between left and right with capacity 1. Maximum flow equals maximum matching.

Why? Each unit of flow corresponds to a matched pair: it enters from the source, passes through a left vertex, crosses a compatibility edge, passes through a right vertex, and exits at the sink. The unit capacities ensure no vertex is used twice. The max-flow min-cut theorem guarantees the maximum flow is the maximum matching.

Bipartite matching is not an exotic problem. Job scheduling, course registration, ad slot allocation, kidney exchange, stable marriage. Any problem where two sets of things must be paired under compatibility constraints is a matching problem, and matching is max-flow.

<!-- → IMAGE: bipartite matching reduction diagram — left column shows workers (L1, L2, L3), right column shows jobs (R1, R2, R3, R4), compatibility edges between them; then the derived flow network adds source s with unit-capacity edges to each Li and sink t with unit-capacity edges from each Rj; a highlighted flow path s→L2→R3→t shows one matched pair; caption should note that each unit of flow is exactly one matched pair -->

**Edge-disjoint paths.** Given a source and sink in a graph, how many paths from source to sink can you find such that no two paths share an edge? Set every edge capacity to 1. Maximum flow equals the maximum number of edge-disjoint paths. This is Menger's theorem (1927): the maximum number of edge-disjoint paths equals the minimum edge cut — the minimum number of edges whose removal disconnects the source from the sink. Max-flow min-cut gives you both the paths and the minimum cut.

**Vertex-disjoint paths.** The same problem, but no two paths may share a vertex. The trick: split every vertex into an "in" copy and an "out" copy, connected by a single unit-capacity edge. All incoming edges go to the in copy; all outgoing edges leave from the out copy. The unit-capacity internal edge forces each vertex to carry at most one unit of flow. Maximum flow on the modified graph equals the maximum number of vertex-disjoint paths.

<!-- → IMAGE: vertex splitting diagram — original graph (4–5 vertices) on left, split-vertex graph on right; one vertex v shown split into v_in and v_out with a single unit-capacity internal edge; all original incoming edges go to v_in, all outgoing edges leave from v_out; caption should emphasize that the internal edge is the constraint — at most one path can pass through this vertex -->

**Project selection.** This one is less well-known but genuinely useful. You have a set of projects. Each has a profit — which can be negative, meaning it costs money. Some projects require others as prerequisites: you cannot do project A without doing project B first. Which projects do you select to maximize total profit?

This is a min-cut problem. Build a graph: source to each positive-profit project with capacity equal to the profit; each negative-profit project to sink with capacity equal to the absolute value; prerequisite edges with infinite capacity (forcing: if A is selected and A requires B, B must be selected too). Maximum profit equals the sum of positive profits minus the min-cut value. The cut partitions projects into selected and rejected; the min-cut chooses the partition that minimizes the "loss" from rejected positive-profit projects and included negative-profit projects.

**Image segmentation.** Each pixel must be labeled foreground or background. Two kinds of evidence constrain the labeling: unary evidence (a pixel's color makes it more or less likely to be foreground on its own) and pairwise evidence (adjacent pixels of similar color should get the same label; the boundary should fall along color edges, not within flat regions).

Build a graph: source and sink represent foreground and background. Each pixel has two edges — one from source (encoding its likelihood of being foreground) and one to sink (encoding its likelihood of being background). Adjacent pixels are connected with edges encoding similarity: high capacity for similar-color neighbors (penalizing a boundary between them), low capacity for dissimilar-color neighbors (the boundary belongs here anyway).

The minimum cut partitions pixels into the source side (foreground) and sink side (background). It minimizes total energy: the cost of labeling each pixel incorrectly relative to its unary evidence, plus the cost of boundaries between pixels that "should" be the same label. Every pixel's label is decided simultaneously, respecting global structure.

This reduction is not an academic exercise. Before the deep learning era, graph-cut segmentation was the dominant method in computer vision and medical imaging. Tumor segmentation in CT scans, object extraction in photographs, stereo vision disparity estimation — all ran on this framework. A 1-megapixel image produces a graph with roughly 5 million edges, and the Boykov-Kolmogorov algorithm handles it in seconds on consumer hardware. The problem looks nothing like flow at first glance. The reduction makes it flow.

<!-- → IMAGE: two-panel image segmentation diagram — left panel shows a small grid of pixels (6×6) colored light/dark, with foreground seeds marked F and background seeds marked B; right panel shows the corresponding flow network: source s connecting to foreground-likely pixels, dark pixels connecting to sink t, and neighbor edges between adjacent pixels; the min-cut shown as a dashed line separates the two regions; caption should note that the cut boundary falls along the color edge, not through the flat region -->

---

## Max-flow min-cut as proof technique

The theorem is useful beyond the algorithm. When you want to prove a combinatorial bound — "the maximum number of X equals the minimum size of Y" — max-flow min-cut is often the tool.

König's theorem: in a bipartite graph, the size of the maximum matching equals the size of the minimum vertex cover (a set of vertices touching every edge). This is not an obvious claim. The proof: formulate the matching as a flow network; observe that the min-cut corresponds to a minimum vertex cover; the equality follows from max-flow min-cut.

Hall's marriage theorem: a bipartite graph has a perfect matching if and only if, for every subset $S$ of left vertices, the neighborhood $N(S)$ satisfies $|N(S)| \geq |S|$. The "only if" direction is obvious — if some set of $k$ left vertices has fewer than $k$ neighbors, you cannot match all of them. The "if" direction is the content. It follows from max-flow min-cut applied to the bipartite matching construction: if Hall's condition holds, no cut can block a perfect matching.

Menger's theorem: the maximum number of edge-disjoint paths from $s$ to $t$ equals the minimum edge cut. This is exactly max-flow min-cut on a unit-capacity graph.

The pattern: state the duality you want to prove. Express it as a flow network. The theorem gives you the equality. These classical results, which took original proofs of considerable ingenuity to establish directly, all follow in a few lines from the same theorem.

---

## When flow is the wrong tool

The power of flow networks is real, but so are their limits.

Flow models additive constraints on edges. Routing problems with path-level constraints — "the latency along this path must be under 10ms" — cannot be expressed as edge capacities because latency depends on the whole path, not the individual edges. These require more general optimization.

Integer flow is harder than it looks. For single-commodity flow, max-flow with integer capacities always produces an integer maximum flow (the integrality theorem), and the algorithms naturally produce integer solutions. For multi-commodity flow — multiple source-sink pairs sharing edge capacities — the integer version is NP-hard. The fractional version (where you can split flow arbitrarily across routes) is a linear program and solvable in polynomial time, but rounding fractional solutions to integers is not always possible without loss.

Non-submodular pairwise energies in image segmentation break the min-cut reduction. The graph-cut method works because the energy function is submodular — there is an alignment between the energy minimization and the cut structure. When pairwise energies are non-submodular (for example, a "contrast-preserving" energy that penalizes adjacent pixels with *different* labels), the reduction fails. Approximate methods take over.

Game-theoretic routing — where each agent selfishly minimizes their own latency — produces congestion games, not flow networks. The equilibrium conditions are different from the conservation constraints.

These are not arbitrary limitations. They reflect the structure that makes flow powerful: it is a linear model of additive constraints. When the problem is linear and additive, flow is often optimal. When it is not, look elsewhere.

---

## What the toolbox actually contains

Here is the summary of what this chapter puts in your hands.

You can model a throughput problem as a flow network and choose among Ford-Fulkerson, Edmonds-Karp, Dinic's, and push-relabel based on graph structure. You can recognize bipartite matching, edge-disjoint paths, vertex-disjoint paths, project selection, and image segmentation as flow reductions and construct the corresponding networks. You can read off the minimum cut from a terminated max-flow computation and use it as either an algorithm output or a proof tool. You can invoke the max-flow min-cut theorem to establish König's, Hall's, and Menger's as corollaries.

The deeper thing: you have a new way of seeing problems. When a problem has capacities on transit between stages, or requires partitioning a graph to minimize crossing costs, or matches two sets under compatibility constraints — the flow frame is available to you. The algorithm is in the library. The work is the modeling.

The next chapter confronts NP-completeness — the class of problems where no polynomial-time algorithm is known, and where the question shifts from "which solver do I call?" to "what do I do when the problem is hard?"

---

## Exercises

### Warm-up

**1.** A flow network has the following edges and capacities: s→A (10), s→B (8), A→C (5), A→D (7), B→C (6), B→D (4), C→t (9), D→t (8). Find a maximum flow by hand — trace augmenting paths and show the flow on each edge at termination. Then identify the minimum cut and verify that the cut capacity equals the flow value.
*(Tests: mechanical execution of augmenting-path flow, min-cut identification from residual graph.)*

**2.** Given the flow network from Exercise 1, draw the residual graph after you have found the maximum flow. For each original edge, label both the forward residual and the reverse residual. Identify which edges are fully saturated (zero forward residual) and explain what the reverse residuals on those edges represent.
*(Tests: residual graph construction; understanding reverse edges as "undo" capacity.)*

**3.** You have four workers (W1–W4) and four jobs (J1–J4). The compatible assignments are: W1→{J1,J2}, W2→{J2,J3}, W3→{J3,J4}, W4→{J1,J4}. Construct the flow network for the bipartite matching reduction (draw it, label all capacities), run max-flow, and state the maximum matching. Is a perfect matching possible?
*(Tests: bipartite matching reduction construction and interpretation.)*

---

### Application

**4.** The Ford-Fulkerson algorithm with arbitrary path selection can perform $O(E \cdot F)$ augmentations where $F$ is the max-flow value. Edmonds-Karp with BFS paths runs in $O(VE^2)$ regardless of $F$. Construct a specific small flow network (you may use the textbook "bad case" with two large-capacity paths and a small bridge) and explain step by step how Ford-Fulkerson with DFS can be forced into $F$ augmentations while Edmonds-Karp takes far fewer. State the flow value you chose and the number of augmentations each algorithm performs.
*(Tests: understanding why path selection determines asymptotic complexity; the DFS/BFS distinction is not cosmetic.)*

**5.** You have five projects with the following profits and prerequisites:

   - A: profit +20, no prerequisites
   - B: profit +15, no prerequisites
   - C: profit −8, requires A
   - D: profit −5, requires A and B
   - E: profit +12, requires C and D

   Construct the project selection flow network. Label all edge capacities. Then compute the minimum cut and state which projects should be selected and the resulting maximum profit. Show your work.

*(Tests: project selection reduction — building the graph correctly, identifying the cut, reading the answer.)*

**6.** A communication network has six nodes and the following directed edges (with capacities): s→1 (3), s→2 (4), 1→3 (3), 1→4 (2), 2→3 (2), 2→4 (3), 3→t (4), 4→t (4). Find the maximum flow and the minimum cut. Now suppose edge 2→4 fails (its capacity drops to 0). Without recomputing from scratch, explain which augmenting paths are invalidated and how you would update the residual graph to find the new maximum flow efficiently.
*(Tests: incremental flow reasoning; understanding that the residual graph encodes existing decisions that can be partially preserved.)*

---

### Synthesis

**7.** A graduate school admissions office must assign 30 admitted students to one of 5 research groups, where each group can accept at most 8 students. Each student has ranked their top 3 group preferences. The office wants to maximize the number of students who receive a first-choice placement. Model this as a flow network. Describe all vertices, edges, and capacities precisely. What does the max-flow value tell you? What does the minimum cut tell you?
*(Tests: formulating a novel matching problem as flow; interpreting both the max-flow and min-cut in problem terms.)*

**8.** Hall's marriage theorem says a bipartite graph has a perfect matching if and only if for every subset $S$ of left vertices, $|N(S)| \geq |S|$ (the neighborhood is at least as large as the subset).

   - (a) Construct a bipartite graph on 4+4 vertices that violates Hall's condition and explain exactly which set $S$ witnesses the violation.
   - (b) Explain how the violation maps to a min-cut in the flow network: what does the min-cut look like, and why does its capacity fall below $|L|$ (the number of left vertices) precisely when Hall's condition fails?

*(Tests: connecting Hall's theorem to the max-flow min-cut proof structure; making the duality concrete rather than abstract.)*

---

### Challenge

**9.** Multi-commodity flow: Suppose you have a network with two source-sink pairs sharing the same edges — commodity 1 from $s_1$ to $t_1$, commodity 2 from $s_2$ to $t_2$. Each edge has a shared capacity (the total flow of both commodities must not exceed it). You want to maximize the total flow of both commodities combined.

   - (a) Explain why this problem cannot be solved by running two independent max-flow computations.
   - (b) Explain why the fractional version (where flow can be split arbitrarily) is solvable in polynomial time via LP, while the integer version (each commodity takes integer amounts on each edge) is NP-hard.
   - (c) Describe a small instance where the fractional optimum is strictly greater than any feasible integer flow — that is, the LP relaxation is not tight.

*(Tests: understanding the boundary of what flow can handle; the integer vs. fractional distinction; why multi-commodity breaks the integrality theorem.)*

**10.** Design a flow reduction for the following problem, or prove that no clean flow reduction exists and identify exactly what structural property of the problem prevents it.

   *Latency-constrained routing.* You have a directed graph where each edge has both a capacity and a latency. You want to route as much flow as possible from source to sink subject to the constraint that every unit of flow must travel along a path whose total latency is at most $L$.

   Attempt the reduction: can you express the latency constraint as edge capacities or modified flow conservation? If yes, construct the network. If no, identify the specific property of the constraint (path-level vs. edge-level, additive vs. non-additive, etc.) that breaks the flow model and name what class of problem this becomes.

*(Tests: understanding the scope and limits of flow as a modeling framework; distinguishing what flow can and cannot represent.)*

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
