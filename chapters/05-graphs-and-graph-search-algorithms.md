# Graphs and Graph Search Algorithms

## TL;DR

Graphs model relationships — vertices connected by edges with optional weights and direction. Reach for this chapter when a problem can be drawn as nodes and connections: maps, networks, dependencies, social structures, anything where "what connects to what" is the question. After consulting it, you can pick the right traversal (BFS, DFS), the right shortest-path algorithm (Dijkstra, Bellman-Ford, Floyd-Warshall, A\*), and recognize when a problem you have not yet drawn as a graph is in fact a graph problem.

## Recognition pattern

You have entities that connect to each other and a question whose answer involves those connections. The entities can be cities, computers, courses, web pages, friends, atoms, build tasks, or rooms in a maze. The connections can be roads, network links, prerequisites, hyperlinks, friendships, bonds, dependencies, or doors. If the question is about reachability ("can I get from A to B?"), shortest path ("what's the cheapest route?"), connectivity ("is everything connected?"), structure ("is there a cycle?"), or scheduling ("in what order must these tasks run?"), the problem is a graph problem.

The signal is *relationship as first-class*. If the data is a list and the operations are "for each item," it is not yet a graph. If the data has both items *and* a notion of which items relate to which, draw the graph. The drawing often makes the right algorithm obvious.

The second signal is *the problem looks like a problem you've seen on graphs before*. Bipartite matching, network flow, MST, topological sort, single-source shortest path — these are catalog problems. The art is recognizing your specific problem as one of them. The chapter's worked example walks through a road-network case where three different shortest-path algorithms apply for three different reasons.

A signal that this chapter is *not* the right one: the graph is small enough that brute force works. If you have a thousand nodes and need a single shortest path, write the cleanest implementation you can and ship it. The classical algorithms earn their place at scale.

## What you need to know first

This chapter assumes Big O (Chapter 2), heaps for Dijkstra's priority queue (Chapter 3 §4), and basic recursion (Chapter 7). For the DP framing of Bellman-Ford and Floyd-Warshall, see Chapter 8. For network-flow algorithms, which are graph algorithms but earn their own chapter, see Chapter 9.

## Representations

Two ways to store a graph.

**Adjacency list.** For each vertex, store a list of its neighbors (and edge weights, if any). Memory: `O(V + E)`. Iteration over a vertex's neighbors: `O(deg(v))`. Edge existence query: `O(deg(v))`. The default representation for sparse graphs.

**Adjacency matrix.** A `V × V` matrix where entry `[i][j]` is the weight of the edge from `i` to `j`, or 0/∞ if no edge. Memory: `O(V²)`. Edge existence query: `O(1)`. Iteration over neighbors: `O(V)`. The default for dense graphs and for algorithms like Floyd-Warshall that operate on the matrix directly.

Real-world graphs are almost always sparse — `E = O(V)` or `O(V log V)` rather than `O(V²)` — so the adjacency list is the dominant representation. The exception is when the graph is small (V < a few hundred) or when matrix operations are the algorithm.

A third representation, **edge list**, stores the graph as a flat list of `(u, v, w)` triples. Useful for Bellman-Ford and Kruskal's MST, both of which iterate over edges rather than vertices.

## Traversal — BFS and DFS

Two foundational traversals; almost every other graph algorithm builds on one of them.

**Breadth-First Search (BFS).** Explore vertices in order of distance from the source: source, then all vertices at distance 1, then all at distance 2, and so on. Implementation: a FIFO queue. Time: `O(V + E)`. The natural fit for *shortest path in unweighted graphs* — the first time BFS reaches a vertex, the path is shortest. Also the natural fit for *minimum-hops* problems on unweighted graphs (number of friendship hops between two people, fewest button presses to reach a state).

**Depth-First Search (DFS).** Explore as deep as possible from the source before backtracking. Implementation: a stack (or recursion). Time: `O(V + E)`. The natural fit for *cycle detection*, *topological sort*, *strongly connected components* (Tarjan's, Kosaraju's), and any problem with a tree-like decomposition.

The choice between them is rarely a tradeoff between performance — both are linear — but between *what you want to know*. If you want shortest paths in an unweighted graph, BFS. If you want structural properties (cycles, components, ordering), DFS.

## Shortest paths

Four canonical algorithms. Each fits a different problem shape.

**Dijkstra's algorithm.** Single-source shortest paths in a graph with non-negative edge weights. Developed by Edsger Dijkstra in 1956 [verify]. Maintain a priority queue of vertices keyed on tentative distance from source; repeatedly extract the closest unfinalized vertex and relax its outgoing edges. Time: `O((V + E) log V)` with a binary heap, `O(E + V log V)` with a Fibonacci heap. The Fibonacci heap improves the asymptotic bound but rarely wins in practice; the binary heap is the production choice (Chapter 3 §4).

The non-negativity constraint matters. With negative edges, Dijkstra can finalize a distance before discovering a shorter path through a negative edge. The algorithm is *correct* only when all weights are non-negative.

**Bellman-Ford algorithm.** Single-source shortest paths in a graph that may contain negative edges. Developed by Richard Bellman and Lester Ford Jr. in the 1950s [verify]. Iterate `|V|−1` times, relaxing every edge each iteration. Time: `O(VE)`. After `|V|−1` iterations, all shortest paths are correct *unless* a negative cycle exists; one more pass that further relaxes any edge proves the negative cycle. Bellman-Ford is the right call when negative weights are part of the model — currency arbitrage detection, certain network protocols, or weights that encode "rebates" alongside costs.

**Floyd-Warshall algorithm.** All-pairs shortest paths via dynamic programming. Roy 1959, Floyd 1962, Warshall 1962 [verify]. Initialize a distance matrix from edge weights; for each intermediate vertex `k`, update every pair `(i, j)` with `min(d[i][j], d[i][k] + d[k][j])`. Time: `O(V³)`. Space: `O(V²)`. Wins on dense graphs where you want every pair's distance and `V` is small enough (a few hundred to a few thousand). The DP framing is in Chapter 8.

**Johnson's algorithm.** All-pairs shortest paths in sparse graphs with possibly-negative edges. Donald Johnson, 1977 [verify]. Add a dummy vertex connected to every vertex with zero-weight edges; run Bellman-Ford from the dummy to compute vertex potentials; reweight edges to be non-negative; run Dijkstra from each vertex. Time: `O(VE log V)`. Beats Floyd-Warshall on sparse graphs.

**A\* (A-star).** Dijkstra with a heuristic. Maintain a priority queue keyed on `g(v) + h(v)`, where `g(v)` is the actual distance from source and `h(v)` is an admissible lower-bound estimate of the distance from `v` to the goal. When `h` is admissible (never overestimates) and consistent, A\* finds the optimal path. With a good heuristic, A\* explores far fewer vertices than Dijkstra. The classic heuristic for road networks is great-circle distance: a straight-line lower bound on travel time that aligns the search toward the goal.

## Minimum spanning trees

A minimum spanning tree (MST) of a connected weighted graph is a subset of edges that connects all vertices with minimum total weight. Three classical algorithms.

**Kruskal's algorithm.** Sort edges by weight; iterate, adding each edge unless it creates a cycle (use Union-Find, Chapter 3 §6, to detect cycles efficiently). Time: `O(E log E)`. Wins on sparse graphs.

**Prim's algorithm.** Grow the tree from a starting vertex; at each step, add the cheapest edge connecting a tree vertex to a non-tree vertex (use a priority queue, Chapter 3 §4). Time: `O(E log V)` with a binary heap. Wins on dense graphs.

**Borůvka's algorithm.** In each phase, every component finds its cheapest outgoing edge and merges. Time: `O(E log V)`. The oldest of the three (Borůvka, 1926) [verify]; useful in parallel implementations because the per-component work is independent.

The MST is the canonical greedy-on-matroid problem (Chapter 6); the proof of correctness is the cut-property exchange argument.

## Decision rules

| You want | Use | Time |
| --- | --- | --- |
| Reachability or shortest unweighted path | BFS | `O(V + E)` |
| Cycle detection, topological sort, SCC | DFS | `O(V + E)` |
| Single-source shortest path, non-negative weights | Dijkstra | `O((V+E) log V)` |
| Single-source with possibly-negative weights | Bellman-Ford | `O(VE)` |
| Single-source with a goal node and a heuristic | A\* | depends on heuristic |
| All-pairs, dense graph, small V | Floyd-Warshall | `O(V³)` |
| All-pairs, sparse graph, possibly-negative weights | Johnson's | `O(VE log V)` |
| MST, sparse graph | Kruskal | `O(E log E)` |
| MST, dense graph | Prim | `O(E log V)` |
| MST, parallel implementation | Borůvka | `O(E log V)` |
| Network flow / matching / min-cut | See Chapter 9 |  |

## Worked example — routing in a road network

A logistics service computes route times between distribution centers. The road network has 10 million vertices (intersections) and 25 million edges (road segments) [verify]; edge weights are travel times in seconds. The same problem is solved three ways, for three reasons.

**Dijkstra (basic).** Source: a specific distribution center. Target: another distribution center. Run Dijkstra from source, stop when target is finalized. Time on a binary heap: roughly proportional to `(V + E) log V`. On the network above, this is roughly `7 × 10⁸ log V ≈ 1.6 × 10¹⁰` heap operations [verify]. Slow but correct. The result: the optimal route in seconds.

**A\* with great-circle heuristic.** Same graph, same source, same target. The heuristic `h(v)` is the great-circle distance from `v` to the target, divided by the maximum permitted speed. This is admissible because no route can be faster than the straight-line distance at the maximum speed. The search now expands vertices in roughly straight-line direction toward the target rather than radiating outward. On real road networks, A\* with this heuristic explores roughly 1–10% of the vertices Dijkstra explores [verify]. Same answer, often 10× to 100× faster.

The heuristic earns its place when it correlates well with actual distance. For a flat-Earth approximation on continental scales, great-circle distance correlates strongly with road distance; A\* dominates. In a maze where straight-line distance is a poor predictor of actual cost (many walls, dead ends), the heuristic informs little and A\* degrades toward Dijkstra.

**Bellman-Ford (when toll pricing creates negative effective weights).** A pricing model offers rebates for taking certain toll roads at off-peak times — the rebate exceeds the toll and the resulting effective edge weight is negative. Dijkstra and A\* are now incorrect: both can finalize a vertex's distance before discovering the negative-effective-weight edge that would have given a cheaper route. Bellman-Ford handles this correctly in `O(VE)` time at the cost of being asymptotically slower. The pricing model breaks the assumption that justified Dijkstra; the algorithm choice tracks the change.

The lesson: the algorithm is determined by the constraints of the model, not by reflex. A change in pricing changes the model. A change in the model changes the right algorithm. Reading "Dijkstra is the shortest-path algorithm" without remembering the non-negative-weights constraint is the failure mode the next section engages.

## Failure modes — when "just use Dijkstra" misleads

The misconception engaged: "Just use Dijkstra."

Dijkstra is the right algorithm for single-source shortest path on a graph with non-negative edge weights and unbounded exploration budget. Outside that envelope, it fails or wastes work.

**Negative edges.** Dijkstra is incorrect on negative weights. The algorithm finalizes a vertex's distance the first time it is extracted from the priority queue; with negative edges, a shorter path via the negative edge can be discovered later, but the finalized distance is never updated. Use Bellman-Ford. The toll-pricing case in the worked example is one production instance.

**Negative cycles.** No shortest path is well-defined when a negative cycle exists; Bellman-Ford reports the cycle. Currency arbitrage detection is a positive use of this property — a negative cycle in the log-rate graph is an arbitrage opportunity.

**A goal node, but Dijkstra explores everywhere.** Plain Dijkstra computes shortest paths to *all* reachable vertices. If you have a single target, A\* with a good heuristic explores a fraction of the graph. The right call when the target is known and a heuristic is available.

**All-pairs query, sparse graph.** Running Dijkstra `V` times costs `O(V(V+E) log V)`. On sparse graphs with possibly-negative edges, Johnson's algorithm wins.

**All-pairs query, dense graph, small V.** Running Dijkstra `V` times costs `O(V² log V + VE)`. Floyd-Warshall is `O(V³)`, simpler, and often wins because the inner loop is a tight matrix update with excellent cache behavior.

**The graph fits in memory but the search dominates.** Bidirectional search runs two simultaneous searches from source and target; meeting in the middle reduces the explored region by roughly square-root [verify]. Useful as an A\* alternative or complement.

**The graph is huge and unweighted.** BFS, not Dijkstra. The priority queue overhead is unjustified when all weights are 1.

The corrective heuristic: state the problem (single-source vs all-pairs, presence of negative weights, presence of a heuristic, weighted vs unweighted), then pick the algorithm that matches. Dijkstra is the right answer for one specific problem shape; that shape covers many real cases but not all.

## Cross-references

For heap-based priority queues underlying Dijkstra and Prim, see Chapter 3 §4. For Union-Find underlying Kruskal, see Chapter 3 §6. For the DP framing of Bellman-Ford and Floyd-Warshall, see Chapter 8. For MST as a canonical greedy-on-matroid, see Chapter 6. For network flow extending the graph model with capacities, see Chapter 9. For randomized graph algorithms (Karger's min-cut), see Chapter 12.

## Companion-page handoffs

OSM-based routing demos with full code; comparative performance benchmarks across graph sizes and density; visualizations of BFS and DFS traversal orders; A\* with various heuristics on real road networks; SCC computation walkthroughs (Tarjan's and Kosaraju's); negative-cycle detection examples. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-5.

## What this chapter does not enable

This chapter does not cover advanced graph topics — graph isomorphism, planarity testing, treewidth, graph minors, spectral graph theory, hypergraphs. Those are research-level material; for graph isomorphism in particular, the breakthrough by Babai (2015) [verify] is at the frontier of theoretical algorithms. The chapter also does not cover the specific algorithms used at planetary scale by mapping services (contraction hierarchies, transit-node routing, customizable route planning); those are layered on top of the classical algorithms here and live in the routing-systems literature.

## Capability statement

You can now identify when a problem is a graph problem, choose between adjacency-list and adjacency-matrix representations, pick the right traversal for the question being asked, distinguish the four classical shortest-path algorithms by their preconditions, choose an MST algorithm given graph density, and recognize the failure modes of each. The next time a problem arrives that involves connections — between cities, computers, tasks, or people — the path from problem to algorithm is in your hands.


---

## LLM Exercise — Chapter 5: Graph Algorithms and a Routing Demo

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** Graph representations, the four classical shortest-path algorithms, MST trio, topological sort, and a routing demo that compares Dijkstra / A* / Bellman-Ford on the same problem.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 5 task in the algorithms-by-bear-toolkit. Bear's worked example
is routing in a road network where the same problem demands three
different shortest-path algorithms for three reasons. The exercise
reproduces that comparison.

In `chapters/ch05_graphs/`:

1. `implementations.py`:

   - `Graph` class supporting both adjacency-list and adjacency-matrix
     representations, switchable by constructor. Methods: `add_node`,
     `add_edge` (with optional weight), `neighbors`, `nodes`, `edges`.
     Use the `UnionFind` from Chapter 3 for Kruskal's MST.
   - Traversals: `bfs(graph, source)`, `dfs(graph, source)` (both
     iterative). Both return the visit order and a parent map.
   - Shortest paths: `dijkstra(graph, source)` (using `MinHeap` from
     Chapter 3), `bellman_ford(graph, source)` (returns
     `(distances, has_negative_cycle)`), `floyd_warshall(graph)` (all
     pairs), `astar(graph, source, target, heuristic_fn)`.
   - `topological_sort(graph)` for DAGs.
   - MST: `kruskal(graph)` and `prim(graph)`.

2. `test_implementations.py` — correctness on small hand-checkable
   graphs plus comparison against `networkx` (`pip install networkx`)
   on random graphs.

3. `benchmarks.py` — the routing study. Load a small road network
   (use `osmnx` to grab a city neighborhood — say a 2km×2km square of
   somewhere walkable; if `osmnx` is unavailable, generate a planar
   grid graph with 10000 nodes and randomized edge weights as a
   fallback). On 100 random source-target pairs, measure runtime for:

   - Dijkstra (baseline)
   - A* with great-circle distance heuristic (admissible for road
     networks)
   - Bellman-Ford (same data; reports the speedup penalty for
     supporting negative-weight edges you don't have)

   Plot runtime distributions. Report the speedup A* achieves over
   Dijkstra and the cost Bellman-Ford pays for generality.

4. `README.md` — decision cards for BFS, DFS, Dijkstra, Bellman-Ford,
   Floyd-Warshall, A*, Kruskal, Prim, topological sort.
   "Surprising findings": the A*/Dijkstra speedup on your real road
   data, and any case you found where the heuristic became
   inadmissible.

Commit with `ch05: graphs, traversals, shortest paths, routing demo`.
```

**What this produces:** Nine graph algorithms, a routing demo on real OSM data (or fallback grid), runtime distribution plots, and decision cards with empirically grounded speedup observations.

**How to adapt this prompt:**

- *For your own project:* If your work involves dependency graphs (build systems, software architecture, citation networks), replace the road-network demo with your own graph data — the algorithms are identical.
- *For ChatGPT / Gemini:* Network setup (`osmnx`) is finicky; if either model struggles, fall back to the synthetic grid and note the trade-off.
- *For Claude Code:* Native fit. Let it test on the small hand-checked graphs before unleashing on the real road network.
- *For a Claude Project:* Skip.

**Connection to previous chapters:** Imports `MinHeap` and `UnionFind` from Chapter 3, harness from Chapter 2.

**Preview of next chapter:** Chapter 6 implements interval scheduling, Kruskal's MST (already done!), and Huffman coding — and asks you to prove the exchange argument empirically by constructing adversarial inputs for the alternatives.
