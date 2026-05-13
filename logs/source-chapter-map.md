# Source-Chapter Map — Algorithms by Bear, Vol. 1

Built: 2026-04-30. One-time mapping pass. Reverse-index by target chapter.

Source paths are relative to `books/algorithms-by-bear-vol1/source/`. Line numbers are approximate, point to section starts.

---

## Chapter 01 — Introduction to Algorithms

- `Introduction_to_Algorithms/README.md` — **EMPTY**. No source content.
- Material to draw from: `book.md` (book identity, audience, architecture, scope), `outline.md` (chapter map, reader walkthroughs).
- Status: Claude must compose primarily from book.md and outline.md. Mark technical claims `[verify]` where they go beyond those two files.

## Chapter 02 — Algorithm Analysis

Primary: `Algorithm_Analysis/Algorithm_Analysis.md`

Keep here:
- `## Introduction to Algorithm Analysis` (lines 3–321) — definitions, complexity measures, run-time evaluation
- `## Understanding Complexity Notations` (lines 322–565) — Big O, Big Omega, Big Theta, comparison
- `## Analytical Tools` (lines 566–755) — asymptotic analysis, recurrence relations, amortized analysis (note: amortized cross-references Ch 3)
- `## Practical Analysis of Algorithms` → `### Binary Search` (lines 763–838) — keep as a brief Big O example
- `## Tools and Techniques` (lines 2014–2101) — profiling and benchmarking, cross-reference companion page

Cache-effects section (Section 6 of Ch 2): **NOT IN SOURCE.** Must be written fresh; the chapter's differentiation from CLRS lives here. [verify] all specific numbers (cache line sizes, latency ratios) against current hardware references.

Crossover benchmark (insertion vs merge): Source mentions sorting comparison generally but does not provide the n≈32 crossover benchmark with concrete numbers. Number is canonical (used by CPython's Timsort, Java's Dual-Pivot Quicksort). [verify] specific cutoff per implementation.

Cut entirely:
- `## End of Chapter Exercises` (lines 2256–2487)
- `## Chapter Summary` (lines 2488–end)
- `## Conclusion → Future Trends` (lines 2125–2138) — speculative
- `## Further Reading and Resources` (lines 2139–2255) — companion page

Route OUT of Ch 2 (Algorithm_Analysis content that belongs in other chapters):

- → **Ch 4 (Sorting):** `### Sorting Algorithms` (lines 839–1139). Includes insertion, bubble, selection, merge, quick, heap. Move the algorithm coverage; keep ONLY the asymptotic comparison table in Ch 2.
- → **Ch 5 (Graphs):** `### Graph Algorithms` (lines 1140–1255). BFS, DFS, Dijkstra, Bellman-Ford, Floyd-Warshall, Kruskal, Prim algorithm complexity discussion.
- → **Ch 6 (Greedy):** `### Greedy Algorithms` (lines 1508–1674). Coin change, fractional knapsack, activity selection.
- → **Ch 7 (D&C):** `### Divide and Conquer Algorithms` (lines 1261–1430). Merge sort, Karatsuba, Strassen analysis.
- → **Ch 8 (DP):** `### Dynamic Programming` (lines 1431–1507). Memoization, tabulation, optimal substructure.
- → **Ch 11 (Approximation):** `### Approximation Algorithms` (lines 1905–2013). TSP and Vertex Cover treatments specifically.
- → **Ch 12 (Randomized):** `### Randomized Algorithms` (lines 1677–1818). Monte Carlo, Las Vegas, randomized quicksort analysis.
- → **Cut to magazine:** `### Parallel Algorithms` (lines 1819–1904). Out of scope for classical canon.

## Chapter 03 — Data Structures

Primary: `Data_Structures/Data_Structures.md`

Keep:
- `## Introduction to Data Structures` (3–98) — classification, applications
- `## Amortized Analysis` (99–220) — concept, methods, dynamic table case study (cross-ref to Ch 2)
- `## Heaps` (221–487) — binary, binomial, Fibonacci
- `## Union-Find` (488–687)
- `## Balanced Search Trees` (688–1074) — BSTs, red-black trees
- `## Hash Tables` (1075–1184) — operations, collisions, universal hashing, implementation
- `## Bloom Filters` (1185–1292)
- `## Analysis with Mathematical Foundations → Space-Time Trade-offs` (1356–1382)
- `## Conclusion → Choosing the Right Data Structure` (1422–1489) — feed into decision rules

Add: hash-table content from `Algorithm_Analysis.md` "Hashing in Randomized Algorithms" subsection if present (basic hashing routes here; randomized analysis stays in Ch 12).

Cut: `## Exercises and Problems`, `## Further Reading and Resources`, all "Future Trends" speculation.

## Chapter 04 — Sorting and Caching

Primary: `Sorting_and_Caching/Sorting_Caching.md` + sorting content routed from `Algorithm_Analysis.md`.

Keep from `Sorting_Caching.md`:
- `## Introduction to Caching` (3–82) — definitions, role
- `## Theoretical Foundations of Caching` (83–168) — locality, eviction policies (overview)
- `## Types of Caches` (169–312) — hardware vs software
- `## Cache Eviction Strategies` (313–445) — LRU, FIFO, Random, ARC
- `## Caching in Distributed Systems → CDNs` (500–599) — informs the worked example
- `## Caching Algorithms and Data Structures` (600–720) — brief, cross-ref to Ch 3 for bloom/count-min
- `## Performance and Optimization` (721–784)
- `## Case Studies → High-Performance Computing` (899–939)

From `Algorithm_Analysis.md` (routed in):
- `### Sorting Algorithms` section (839–1139) — insertion, bubble, selection, merge, quick, heap, comparison table

Add (not in source): **Timsort** specifics — natural-runs detection, merging policy, reasons CPython adopted it (Python's `list.sort()` since 2.3, 2002 [verify]). **Introsort** specifics — quicksort + heapsort + insertion sort hybrid used by libstdc++ and MSVC STL. [verify all].

Cut: `## Emerging Trends → Machine Learning for Cache Management` (792–823) — out of scope for classical canon. `## Exercises and Problems`.

## Chapter 05 — Graphs and Graph Search

Primary: `Graphs_and_Graph_Search_Algorithms/Graph_Algorithms.md`

Keep:
- `## Introduction to Graph Theory` (3–550) — definitions, types, representations
- `## Graph Traversal Techniques` (551–840) — BFS, DFS, comparison
- `## Special Graph Algorithms` (841–1095) — topological sort, strong components
- `## Graph Search Strategies` (1553–1739) — advanced BFS/DFS, A* and heuristic search
- `## Graph Path Finding Algorithms` (1740–2082) — Dijkstra, Bellman-Ford, Floyd-Warshall, Johnson's

From `Algorithm_Analysis.md`:
- Graph algorithms complexity table from `### Graph Algorithms` (1140–1255)

Route OUT of Ch 5:
- → **Ch 9 (Network Flow):** `## Network Flow and Matching` (2083–2330). Max flow / min cost flow material.
- → **Ch 12 (Randomized):** `## Minimum Cut → Random Contraction` (1096–1406) and `## Randomized Algorithms in Graph Theory` (1407–1552). Karger's algorithm is the worked example for Ch 12.
- → **Ch 6 (Greedy):** MST treatment — keep in Ch 5 the brief introduction; greedy proof patterns and full MST algorithms (Kruskal, Prim, Borůvka) live in Ch 6 Greedy_Algorithms source.

Cut: `## Conclusion → Future Directions`, `## End of Chapter Exercises`.

## Chapter 06 — Greedy Algorithms

Primary: `Greedy_Algorithms/Greedy_Algorithms.md`

Keep:
- `## Introduction to Greedy Algorithms` (3–374) — definition, principles, applications, efficiency
- `## Interval Scheduling` (375–497) — earliest-finish-time + proof of optimality (worked example anchor)
- `## Interval Partitioning` (498–627) — depth lower bound argument
- `## Shortest Paths and MSTs` (628–1193) — keep the MST greedy material here (Kruskal, Prim, Borůvka). Cross-ref Dijkstra to Ch 5.
- `## Huffman Coding` (1194–1387)
- `## Conclusion → Summary` (1711–1856)

From `Algorithm_Analysis.md`:
- Routed greedy content from `### Greedy Algorithms` (1508–1674)

Note: Vertex cover and set cover proofs/tightness analysis go to Ch 11 Approximation. Ch 6 introduces these algorithms briefly only.

Cut: `## Challenges and Future Directions → Emerging Research`, `## Exercises and Problems`, `## Further Reading and Resources`.

## Chapter 07 — Divide and Conquer

Primary: `Divide_and_Conquer_Strategies/Divide_and_Conquer_Algorithms.md`

Keep:
- `## Introduction to Divide and Conquer` (3–298)
- `## Sorting and Selection → Merge Sort, Quickselect` (299–736) — keep as brief reference; merge sort full coverage in Ch 4
- `## Integer and Polynomial Multiplication → Karatsuba` (737–1204)
- `## Matrix Multiplication → Strassen` (1205–1495) — referenced, not implemented
- `## Analyzing and Solving Problems → Counting Inversions, Closest Pair` (1496–1744) — **Closest Pair is the worked example**
- `## Quicksort` (1745–1896) — brief, point to Ch 4
- `## Comparative Analysis` (1897–1986)

From `Algorithm_Analysis.md`:
- Routed D&C content (1261–1430). Master theorem stays in Ch 2 (cross-ref).

Cut: `## Advanced Topics → Parallelization`, `## Practical Applications → Modern Software Development` (vague), `## Challenges and Future Directions`, `## Conclusion`.

## Chapter 08 — Dynamic Programming

Primary: `Dynamic_Programming/Dynamic_Programming.md` + `Bellman_Ford.md`

Keep:
- `## Introduction to DP` (3–341) — principles, history, basic techniques, memoization vs tabulation
- `## The Knapsack Problem` (342–418)
- `## Sequence Alignment` (419–576) — **edit distance / sequence alignment is the worked example**
- `## Optimal Binary Search Trees` (577–674)
- `## Shortest Paths in Graphs → Bellman-Ford, Floyd-Warshall` (675–750) — DP framing; cross-ref Ch 5
- `## DP for NP-Complete → Faster Exact` (751–841)
- `## Advanced DP → State Space Reduction` (948–1060)
- `## Comparative Analysis → DP vs Greedy, DP vs D&C` (1061–1124)
- `## Practical Implementations → Common Pitfalls` (1145–1170)

From `Algorithm_Analysis.md`:
- Routed DP content (1431–1507).

Bellman_Ford.md: Use as supplementary detail for the DP-framing of Bellman-Ford in Ch 5 cross-reference.

Cut: `## DP for NP-Complete → Approximation Algorithms` (842–947) — route to Ch 11. Quiz_Questions.md — out of scope for reference. `## Applications → CV/NLP` — speculative.

## Chapter 09 — Network Flow

Primary: `Network_Flow/Network_Flow.md`

Keep:
- `## Introduction to Network Flow` (3–159)
- `## Fundamentals` (160–279)
- `## Maximum Flow Theory` (280–416) — max-flow min-cut, augmentation
- `## Algorithms for Maximum Flow → Ford-Fulkerson, Push-Relabel` (417–1076)
- `## Applications of Maximum Flow → Assignment, Connectivity` (850–1093)
- `## Advanced → Min-Cost Flow, Multi-commodity` (1094–1341) — brief, distinguish in decision rules

Add (not in source): **Image segmentation via min-cut** worked example (foreground/background, energy-based edge weights). [verify] standard formulation references — Boykov & Kolmogorov 2004, Greig/Porteous/Seheult 1989.

Cut: `## Software and Tools` (cross-ref companion page), `## Case Studies` use selectively, `## Challenges → ML Integration`, `## Exercises`, `Quiz_Questions_Network_Flow.md`.

## Chapter 10 — NP-Completeness and Intractability

Primary: `Intractability/README.md` — **VERY THIN.** Only lesson outlines, no content. Most coverage must be composed from canonical knowledge with [verify] tags.

Material from outline:
- NP-Completeness definition
- Reductions and NP-Hard
- NP-Completeness proofs
- PSPACE
- Extending tractability

Worked example (course scheduling → graph coloring): Standard textbook reduction. [verify] specific construction details.

Cross-references (heavy): Ch 11 (approximation paths), Ch 13 (LP for NP-hard), Ch 12 (randomized).

Note: This chapter is highest-risk for fabrication. Mark every reduction, every canonical NP-complete problem, every complexity-class claim as `[verify]` or anchor to widely-canonical references (Cook 1971, Karp 1972, Garey & Johnson 1979).

## Chapter 11 — Approximation Algorithms

Primary: `Approximation_Algorithms/Approximation_Algorithms.md`

Keep:
- `## Introduction to Approximation` (3–139)
- `## Fundamentals → Ratio, PTAS, FPTAS` (140–309)
- `## Design Techniques → Greedy, DP, LP Rounding` (310–425)
- `## List Scheduling Algorithms` (426–536)
- `## Local Search` (537–783) — brief, point to broader treatment
- `## Probabilistic and Metaheuristic → Simulated Annealing, Hopfield` (784–922) — keep simulated annealing only; Hopfield routes to magazine
- `## Case Studies → Vertex Cover, Set Cover, Network Design` (923–1078) — **Set Cover is the worked example**
- `## Challenges → Limits of Approximability` (1079–1134)

From `Algorithm_Analysis.md`:
- Routed approximation content (1905–2013) — TSP and Vertex Cover

Cut: `## Future of Approximation Algorithms` (speculative), `## Practical Implementations → Real-World` use selectively, `## Exercises`.

## Chapter 12 — Randomized Algorithms

Primary: `Randomized_Algorithms/Randomized_Algorithms.md` + Karger's from `Graph_Algorithms.md`

Keep:
- `## Introduction` (3–72)
- `## Theoretical Foundations` (73–189) — probability basics, random variables, concentration, randomized quicksort
- `## Classification → Monte Carlo, Las Vegas` (190–352)
- `## Design and Analysis → Sampling, Randomized Data Structures, Random Walks` (353–697)
- `## Applications → Graph Algorithms (Karger's)` (807–997)
- `## Cryptography → Probabilistic Encryption, Random Oracle` (998–1118) — **brief, this is the cryptographic-vs-algorithmic randomness section**
- `## Computational Geometry → Closest Pair` (1119–1187)
- `## Challenges → Derandomization, Randomness Quality` (1286–1439)

From `Algorithm_Analysis.md`:
- Routed randomized content (1677–1818)

From `Graph_Algorithms.md`:
- Karger's min-cut treatment (1096–1406) — worked example anchor

Cut: `## Future Directions → Quantum Computing, Algorithmic Fairness` — magazine. `## Exercises`. `Quiz_Questions_Randomized_Algorithms.md`.

## Chapter 13 — Linear Programming

Primary: `Linear_Programming/Linear_Programming.md`

Keep:
- `## Introduction to LP` (3–80)
- `## Theoretical Foundations` (81–229)
- `## The Simplex Algorithm` (230–358)
- `## Linear Programming Duality` (359–465)
- `## The Ellipsoid Algorithm` (466–628)
- `## Interior Point Methods` (719–822)
- `## Cutting Plane Methods` (823–907)
- `## Computational Aspects → Numerical Stability, Scaling` (908–1127)
- `## Advanced Topics → Sensitivity Analysis` (1128–1228) — feeds the worked example
- `## Case Studies → Resource Allocation, Production Planning` (1272–1409) — **production planning at refinery/mill is the worked example**
- `## Case Studies → Financial Portfolio` (1410–1479) — brief reference

Cut: `## Stochastic LP` (1182–1228) — flag as Vol. 2. `## Multi-objective LP` — out of scope. `## Conclusion → Future`.

---

## Cross-routing decisions (one-time)

Algorithm_Analysis.md is the most cross-routed source. The chapter spec for Ch 2 is explicit: keep only foundations (Big O, recurrences, amortized) plus binary search as example. Everything else routes out per the table above.

## Out-of-scope flags (cut entirely)

- All "End of Chapter Exercises" sections in every source folder
- All "Chapter Summary" sections (these are recap, not reference content)
- All "Future Trends" / "Future Directions" speculative sections
- All "Further Reading and Resources" sections (companion page handles)
- All "Quiz Questions" files
- `Algorithm_Analysis.md` `### Parallel Algorithms` — out of scope
- All ML-for-X subsections — magazine

## Vol. 2 flags

- Stochastic Linear Programming (Linear_Programming.md 1182–1228)
- Bayesian methods, optimal stopping, game theory, scheduling, social networks — none in current source folders, but if encountered: flag.

## Magazine flags

- ML-driven cache management
- Quantum computing extensions
- Algorithmic fairness and bias
- Hopfield networks (neural-net adjacency)
- Any LLM / RL / generative-model content
