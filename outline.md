# Outline — Algorithms by Bear, Vol. 1

13 chapters in 4 topic clusters. Each chapter's status is one of:
- `to write` — not yet drafted
- `drafted` — first draft complete
- `revised` — after editorial pass

When all chapters are `revised`, the book is ready for KDP submission.

## Cluster 1 — Foundations

### Chapter 01 — Introduction to Algorithms
- **Status:** drafted
- **Length target:** ~1,500 words
- **Source folders:** `Introduction_to_Algorithms/`
- **Primary capability:** Reader can use this book — knows what reference type it is, how chapters are structured, when to use it vs CLRS / Sedgewick / Skiena, how to navigate.
- **Anti-capability:** No algorithms learned yet. Orientation only.
- **Misconception engaged:** None directly.
- **Worked example:** Three "Reader X has problem Y" walkthroughs — Reader A picking a sort for a streaming workload, Reader B looking up Bayesian inference (and being directed to Vol. 2), Reader C cramming for an exam.
- **Companion-page handoffs:** None — this chapter is purely book-resident.
- **Notes:** Voice setting matters here. Direct, plain, slightly dry. Do NOT begin with "Welcome." Do NOT begin with "Algorithms are..." Begin with what the reader is here to do. Includes the explicit ML-scope statement and the recommendation to read Chapters 2 and 3 first for sequential readers.

### Chapter 02 — Algorithm Analysis
- **Status:** drafted
- **Length target:** ~3,200 words
- **Source folders:** `Algorithm_Analysis/` (primary). Note: this source contains substantial material that belongs in OTHER chapters (Sorting → Ch 4, Graphs → Ch 5, Greedy → Ch 6, D&C → Ch 7, DP → Ch 8, Approximation → Ch 11, Randomized → Ch 12). Route those out; do NOT include in this chapter.
- **Primary capability:** Determine asymptotic complexity using Big O; compare two algorithms' complexities.
- **Secondary capabilities:** Apply master theorem; distinguish worst-case / average / amortized; diagnose when asymptotic analysis misleads.
- **Anti-capability:** Cannot derive tight lower bounds for arbitrary problems.
- **Misconception engaged:** "O(n log n) is always good enough" — engage in the cache-effects section.
- **Worked example:** Insertion sort vs merge sort with a real benchmark crossover. Show the point (around n ≈ 32) where insertion sort becomes faster, and explain why standard libraries (Timsort, introsort) use insertion sort for small subarrays. Concrete benchmark numbers.
- **Section 6 (cache effects) is critical and is missing from the source — must be written fresh.** The chapter's differentiation from CLRS lives here.
- **Companion-page handoffs:** Extended benchmark code in Python with profiling. Master theorem reference card. Visualization gallery showing growth-rate curves at practitioner-relevant scales.
- **Cross-references:** Master theorem application → Chapter 7. Amortized → Chapter 3. Cache effects deeper → Chapter 4.
- **Notes:** Most cited chapter in the book. Take the space. Cut the source's "End of Chapter Exercises" section entirely — references do not have exercises.

### Chapter 03 — Data Structures
- **Status:** drafted
- **Length target:** ~3,000 words
- **Source folders:** `Data_Structures/`. Also extract hash-table content from `Algorithm_Analysis/`'s "Hashing in Randomized Algorithms" subsection (route the basic hashing here; randomized analysis stays in Ch 12).
- **Primary capability:** Choose appropriate data structure for a problem; justify based on operation costs.
- **Secondary capabilities:** Compare structures across operations; recognize when a structure is the bottleneck; identify when specialized structures are warranted.
- **Anti-capability:** Not implementation-ready code for every structure. Standard libraries provide implementations.
- **Misconception engaged:** "Use a hash map by default."
- **Worked example:** Designing storage for a real-time analytics system: count-distinct queries, top-k tracking, range queries on timestamped events.
- **Companion-page handoffs:** Implementation comparisons across Python / Java / C++ standard libraries. Specialized structures deep-dives (bloom filters, skip lists, count-min sketch).
- **Notes:** Resist the temptation to teach implementation. This is a reference for choice, not a textbook for building structures from scratch.

## Cluster 2 — Core Methods

### Chapter 04 — Sorting and Caching
- **Status:** drafted
- **Length target:** ~2,800 words
- **Source folders:** `Sorting_and_Caching/`. Also extract sorting algorithm coverage from `Algorithm_Analysis/`'s "Sorting Algorithms" section.
- **Primary capability:** Choose appropriate sorting algorithm given constraints (size, memory, distribution, stability).
- **Secondary capabilities:** Distinguish comparison vs non-comparison sorts; diagnose when cache behavior dominates; apply caching strategies (LRU, LFU).
- **Anti-capability:** Cannot optimize sorts at hardware level.
- **Misconception engaged:** "Quicksort is always fastest."
- **Worked example:** Two real cases. (1) Why Timsort beats classical merge sort on real Python data — natural-runs detection, stability, cache behavior. (2) Cache replacement in a real system (Linux page cache or a CDN), showing how the right replacement strategy affects hit rate by 10–30%.
- **Companion-page handoffs:** Benchmark suite, LRU/LFU/ARC implementations, real cache strategy tuning examples.
- **Cross-references:** Heapsort → Chapter 3 (heaps). Randomized quicksort → Chapter 12.
- **Notes:** The Timsort discussion is the chapter's signature. The caching half is unique to this book among algorithms references — do not undersell it.

### Chapter 05 — Graphs and Graph Search Algorithms
- **Status:** drafted
- **Length target:** ~3,000 words
- **Source folders:** `Graphs_and_Graph_Search_Algorithms/`. Also extract graph algorithm coverage from `Algorithm_Analysis/`'s "Graph Algorithms" section.
- **Primary capability:** Identify which classical graph algorithm applies (BFS, DFS, Dijkstra, Bellman-Ford, Floyd-Warshall, A*).
- **Secondary capabilities:** Represent real-world problems as graphs; choose representation; apply MST algorithms; diagnose failure modes.
- **Anti-capability:** No coverage of advanced graph topics (isomorphism, planarity, treewidth).
- **Misconception engaged:** "Just use Dijkstra."
- **Worked example:** Routing in a road network. Show the same problem solved with Dijkstra (basic), A* (with great-circle distance heuristic for speedup), and Bellman-Ford (when toll-pricing creates negative-effective-weight edges).
- **Companion-page handoffs:** OSM-based routing demos. Comparative performance benchmarks across graph sizes.
- **Cross-references:** Dijkstra requires heaps (Chapter 3). Bellman-Ford is DP-shaped (Chapter 8). MST connects to greedy (Chapter 6). Network flow extends graph models (Chapter 9).

### Chapter 06 — Greedy Algorithms
- **Status:** drafted
- **Length target:** ~2,600 words
- **Source folders:** `Greedy_Algorithms/`. Also extract greedy content from `Algorithm_Analysis/`'s "Greedy Algorithms" section.
- **Primary capability:** Determine whether a greedy approach is provably optimal, provably approximate, or a heuristic.
- **Secondary capabilities:** Recognize matroid and exchange-argument patterns; distinguish optimal-greedy from approximate-greedy; apply canonical greedy algorithms.
- **Anti-capability:** No procedure for inventing greedy algorithms for novel problems.
- **Misconception engaged:** "Greedy algorithms are sloppy; DP is the rigorous one."
- **Worked example:** Meeting room scheduling at a real organization. Show why earliest-finish-time is provably optimal (exchange argument), why earliest-start-time is wrong, and how the same problem with different constraints drifts toward heuristic territory.
- **Companion-page handoffs:** Extended catalog of greedy algorithms with proof patterns. Implementations in Python.
- **Cross-references:** MST greedy → Chapter 5. **For vertex cover and set cover proofs and tightness analysis, see Chapter 11 — do not carry the full treatment here, only introduce the algorithms.** DP contrast → Chapter 8.
- **Notes:** The matroid and exchange-argument patterns are the chapter's intellectual spine.

### Chapter 07 — Divide and Conquer Strategies
- **Status:** drafted
- **Length target:** ~2,600 words
- **Source folders:** `Divide_and_Conquer_Strategies/`. Also extract D&C content from `Algorithm_Analysis/`'s "Divide and Conquer Algorithms" section.
- **Primary capability:** Design a divide-and-conquer algorithm for a problem with self-similar structure; analyze using master theorem or recurrence-tree.
- **Secondary capabilities:** Apply canonical D&C algorithms; recognize when D&C beats alternatives; distinguish from DP via subproblem overlap.
- **Anti-capability:** Not implementation-ready FFT or Strassen — referenced with pointers.
- **Misconception engaged:** "D&C is just recursion."
- **Worked example:** Closest pair of points in 2D — a non-trivial geometric problem solved cleanly by D&C, going beyond merge sort. Show divide step, conquer step, and combine step.
- **Companion-page handoffs:** Strassen, FFT deep-dives. Closest-pair implementation. **Master theorem reference card lives with Chapter 2 — cross-reference, do not duplicate.** Recursion refresher.
- **Cross-references:** Master theorem → Chapter 2. Merge sort, quicksort → Chapter 4. DP contrast → Chapter 8.
- **Notes:** Section 2 (prerequisites) includes a one-line pointer to the recursion refresher on the companion page.

### Chapter 08 — Dynamic Programming
- **Status:** drafted
- **Length target:** ~3,200 words
- **Source folders:** `Dynamic_Programming/`. Also extract DP content from `Algorithm_Analysis/`'s "Dynamic Programming" section.
- **Primary capability:** Formulate a DP recurrence given a problem with optimal substructure and overlapping subproblems; choose top-down vs bottom-up; implement.
- **Secondary capabilities:** Recognize DP-amenable structure; apply canonical DP patterns; apply space optimization.
- **Anti-capability:** No procedure for solving every problem with DP.
- **Misconception engaged:** Continues "Greedy is sloppy; DP is rigorous" engagement from Chapter 6.
- **Worked example:** Edit distance in production. Real applications: spell checkers, DNA sequence alignment, fuzzy string matching in databases.
- **Companion-page handoffs:** Extended DP problem catalog. Visualizations of DP table evolution. Recursion refresher.
- **Cross-references:** D&C contrast → Chapter 7. Greedy contrast → Chapter 6. Bellman-Ford as DP → Chapter 5.
- **Notes:** This chapter delivers the book's most ambitious capability — Create-level recurrence design. Section 3 (designing a DP recurrence) is load-bearing.

## Cluster 3 — Optimization

### Chapter 09 — Network Flow
- **Status:** drafted
- **Length target:** ~2,650 words
- **Source folders:** `Network_Flow/`.
- **Primary capability:** Model a problem as a flow network and apply max-flow.
- **Secondary capabilities:** Recognize canonical reductions to flow; distinguish max-flow / min-cost flow / multi-commodity flow; apply max-flow min-cut.
- **Anti-capability:** Not implementing high-performance flow solvers from scratch.
- **Misconception engaged:** "Network flow is a niche topic."
- **Worked example:** Image segmentation via min-cut. Real practitioner problem (computer vision, medical imaging). Show how foreground/background separation reduces to max-flow min-cut on a pixel graph with energy-based edge weights.
- **Companion-page handoffs:** Solver comparisons (NetworkX, OR-Tools, LEMON). Image segmentation demo with code.
- **Cross-references:** Graph foundations → Chapter 5. LP duality contrast → Chapter 13. Approximation via LP relaxation → Chapter 11.
- **Notes:** The reductions section is the chapter's most practically useful part. Make the reduction patterns vivid.

### Chapter 10 — NP-Completeness and Intractability
- **Status:** drafted
- **Length target:** ~3,000 words
- **Source folders:** `Intractability/`.
- **Primary capability:** Determine whether a problem is likely NP-hard via reduction; correctly interpret what NP-hardness means for practical solving.
- **Secondary capabilities:** Apply standard reductions; distinguish theoretical from practical intractability; choose between exact-exponential / approximation / heuristic / special-case approaches.
- **Anti-capability:** Cannot prove P ≠ NP.
- **Misconception engaged:** "NP-complete means impossible."
- **Worked example:** Course scheduling reduced to graph coloring, with the modeling explicit. Then the practical follow-up: how universities actually schedule courses (heuristic-driven, partial-instance, with iteration).
- **Companion-page handoffs:** Reduction catalog. SAT solver demo. Phase-transition visualizations.
- **Cross-references:** Approximation paths → Chapter 11. LP for NP-hard → Chapter 13. Randomized → Chapter 12.
- **Notes:** The "what to do" section is what makes the chapter actionable rather than depressing. Don't bury it.

### Chapter 11 — Approximation Algorithms
- **Status:** drafted
- **Length target:** ~2,750 words
- **Source folders:** `Approximation_Algorithms/`. Also extract approximation content from `Algorithm_Analysis/`'s "Approximation Algorithms" section (TSP and Vertex Cover treatments).
- **Primary capability:** Identify or apply an approximation algorithm with a known approximation ratio; correctly interpret what the ratio guarantees.
- **Secondary capabilities:** Distinguish constant-factor / PTAS / FPTAS / inapproximable; apply canonical approximation algorithms; recognize practically meaningful vs theoretically tight ratios.
- **Anti-capability:** Approximation algorithms exist for many but not all NP-hard problems.
- **Misconception engaged:** "An approximation guarantee tells you how good the answer is on this instance."
- **Worked example:** Set cover greedy approximation applied to a real practitioner problem — sensor placement, server location, or test-suite reduction. Show why ln(n) approximation is meaningful in some instances and worse than alternatives in others.
- **Companion-page handoffs:** Comparative approximation algorithm benchmarks. PCP theorem reading list. Inapproximability catalog.
- **Cross-references:** NP-hardness setup → Chapter 10. Greedy approximation → Chapter 6 (this chapter owns the analysis; Chapter 6 references it). LP rounding → Chapter 13. Randomized rounding → Chapter 12.
- **Notes:** Section on when ratios mislead is the practitioner-discriminating content.

## Cluster 4 — Probabilistic

### Chapter 12 — Randomized Algorithms
- **Status:** drafted
- **Length target:** ~2,800 words
- **Source folders:** `Randomized_Algorithms/`. Also extract randomized content from `Algorithm_Analysis/`'s "Randomized Algorithms" section.
- **Primary capability:** Determine whether a randomized algorithm offers a useful tradeoff; choose between Monte Carlo and Las Vegas.
- **Secondary capabilities:** Apply linearity of expectation and basic concentration; recognize canonical randomized algorithms; distinguish algorithmic from cryptographic randomness.
- **Anti-capability:** No full coverage of derandomization.
- **Misconception engaged:** "Randomness in algorithms is just statistical noise."
- **Worked example:** Karger's min-cut algorithm — the surprise that random edge contraction works. Walk through an instance, the analysis showing the success probability per run, the amplification by repeated runs.
- **Companion-page handoffs:** Probability primer. Karger's algorithm visualization. LSH demo.
- **Cross-references:** Quicksort → Chapter 4. Hashing → Chapter 3. Randomized rounding → Chapter 11.
- **Notes:** Section 1 elevates the probability-primer pointer prominently. The cryptographic-vs-algorithmic randomness section is short but high-stakes.

## Capstone — Optimization (placed last)

### Chapter 13 — Linear Programming
- **Status:** drafted
- **Length target:** ~3,200 words
- **Source folders:** `Linear_Programming/`.
- **Primary capability:** Formulate a problem with linear constraints and a linear objective as an LP; choose between simplex and interior-point; interpret the dual.
- **Secondary capabilities:** Distinguish LP / IP / MIP / QP; apply LP duality; diagnose modeling failures.
- **Anti-capability:** Not implementing simplex or interior-point methods from scratch.
- **Misconception engaged:** "LP is for academics; production systems use heuristics."
- **Worked example:** Production planning at a steel mill or oil refinery. Multiple products, capacity constraints, demand forecasts. Show the LP formulation, the optimal solution, the dual interpretation as marginal value of capacity, the sensitivity analysis. Real practitioner problem with insight that goes beyond "here's the answer."
- **Companion-page handoffs:** Solver comparison (Gurobi, CPLEX, OR-Tools, GLPK, SciPy.linprog). Modeling tutorial in Python. Real-world LP case studies.
- **Cross-references:** LP relaxation → Chapter 11. LP duality vs max-flow min-cut → Chapter 9. Greedy on matroid LPs → Chapter 6.
- **Notes:** LP at Chapter 13 carries the "capstone optimization method" framing. The chapter explicitly closes threads opened in earlier chapters.