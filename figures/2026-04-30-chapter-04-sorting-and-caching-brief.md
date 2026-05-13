# Figure brief — Chapter 04: Sorting and Caching

## Recommended figures

### Figure 1 — Sort algorithm comparison table
**Path:** `../images/chapter-4-sort-comparison-table.jpg`
**Description:** Visual table. Rows: Bubble, Selection, Insertion, Merge, Quick, Heap, Timsort, Introsort, Counting, Radix. Columns: Best, Average, Worst, Space, Stable, In-place. Color cells by performance class (green/yellow/red).
**Use:** referenced near the catalog section as visual anchor.

### Figure 2 — Sort crossover benchmark
**Path:** `../images/chapter-4-sort-crossover-benchmark.jpg`
**Description:** Wall-clock plot. X-axis `n` from 1 to 10,000 (log). Y-axis time (log). Curves: insertion sort, quicksort, merge sort, Timsort. Crossover annotations at n≈32 (insertion exits) and at the inflection where Timsort wins on partially-sorted input.
**Use:** referenced in §3 (catalog) and §5 (worked example A).

### Figure 3 — Timsort run-detection animation (frame)
**Path:** `../images/chapter-4-timsort-runs.jpg`
**Description:** Sample input sequence with three natural ascending runs highlighted in different colors. Below: the merge tree showing how runs combine. Caption: "Real-world data often has natural runs Timsort exploits. Random data has none."
**Use:** referenced in §4 (Why your standard library hybridizes) and §5.

### Figure 4 — Cache hit-rate vs cache size
**Path:** `../images/chapter-4-hit-rate-vs-size.jpg`
**Description:** Plot. X-axis cache size (log). Y-axis hit rate. Curves for LRU, LFU, ARC on a representative workload trace. Knee in the curve labeled.
**Use:** referenced in §6 (Cache eviction) and §5 (worked example B).

### Figure 5 — Memory-hierarchy diagram
**Path:** `../images/chapter-4-memory-hierarchy.jpg`
**Description:** Diagram of registers / L1 / L2 / L3 / RAM / SSD / HDD with latencies, capacities, and "what fits where" annotations for typical sort and cache problems.
**Use:** could be shared with Chapter 2 §6; consolidate into one canonical figure.

## Figures NOT included
The CDN architecture diagram for case B is useful but lives better on the companion page where the reader can see real network topology.

## Inline figure-call markers
None inserted in current draft; placement deferred to editorial pass.
