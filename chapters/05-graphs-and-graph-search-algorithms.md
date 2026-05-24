# Chapter 5 — Graphs and Graph Search Algorithms

*What it means to ask "can I get there from here?"*

---

Here is a question that sounds like it belongs to geography but doesn't: you have a collection of things, and some pairs of those things are connected. Can you get from this thing to that thing? What's the cheapest way? Is everything reachable from everywhere else?

The things can be cities, computers, courses, web pages, people, atoms, build tasks, or rooms in a maze. The connections can be roads, cables, prerequisites, hyperlinks, friendships, chemical bonds, dependencies, or doors. The question is always the same: what does the structure of the connections tell you?

That structure has a name. It's called a graph — not in the sense of a chart or a plot, but in the mathematical sense: a set of vertices connected by edges. And the reason graphs matter is not that they're a clever abstraction but that an enormous class of real problems, problems that look completely different on the surface, have exactly the same underlying structure. Once you see that structure, the algorithm follows almost inevitably.

![Panel showing four visually distinct domains ](images/05-graphs-and-graph-search-algorithms-fig-01.png)
*Figure 5.1 — Panel showing four visually distinct domains *

---

## What a Graph Actually Is

A graph is vertices and edges. Vertices are the things; edges are the connections between them. That's the entire definition.

Edges can be *directed* — the connection goes one way. A web page can link to another page without being linked back. A course can be a prerequisite for another without the reverse being true. Directed graphs are sometimes called digraphs.

Edges can be *weighted* — the connection has a cost or capacity. A road has a travel time. A network link has a bandwidth. A financial transaction has a fee. When weights are present, the question shifts from "can I get there" to "what's the cheapest way to get there."

Edges can be both directed and weighted, or neither. The algorithm you use depends on what you have.

Two ways to represent a graph in memory, and the choice matters for performance.

The **adjacency list** stores, for each vertex, a list of its neighbors (and edge weights, if any). Memory is proportional to the number of vertices plus the number of edges — `O(V + E)`. Iterating over a vertex's neighbors takes time proportional to how many neighbors it has. This is the right representation when the graph is *sparse* — when most vertices connect to relatively few others. Real-world graphs are almost always sparse: a road network has millions of intersections but each intersection connects to only a handful of roads.

The **adjacency matrix** stores a `V × V` grid where entry `[i][j]` holds the weight of the edge from vertex `i` to vertex `j`, or infinity if no edge exists. Memory is proportional to the square of the number of vertices — `O(V²)`. Checking whether an edge exists between two specific vertices is instant, `O(1)`. But iterating over all neighbors of a vertex takes `O(V)` time regardless of how many neighbors it actually has. This is the right representation when the graph is *dense* — when most pairs of vertices are connected — or when the algorithm needs direct matrix access. Floyd-Warshall, which we'll reach, is built around matrix operations and naturally uses this form.

A third representation, the **edge list**, stores the graph as a flat list of triples: source vertex, destination vertex, weight. It's less convenient for traversal but natural for algorithms that iterate over edges rather than vertices. Bellman-Ford and Kruskal's MST algorithm work this way.

| representation | memory | edge existence query | neighbor iteration | best used when |
| --- | --- | --- | --- | --- |
| three-way comparison of adjacency list, adjacency matrix, and edge list — | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## The Two Fundamental Traversals

Before you can do anything sophisticated with a graph, you need to be able to walk it — to visit every reachable vertex starting from some source. Two approaches, both taking `O(V + E)` time, differing only in the order they visit things.

**Breadth-first search** (BFS) explores vertices in order of their distance from the source. It visits the source first, then all vertices one edge away, then all vertices two edges away, and so on. The implementation uses a queue: put the source in, then repeatedly pull from the front, add unvisited neighbors to the back. The queue enforces the level-by-level order.

Why does this matter? Because BFS, on an unweighted graph, is a shortest-path algorithm. The first time BFS reaches any vertex, the path it followed is one of the shortest paths to that vertex — shortest in the sense of fewest edges. If you want to know the minimum number of hops between two things (minimum friendship distance in a social network, fewest keystrokes to reach a state, minimum road segments between two intersections), BFS gives you the answer directly.

**Depth-first search** (DFS) explores as deep as possible before backtracking. Starting from the source, it follows one path all the way until it hits a dead end or a vertex already visited, then backs up and tries another direction. The implementation uses a stack — or equivalently, recursion, which uses the call stack implicitly.

The natural use for DFS is not path-finding but *structure detection*. When DFS finishes exploring from a vertex, it knows something about the structure around that vertex: whether there's a cycle (if it ever reaches a vertex already on the current path), what order to process tasks (topological sort, for directed acyclic graphs), which parts of the graph form strongly connected components. These are questions about the shape of the graph, not the cost of a path, and DFS is designed to answer them.

The choice between BFS and DFS is rarely about performance — both are linear in the size of the graph. It's about what you want to know. Minimum hops, BFS. Structure and ordering, DFS.

![The same small graph (8–10 vertices) traversed twice](images/05-graphs-and-graph-search-algorithms-fig-02.png)
*Figure 5.2 — The same small graph (8–10 vertices) traversed twice*

---

## Shortest Paths: Four Algorithms, Four Situations

The interesting algorithms on graphs are the ones that find shortest paths when edges have weights. There are four classical approaches, each correct under a different set of constraints. Using the wrong one is not just inefficient — on negative-weight edges, Dijkstra is not merely slow, it is *wrong*.

**Dijkstra's algorithm** solves single-source shortest paths when all edge weights are non-negative. You want the shortest path from one source vertex to all others (or to a specific target). The idea: maintain a priority queue of vertices keyed on their tentative distance from the source. Repeatedly extract the vertex with the smallest tentative distance, finalize that distance, and update ("relax") the distances of its neighbors if you've found a shorter route through it. With a binary heap, the total time is `O((V + E) log V)`.

The non-negativity requirement is not a technicality. Here is why it matters. When Dijkstra extracts a vertex from the priority queue, it declares that vertex's distance final — the claim is that no shorter path will be discovered later. This claim holds when all edges are non-negative: you can never find a shorter path to an already-finalized vertex because any extension through positive-weight edges can only increase the cost. With a negative edge, that claim breaks. You could extract a vertex, declare its distance final, and then discover a path through a negative edge that would have been cheaper. Dijkstra has no mechanism to revisit finalized vertices.

![Directed graph (4–5 vertices) where Dijkstra produces an](images/05-graphs-and-graph-search-algorithms-fig-03.png)
*Figure 5.3 — Directed graph (4–5 vertices) where Dijkstra produces an*

**Bellman-Ford** handles single-source shortest paths when edges can be negative. The approach is fundamentally different: instead of greedily extracting the closest vertex, simply iterate over every edge in the graph `|V| − 1` times, relaxing each edge each time. After `|V| − 1` iterations, if no negative cycle exists, all shortest distances are correct. One more pass that relaxes any edge further proves a negative cycle exists.

Why `|V| − 1` iterations? The shortest path between any two vertices in a graph with no negative cycles visits at most `|V| − 1` edges (any more would imply a repeated vertex, i.e., a cycle, which could be removed to get a shorter or equal-length path). So `|V| − 1` passes are sufficient to propagate distance information along any possible shortest path.

The cost of this generality is time: `O(VE)`, slower than Dijkstra by a factor of roughly `V / log V`. You reach for Bellman-Ford when the model demands it — negative-weight edges appear naturally in financial models (rebates, currency exchange rates where arbitrage opportunities create effective negative costs), certain network protocols, and optimization problems where costs can be negative.

The negative-cycle detection is not a side effect but a feature. If you run Bellman-Ford on a graph of currency exchange rates where edge weights are the negative logarithm of exchange ratios, a negative cycle in that graph is a cycle of trades that ends with more money than you started with — an arbitrage opportunity.

**Floyd-Warshall** solves a different problem: *all-pairs* shortest paths, meaning the shortest path between every pair of vertices simultaneously. The approach is dynamic programming. Initialize a distance matrix from the edge weights. Then, for each vertex `k` in turn, update every pair `(i, j)`: if the path from `i` to `j` through `k` is cheaper than the current best direct path, update it. Three nested loops, one over intermediate vertices `k`, two over all pairs `(i, j)`. Time: `O(V³)`. Space: `O(V²)` for the matrix.

Floyd-Warshall wins when you need all pairs and the graph is small enough. On dense graphs where `E ≈ V²`, running Dijkstra from every vertex costs `O(V² log V + V³ / log V)`, which is comparable to or worse than Floyd-Warshall's `O(V³)` — and Floyd-Warshall's inner loop is a tight matrix update with excellent cache behavior. For sparse all-pairs problems on larger graphs with possible negative edges, Johnson's algorithm reruns Dijkstra from each vertex after reweighting edges to eliminate negatives; time `O(VE log V)`.

**A\*** (A-star) is Dijkstra with a heuristic. Instead of keying the priority queue on the actual distance from source `g(v)`, it keys on `g(v) + h(v)`, where `h(v)` is an estimate of the remaining distance from `v` to the goal. When the heuristic is *admissible* — it never overestimates the true remaining distance — A\* is still optimal: it will find the shortest path. But it explores far fewer vertices, focusing the search in the direction of the goal rather than radiating outward uniformly.

The classic heuristic for road networks is great-circle distance: the straight-line distance between two geographic points, which is a lower bound on road travel time if you divide by the maximum speed allowed on any road. On real road networks, A\* with this heuristic explores roughly 1 to 10 percent of the vertices that Dijkstra explores for the same source-target pair. Same answer, dramatically less work.

The heuristic's quality determines A\*'s advantage. In a maze where straight-line distance is a poor predictor of actual path cost — because walls force long detours — the heuristic informs little and A\* degrades toward Dijkstra. The quality of the heuristic is a claim about the geometry of the problem: whether the shortest path tends to go roughly in the direction of the goal.

![Two side-by-side grid graphs showing explored vertices for](images/05-graphs-and-graph-search-algorithms-fig-04.png)
*Figure 5.4 — Two side-by-side grid graphs showing explored vertices for*

---

## Minimum Spanning Trees

A different kind of question: you want to connect all vertices in a weighted graph using the minimum total edge weight. Not shortest paths between pairs — a tree of edges that reaches every vertex as cheaply as possible. This is the minimum spanning tree (MST).

Why would you want this? Network design, for one: you want to lay the minimum total cable to connect all offices. Clustering algorithms sometimes use MSTs to find structure in data. Any problem where you want to wire up a set of things at minimum total cost.

Three classical algorithms, each with a different implementation trade-off.

**Kruskal's algorithm** thinks about edges. Sort all edges by weight. Iterate through them cheapest first. Add each edge to the MST unless it would create a cycle — which you detect using a data structure called Union-Find (Chapter 3) that tracks which vertices are already connected. Time: `O(E log E)`, dominated by the sort. Natural fit for sparse graphs where `E` is small relative to `V²`.

**Prim's algorithm** thinks about vertices. Start from any vertex. At each step, add the cheapest edge that connects the current partial tree to a vertex not yet in it. Use a priority queue to find that cheapest edge efficiently. Time: `O(E log V)` with a binary heap. Natural fit for dense graphs.

Both algorithms are greedy: they always take the locally cheapest option. The proof that both are correct rests on the *cut property*: for any partition of vertices into two sets, the minimum-weight edge crossing the partition belongs to some MST. Kruskal and Prim both, in different ways, repeatedly apply this property.

**Borůvka's algorithm**, the oldest of the three (1926), has each component independently find its cheapest outgoing edge and merge. Because components work independently, it's naturally parallel. Useful when the graph is too large to process centrally or when parallel hardware is available.

![Step-by-step construction of an MST on the same](images/05-graphs-and-graph-search-algorithms-fig-05.png)
*Figure 5.5 — Step-by-step construction of an MST on the same*

---

## When "Just Use Dijkstra" Goes Wrong

The most common mistake in graph algorithms is not misimplementing an algorithm but misapplying one. Dijkstra is the shortest-path algorithm people reach for by reflex, and it's wrong in several situations.

**Negative edges.** As argued above, Dijkstra finalizes distances and cannot revise them. Use Bellman-Ford.

**Negative cycles.** No well-defined shortest path exists. Bellman-Ford will report the negative cycle.

**A specific target, not all reachable vertices.** Dijkstra computes shortest paths to *every* reachable vertex. If you only need the path to one target, A\* with a good heuristic explores a fraction of the graph.

**All-pairs queries on a dense small graph.** Running Dijkstra `V` times costs `O(V² log V + VE)`. Floyd-Warshall costs `O(V³)` but with a simpler, cache-friendly inner loop — on dense graphs with small `V`, Floyd-Warshall often wins in practice.

**Unweighted graphs.** Dijkstra's priority queue overhead is unjustified when all edges have weight 1. BFS is linear and simpler.

**Huge graphs where the search itself dominates.** Bidirectional search runs two simultaneous searches from source and target, meeting in the middle; the explored region shrinks by roughly the square root. Applicable as an extension to Dijkstra or A\*.

The corrective habit is simple: state the problem before picking the algorithm. Single-source or all-pairs? Weighted or unweighted? Negative weights possible? Is there a specific target and a useful heuristic? Each answer rules out some algorithms and points to others. The decision is in the constraints of the model, not in reflex.

---

## How the Model Determines the Algorithm

Here is a concrete instance of why this matters. A logistics service computes route times between distribution centers on a road network with tens of millions of intersections and edges weighted by travel time in seconds. Three different versions of the same problem demand three different algorithms.

In the basic version, you want the fastest route between two specific distribution centers. Dijkstra with a binary heap, stopped when the target vertex is finalized. Correct and fast.

In the improved version, you still want the same route, but you want it computed quickly enough to respond to a live user. A\* with a great-circle heuristic explores a small fraction of the graph — the heuristic focuses the search toward the destination. Same answer, often ten to a hundred times faster.

In the pricing-model version, the company has implemented variable toll pricing where certain tolls offer rebates at off-peak hours. The rebate can exceed the toll, making the effective edge weight negative. Dijkstra and A\* are now incorrect: they can finalize a vertex's distance before discovering the negative-effective-weight toll road that would have yielded a cheaper route. Bellman-Ford is correct at the cost of being slower.

The algorithm changed because the model changed. A change in business requirements — adding a rebate pricing system — changed the graph's properties, which changed the correct algorithm. Reading "Dijkstra is the shortest-path algorithm" without remembering the non-negative-weights precondition is the failure mode. The preconditions are not fine print. They are the definition of what problem the algorithm solves.

| single-source or all-pairs | negative weights allowed | specific target | unweighted | time complexity |
| --- | --- | --- | --- | --- |
| Dijkstra, Bellman-Ford, Floyd-Warshall, A*, BFS | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| columns: single-source or all-pairs, negative weights allowed, specific target, unweighted, time complexity | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| fill each cell with yes | no | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| student should be able to run down the columns for their problem and arrive at the right row | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## What You Can Do Now

You can identify when a problem is a graph problem — when the question involves reachability, shortest paths, connectivity, cycles, or ordering, the structure to reach for is a graph. You can choose between adjacency-list and adjacency-matrix representations based on graph density. You can pick between BFS (minimum hops, unweighted) and DFS (cycles, components, topological ordering) based on what you want to know. You can distinguish the four classical shortest-path algorithms by their preconditions: Dijkstra for non-negative weights, Bellman-Ford for negative weights, Floyd-Warshall for all-pairs on dense small graphs, A\* when you have a target and a heuristic. You can choose between Kruskal and Prim for MST based on graph density.

More fundamentally: you can see a new problem, draw the graph, and let the structure tell you which algorithm applies. That's the skill. The algorithms themselves are tools; the skill is recognizing which tool fits.

The next two chapters build directly on this foundation. Chapter 6 shows how the greedy structure behind Kruskal's and Prim's MST algorithms generalizes to an entire class of optimization problems — matroids — and why greedy works when it works. Chapter 8 shows how Bellman-Ford and Floyd-Warshall are instances of dynamic programming, and how the same principle extends to shortest paths in graphs you haven't drawn yet.

---

## Exercises

### Warm-Up

**1.** For each of the following problems, state whether it is a graph problem. If it is, name the vertices, the edges, and the question being asked. If it isn't, explain why the graph model doesn't add anything.

- Finding all prerequisites you must complete before taking a given course
- Sorting a list of numbers in ascending order
- Detecting whether two users in a social network are connected by fewer than six degrees of separation
- Finding the longest substring in a text without repeating characters

*Tests: recognizing when the graph abstraction applies; translating a real-world problem into vertices, edges, and a graph question.*

**2.** You are given a small graph stored as an adjacency matrix and asked to convert it to an adjacency list. The graph has 5 vertices and 4 edges. After converting:

(a) Which representation uses less memory? By how much (in terms of V and E)?

(b) If the graph instead had 5 vertices and all possible edges (a complete graph), which representation would be more efficient, and why?

(c) A colleague proposes always using the adjacency matrix "because edge lookups are O(1)." Under what conditions is this advice correct? When does it lead you astray?

*Tests: adjacency list vs. matrix trade-offs; reasoning about sparse vs. dense graphs.*

**3.** Trace BFS and DFS on the following graph starting from vertex A. (Draw or describe the graph: A connects to B and C; B connects to D and E; C connects to F; D, E, F have no outgoing edges.)

(a) Give the vertex visit order for BFS.

(b) Give the vertex visit order for DFS (assume neighbors are processed in alphabetical order).

(c) Which traversal would you use to find the minimum number of edges between A and F? Why?

*Tests: tracing BFS and DFS; selecting the right traversal for a path-length question.*

---

### Application

**4.** Apply Dijkstra's algorithm to the following weighted directed graph. Source vertex: A.

Edges: A→B (weight 4), A→C (weight 2), B→D (weight 3), B→C (weight 1), C→B (weight 1), C→D (weight 5), D→E (weight 1).

Show the state of the priority queue after each extraction step. Give the final shortest distances from A to every other vertex.

*Tests: executing Dijkstra correctly; tracking priority queue state; reading off final distances.*

**5.** The graph from Exercise 4 has an edge added: E→A (weight −8). Dijkstra's algorithm, run from A, produces an incorrect answer.

(a) Identify which vertex's distance is incorrectly finalized and explain why.

(b) Run Bellman-Ford on the modified graph. Does it detect a negative cycle? Show the distances after each of the required iterations.

(c) What does the negative cycle mean for the question "what is the shortest path from A to E in this graph?"

*Tests: identifying Dijkstra's failure mode on negative edges; applying Bellman-Ford; reasoning about negative cycles and their consequences.*

**6.** You are designing a system that must answer "what is the shortest travel time between any two cities?" for a network of 200 cities with direct flight connections (all travel times positive). Two approaches are proposed: (a) run Dijkstra from each of the 200 cities, or (b) run Floyd-Warshall once.

For each approach, give the time complexity in terms of V (cities) and E (edges). Identify the conditions under which Floyd-Warshall wins despite having `O(V³)` complexity. Your answer should reference the graph's density and the constant-factor behavior of Floyd-Warshall's inner loop.

*Tests: comparing all-pairs approaches; applying Floyd-Warshall's trade-off against repeated Dijkstra.*

---

### Synthesis

**7.** A mapping application wants to find the fastest walking route between two points in a city. The street network has 500,000 intersections (vertices) and 1.2 million street segments (edges), all with positive weights representing walking time in seconds.

(a) Which shortest-path algorithm should the application use for each of these requirements? Justify each choice with reference to the algorithm's preconditions and the problem's constraints.

- Fastest path between two specific intersections, no latency budget
- Fastest path between two specific intersections, must respond in under 50 ms
- Fastest path between all pairs of intersections (for precomputation)

(b) For the 50 ms requirement, describe what property the heuristic must satisfy for correctness, and propose a specific heuristic. Explain why it satisfies that property.

*Tests: selecting algorithms from their preconditions across varied requirements; constructing and validating an admissible heuristic.*

**8.** A build system represents software packages as vertices and dependencies as directed edges — package A points to package B if A depends on B. The system needs to determine a valid build order (build dependencies before the packages that depend on them).

(a) What graph property must hold for a valid build order to exist? Which traversal detects the violation?

(b) Which traversal produces the build order, and what is the output called? Describe the algorithm in terms of DFS.

(c) Suppose a new requirement is added: among all valid build orders, choose the one that minimizes total build time (each package has an estimated build duration). Is this still a graph traversal problem, or has it become something else? Name the new problem class.

*Tests: applying DFS to topological sort; recognizing when a graph problem becomes a different kind of optimization problem.*

---

### Challenge

**9.** The negative-cycle detection property of Bellman-Ford is used in currency arbitrage detection. Construct the graph model explicitly: given a set of currencies and pairwise exchange rates, define the vertices, the edge weights (as a formula involving the exchange rates), and the graph property whose presence signals an arbitrage opportunity.

Your answer should explain why the negative logarithm transformation converts the multiplicative arbitrage condition into an additive graph property, and why that matters for the algorithm. If you have access to real exchange rate data, verify that no negative cycle exists in the current rate table (or find one if it does).

*Tests: constructing a graph model from a financial problem; applying logarithmic transformation to convert multiplicative to additive; connecting Bellman-Ford's negative-cycle detection to a real-world use case.*

**10.** A\* is guaranteed to find the optimal path when the heuristic is admissible. But "optimal" depends on what you're optimizing. Suppose you want the path that minimizes the *maximum single-edge weight* along the route (the "bottleneck shortest path" — useful for finding the route with the least-congested segment).

(a) Does Dijkstra solve this problem as stated? If not, what modification would be needed?

(b) Can A\* be adapted for this objective? What would admissibility mean for a heuristic in this context?

(c) Is there a simpler algorithm — perhaps involving MSTs — that solves the bottleneck shortest path problem more directly?

*Tests: recognizing when standard algorithms solve a problem and when they need modification; reasoning about what "optimality" means for non-standard objectives; connecting shortest-path and MST problems.*

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
