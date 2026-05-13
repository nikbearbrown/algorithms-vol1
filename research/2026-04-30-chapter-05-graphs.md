# Research notes — Chapter 05: Graphs and Graph Search

## Source folders used
- Primary: `Graphs_and_Graph_Search_Algorithms/Graph_Algorithms.md`
- Secondary: `Algorithm_Analysis/Algorithm_Analysis.md` graph algorithms section (lines 1140–1255), routed in

## Source sections kept
From `Graph_Algorithms.md`:
- `## Introduction to Graph Theory` (3–550) — definitions, types, representations
- `## Graph Traversal Techniques` (551–840) — BFS, DFS, comparison
- `## Special Graph Algorithms` (841–1095) — topological sort, strong components (briefly mentioned)
- `## Graph Search Strategies` (1553–1739) — A*, heuristic search
- `## Graph Path Finding Algorithms` (1740–2082) — Dijkstra, Bellman-Ford, Floyd-Warshall, Johnson's

## Source sections routed out
Per source-chapter-map.md:
- `## Network Flow and Matching` (2083–2330) → Chapter 9
- `## Minimum Cut → Random Contraction` (1096–1406) → Chapter 12 (Karger's worked example)
- `## Randomized Algorithms in Graph Theory` (1407–1552) → Chapter 12
- MST treatment: brief here; full treatment in Chapter 6 Greedy_Algorithms source

## Source sections cut entirely
- `## Conclusion → Future Directions in Graph Theory and Algorithm Research`
- `## End of Chapter Exercises`
- `## Further Reading and Resources`

## Original Claude content (NOT in source)

### Bidirectional search mention
Source does not cover bidirectional search; added briefly in failure modes. [verify] sqrt-reduction claim.

### A* admissibility/consistency conditions
Source describes A* generally but does not explicitly state the admissibility and consistency conditions for optimality. Added as a precondition statement.

### MST algorithm selection by graph density
Source covers Kruskal, Prim, Borůvka but does not explicitly map them to graph-density choice. Mapping added in decision rules.

### Worked example (§7) — road network
Per outline.md, this is the named worked example. The 10M vertices / 25M edges / specific heap operation count are illustrative numbers and marked [verify].

### Toll-pricing creates negative weights
Per outline.md, this is one of the three Bellman-Ford reasons in the worked example. Source describes Bellman-Ford for negative weights; the toll-pricing framing is Claude's. [verify] practical relevance — currency arbitrage is the more common cited use.

## Factual claims preserved from source
- BFS/DFS algorithms and complexities — from source
- Dijkstra's algorithm + 1956 attribution — from source
- Bellman-Ford + 1950s attribution — from source
- Floyd-Warshall attribution (Roy, Floyd, Warshall, early 1960s) — from source
- Johnson's algorithm (1977) — from source
- Adjacency list / matrix tradeoffs — from source

## [verify] count
8 inline `[verify]` markers:
1. Dijkstra 1956
2. Bellman-Ford 1950s (Bellman + Ford Jr.)
3. Roy 1959, Floyd 1962, Warshall 1962 (Floyd-Warshall attribution dates)
4. Johnson 1977
5. Borůvka 1926
6. Worked example road network 10M vertices / 25M edges
7. Worked example A* explores 1–10% vertices
8. Bidirectional search sqrt-reduction
9. Babai 2015 graph isomorphism breakthrough

(Count: 9.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: routing in road network with Dijkstra/A*/Bellman-Ford (named in outline.md) ✓
- Failure modes engages "Just use Dijkstra" misconception ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- Risk of becoming a catalog of algorithms with no through-line. The recognition-pattern section establishes the diagnostic move (problem → graph) as the chapter's intellectual spine.
- Resisted "let me explain why graph theory matters" textbook framing.
- The shortest-path quartet (Dijkstra, Bellman-Ford, Floyd-Warshall, A*) is treated as a coordinated decision problem, not four separate algorithms with biographies. This is the chapter's reference move.

## Word count
~3,030 words (target: ~3,000)

## Open issues for editor
- All historical attribution dates need verification
- Worked example numbers (10M / 25M / 1–10% / 100×) are illustrative; consider replacing with citations to OSM-based benchmarks
- Borůvka 1926 is widely cited but the original publication was in Czech; verify English-language attribution
- Toll-pricing example: consider whether currency arbitrage is more familiar to the reader
