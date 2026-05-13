# Figure brief — Chapter 03: Data Structures

## Recommended figures

### Figure 1 — Operation cost matrix
**Path:** `../images/chapter-3-operation-cost-matrix.jpg`
**Description:** Heat-map matrix. Rows: data structures (Dynamic Array, Linked List, Binary Heap, BST, Red-Black Tree, B-Tree, Hash Table, Bloom Filter, Union-Find, HyperLogLog, Count-Min Sketch). Columns: operations (Insert, Delete, Lookup, Min/Max, Range Query, Membership, Count Distinct, Decrease-Key, Merge). Cell color: green for `O(1)` or `O(log n)`, yellow for `O(n)` or amortized, gray for "not supported." Cells annotated with the bound.
**Use:** referenced in Decision rules.

### Figure 2 — Heap as array layout
**Path:** `../images/chapter-3-heap-array-layout.jpg`
**Description:** Side-by-side: a complete binary tree with nodes labeled 0–9, and the corresponding array `[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`. Arrows showing parent-of-`i` at `(i-1)/2` and children at `2i+1`, `2i+2`. Caption explains the index arithmetic.
**Use:** referenced in §4 (Heaps).

### Figure 3 — Bloom filter operation
**Path:** `../images/chapter-3-bloom-filter.jpg`
**Description:** Bit array of size 16, three hash functions. Two elements being inserted, with arrows from the hash functions to the bits being set. A third element being tested, with one bit hitting a 0 (definite "not present"). Caption explains why false negatives are impossible.
**Use:** referenced in §7 (Bloom filters).

### Figure 4 — Real-time analytics architecture
**Path:** `../images/chapter-3-realtime-analytics.jpg`
**Description:** Block diagram. Event stream → three branches: (1) sliding-window heap for top-k, (2) HyperLogLog sketches for count-distinct, (3) B+ tree for range queries. Each block annotated with operation cost. Caption: "One workload, three structures, each tuned to one query."
**Use:** referenced in §8 (Worked example).

### Figure 5 — Union-Find with path compression
**Path:** `../images/chapter-3-union-find-path-compression.jpg`
**Description:** Two side-by-side trees showing before/after path compression on a Find operation. Before: chain of length 5. After: all nodes attached directly to root.
**Use:** referenced in §8 (Union-Find).

## Figures NOT included
The bloom filter false-positive curve as a function of `m/n` and `k` is a useful figure but lives better on the companion page where the reader can experiment.

## Inline figure-call markers
None inserted in current draft; placement deferred to editorial pass for layout integration.
