# Data Structures

## TL;DR

A data structure organizes data so the operations you need are cheap and the operations you don't need don't pay rent. Reach for this chapter when you are choosing between a hash map and a tree, designing storage for a workload with mixed access patterns, or asking why your `O(1)` structure feels slow at scale. After consulting it, you can match data structures to operation costs, recognize when a specialized structure is warranted, and avoid the default trap of reaching for the same map every time.

## When the choice of data structure matters

You have a workload with named operations — insert, lookup, delete, range query, top-k, count-distinct, membership test — and a question about which structure makes those operations cheap together. Different structures optimize for different mixes. There is no universal winner.

The signal: *you can list the operations and their relative frequencies*. If the workload is "lookup-heavy, occasional insert, no deletion," a hash table or a sorted array works. If the workload is "frequent insert and delete with bounded range queries," a balanced tree or a skip list earns its place. If the workload is "membership test only, billions of items, false positives acceptable," a bloom filter saves orders of magnitude in memory.

The second signal: *the data structure is the bottleneck*. A profiler shows hot spots in your map's lookup or your queue's pop. Asymptotic bounds say everything is fine. You suspect the structure is wrong for the access pattern. This chapter tells you what to swap in.

The third signal: *you reached for a hash map by default and want to know whether that was right*. Often it was. Sometimes it was not. The misconception engaged in §9 is exactly this default.

## What you need to know first

This chapter assumes Big O notation, amortized analysis, and basic recursion (Chapter 2). The dynamic-table case study in §3 is the canonical worked amortized analysis; if amortized analysis is rusty, re-read Chapter 2 §5 first. Implementations are not provided in full — for working code in your language, consult your standard library or the companion-page implementations.

## Arrays and dynamic arrays

A static array is contiguous memory: `O(1)` access by index, `O(n)` insert at arbitrary position. Dynamic arrays — Python's `list`, Java's `ArrayList`, C++'s `std::vector` — wrap a static array with automatic growth: when full, allocate a buffer twice the size, copy elements over.

The cost of growth is `O(n)` for any single resize. The cost across `n` appends is `O(n)` total — `O(1)` amortized per append. The proof: the geometric series `1 + 2 + 4 + ... + n/2 + n` sums to less than `2n`. The accounting method, the aggregate method, and the potential method (Chapter 2 §5) all give the same bound.

Why this matters for choice: dynamic arrays beat linked lists for almost all production workloads. Both have `O(1)` append amortized. The array's contiguous memory wins on cache locality, prefetching, and constant-factor overhead per element. The linked list earns its place only when arbitrary-position insertion or deletion dominates the workload — and even then, a tree is often the better answer.

## Heaps

A heap is a complete binary tree (filled left-to-right at every level) where every parent dominates its children: `parent ≥ children` for a max-heap, `parent ≤ children` for a min-heap. The complete-tree shape lets you store the heap in an array with arithmetic on indices: parent of `i` is at `(i-1)/2`, children at `2i+1` and `2i+2`.

Operations: insert is `O(log n)` (push to end, sift up). Extract-min/max is `O(log n)` (swap root with last, shrink, sift down). Peek is `O(1)`. Heapify (build a heap from an unordered array) is `O(n)`, not `O(n log n)` — the analysis is a recurrence-tree argument, not a bound from `n` insertions.

**Binary heap** is the workhorse implementation. The dominant use is the priority queue. The dominant algorithmic uses are heap sort (Chapter 4) and Dijkstra's shortest-path algorithm (Chapter 5).

**Binomial heap** supports `O(log n)` merge of two heaps, useful when you need to combine independent priority queues.

**Fibonacci heap** supports `O(1)` amortized decrease-key, which improves Dijkstra's algorithm's bound from `O((V+E) log V)` to `O(E + V log V)` and Prim's MST algorithm similarly. In practice, the constants are large; production implementations rarely use Fibonacci heaps. The bound matters in theory; the constants matter in production. [verify] best-case implementation comparisons on real graphs.

Use a heap when the operations are insert plus extract-min (or extract-max). Use a balanced search tree when you also need lookup by key, range queries, or in-order traversal.

## Balanced search trees

A binary search tree maintains the invariant: for every node, all keys in the left subtree are smaller, all keys in the right subtree are larger. Lookup, insert, and delete are `O(h)` where `h` is the tree's height. The trouble is that adversarial insert order — sorted input, for instance — produces a tree of height `n`, degrading every operation to `O(n)`.

Self-balancing trees enforce `h = O(log n)`. Three common variants.

**Red-black trees** maintain balance with color invariants and rotations. Worst-case `O(log n)` for insert, delete, lookup. The C++ standard library's `std::map` and `std::set`, the Java standard library's `TreeMap` and `TreeSet`, and the Linux kernel's process scheduler use red-black trees. [verify] Linux kernel CFS internals if cited specifically.

**AVL trees** maintain a tighter balance invariant — height difference between subtrees at most 1. Lookups are slightly faster than red-black trees in practice; inserts are slightly slower because rebalancing is more aggressive.

**B-trees** generalize binary search trees to higher branching factors. A B-tree node holds many keys and many children, so the tree is shallow. The dominant use is on-disk storage — databases (PostgreSQL, MySQL InnoDB) and filesystems (ext4, NTFS) use B-trees and B+ trees because each tree-node fetch is a disk page, and shallow trees mean few page fetches per query. [verify] specific database internals.

Use a balanced tree when the workload mixes insertion, deletion, lookup, and ordered-range queries. Range queries are the discriminator: hash tables cannot do them.

## Hash tables

A hash table maps keys to values via a hash function: keys hash to a bucket index, the bucket stores the value. Expected `O(1)` lookup, insert, and delete when the table is well-sized and the hash function is good.

Two common collision strategies. **Separate chaining** stores a linked list (or small array) at each bucket; collisions append to the list. **Open addressing** stores values directly in the bucket array; collisions probe to the next slot via linear probing, quadratic probing, or double hashing. Open addressing has better cache behavior; separate chaining handles high load factors more gracefully.

**Universal hashing** picks a hash function at random from a family of functions, defeating any specific adversarial input. The construction `h_{a,b}(k) = ((ak + b) mod p) mod m` for random `a` and `b` and prime `p > |key set|` is the canonical example.

The expected `O(1)` bound assumes a uniform distribution of keys across buckets. Adversarial input — keys crafted to hit the same bucket — produces `O(n)` lookup. The algorithmic complexity attack on hash tables (Crosby and Wallach, 2003) [verify] is a real production concern: it took down web servers in the early 2000s. Modern languages defend against it by seeding hash functions with per-process random values; check your language's documentation.

Hash tables do not support range queries. They do not iterate in any defined order (unless they're insertion-ordered, like Python `dict` since 3.7 [verify]). They pay a cache penalty for the bucket array if the table grows beyond L2 or L3. They pay a memory overhead for empty buckets at low load factors.

Use a hash table when lookups dominate, the keys do not need ordering, range queries are absent, and the input is either trusted or hash randomization is enabled.

## Bloom filters and probabilistic structures

When the workload is "membership test only" — does this URL appear in the visited set? has this user ID seen this ad? — and the set is large, a bloom filter trades a bounded false-positive rate for orders of magnitude less memory.

**Bloom filter.** A bit array of size `m` and `k` independent hash functions. To insert: hash the element with each of `k` functions, set those bits to 1. To test membership: hash with all `k` functions, if any of those bits is 0 the element is definitely absent; if all are 1 the element is *probably* present (could be a false positive). False negatives are impossible. False positive rate `p ≈ (1 − e^(−kn/m))^k` for `n` inserted elements, optimal at `k = (m/n) ln 2`.

**Count-min sketch.** Estimates the count of each item in a stream. Two-dimensional array of `d` rows and `w` columns; `d` independent hash functions. Insert: for each row, hash and increment. Query: minimum over rows. Bounded overcount with high probability.

**HyperLogLog.** Estimates the cardinality (count of distinct items) in a stream using a few KB of memory regardless of stream size. Used by Redis, BigQuery, Presto, and most analytics systems for `COUNT DISTINCT` queries on billions of items. [verify] specific system implementations.

These structures fail loudly: count-min sketch overestimates, bloom filter false-positives, HyperLogLog has bounded relative error. They are correct given that the user accepts the failure mode. Use them when memory is the constraint and bounded error is acceptable.

## Union-Find

A disjoint-set data structure supporting two operations: `Find(x)` returns the representative of the set containing `x`; `Union(x, y)` merges the sets containing `x` and `y`. The dominant use is Kruskal's MST algorithm (Chapter 6) and connectivity queries in dynamic graphs.

The naive implementation gives `O(n)` per operation. Two optimizations together give nearly-constant amortized time:

**Path compression** flattens the tree during `Find`: every node on the path from `x` to the root becomes a direct child of the root.

**Union by rank** (or by size) merges the smaller tree into the larger one, keeping trees shallow.

With both, the amortized cost per operation is `O(α(n))`, where `α` is the inverse Ackermann function — effectively constant for any `n` you will encounter (it exceeds 4 only for `n` larger than the number of atoms in the observable universe). [verify] specific Tarjan 1975 paper attribution.

Use union-find for connected-components queries on a dynamic graph. Use it for Kruskal's. Otherwise, a hash table mapping element to representative is simpler and competitive at small scale.

## Decision rules

| Workload signature | Recommended structure |
| --- | --- |
| Index-by-position, append-heavy | Dynamic array |
| Lookup-by-key only, no order | Hash table |
| Lookup, insert, delete, ordered iteration, range queries | Balanced BST (red-black, AVL) |
| On-disk storage, large pages | B-tree or B+ tree |
| Insert + extract-min only | Binary heap |
| Insert + decrease-key + extract-min, large graph | Fibonacci heap (theory) or binary heap (practice) |
| Membership test, large set, false positives OK | Bloom filter |
| Count distinct over a stream | HyperLogLog |
| Frequency estimate over a stream | Count-min sketch |
| Connected components in a dynamic graph | Union-Find with path compression and union-by-rank |

## Worked example — real-time analytics storage

A real-time analytics service ingests events at 100,000 per second [verify]. Each event has a timestamp, a user ID, an event type, and a payload. Three queries dominate the workload:

1. **Top-k event types in the last 5 minutes.**
2. **Count of distinct users who saw event type X in the last hour.**
3. **All events of type X with timestamp in `[t1, t2]`.**

A naive design uses a single hash map from event type to a list of events. Top-k requires a full scan. Count-distinct requires deduplicating user IDs, which costs memory proportional to the number of distinct users. Range queries on timestamp are linear scans. At 100K events per second, none of this scales.

The right design uses three structures, each tuned to one query.

**Top-k.** A min-heap of size `k` keyed on event-type counts, refreshed by a sliding window. Each event increments a counter for its event type; if the count exceeds the heap's minimum, the heap is updated. Cost per event: `O(log k)`. The window slides by expiring counts older than 5 minutes, which can be implemented with a circular buffer of one-minute windows. The heap holds the top-k summary; the sliding window holds the per-minute breakdown.

**Count-distinct.** A HyperLogLog sketch per event type, refreshed every minute. The sketch costs about 12 KB per event type for ≤2% relative error on cardinalities up to billions [verify]. Compare to a hash set, which would cost `O(d)` memory in the count of distinct users — gigabytes for a large workload. The sketch trades a small bounded error for orders of magnitude less memory and constant-time updates.

**Range queries.** A B+ tree keyed on `(event_type, timestamp)`. Range queries on timestamp within a single event type become a leaf-level scan of contiguous nodes. Cost per query: `O(log n + k)` where `k` is the result size. This is what databases do; we are not reinventing the wheel, we are reaching for the structure databases reach for.

The naive single-hash-map design failed because it tried to support three different access patterns with one data structure. The right design uses three structures because the workload has three operation profiles. None of the structures is exotic; the discrimination is matching structure to operation, not inventing structures.

## Failure modes — the "use a hash map by default" trap

The misconception: a hash map's `O(1)` lookup makes it the right default for any keyed access.

Three concrete failure modes.

**Range queries.** Hash maps cannot answer "all keys between `a` and `b`" in any sub-linear way. The keys are not stored in any order. If your access pattern includes range queries — timestamps, IDs, lexical prefixes — you need a tree or a sorted structure. The hash map's bound is great for the operations it supports; it does not support yours.

**Cache pressure at scale.** A hash map with ten million entries does not fit in L2 cache on most hardware. Random-access lookup misses cache, fetches from RAM, pays the 100× latency hit. A sorted array of the same data, accessed by binary search, has worse asymptotic cost (`O(log n)` vs `O(1)`) but often wins on wall-clock time because each access is in a contiguous block that prefetches well. Asymptotic bounds describe the field, not the conclusion. The corrective heuristic: when the structure is sized larger than your last-level cache and access is random, measure both.

**Adversarial input.** A public web service that hashes user-supplied strings is vulnerable to collision attacks unless the hash function is keyed by a per-process random secret. The `O(1)` bound is *expected* over a uniform input distribution; an attacker who controls the input distribution can drive every operation to `O(n)`. Modern languages default to keyed hashes, but configurations exist (custom hash functions, low-level library use) where this protection is absent.

**Memory overhead at low load factor.** A hash map with one million entries and a 50% load factor uses memory for two million bucket slots. A balanced tree with the same one million entries uses memory for one million nodes plus pointer overhead. The tree is often more memory-efficient at moderate scale. If memory is the constraint, the default isn't free.

The corrective: state the operations you need (lookup, ordered iteration, range query, membership test, decrease-key, merge), state the working-set size relative to cache, state whether the input is adversarial. The data structure choice falls out. The hash map is a fine default; it is not a universal default.

## Cross-references

For amortized analysis worked through dynamic tables, see Chapter 2 §5. For heap sort and the heap-as-priority-queue use, see Chapter 4. For Dijkstra and Prim using heaps, see Chapter 5. For Kruskal using union-find, see Chapter 6. For randomized analysis of hashing and the algorithmic complexity attack, see Chapter 12. For LSH and other randomized data structures, see Chapter 12.

## Companion-page handoffs

Implementation comparisons across Python, Java, and C++ standard libraries; specialized structures deep-dives — bloom filter sizing calculator, count-min sketch implementations, skip lists, treaps, persistent data structures; benchmark harnesses comparing tree variants and hash-table strategies on representative workloads. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-3.

## What this chapter does not enable

This chapter does not give you implementation-ready code for every structure. Standard libraries do that, and reimplementing a red-black tree by hand is rarely the right call in production. The chapter also does not cover persistent data structures (immutable structures with cheap "modified" copies), succinct data structures (information-theoretically near-minimal storage), or the engineering tricks that distinguish a textbook hash table from `absl::flat_hash_map`. Those live on the companion page or in specialized references.

## Capability statement

You can now match data structures to operation profiles, predict where the structure will be the bottleneck, recognize when a specialized structure is warranted, and avoid the default trap of reaching for the same map every time. The next time a workload arrives with a mix of access patterns, you can name the structure for each pattern, justify the choice from operation costs, and explain to a reviewer why the default would have failed.


---

## LLM Exercise — Chapter 3: The Structure Library

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** A library of six canonical data structures, implemented from scratch, with operation costs verified empirically by the harness from Chapter 2.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 3 task in the algorithms-by-bear-toolkit. The book's central
misconception target for this chapter is "use a hash map by default."
The exercise builds the structures and forces the empirical comparison.

In `chapters/ch03_data_structures/`:

1. `implementations.py` — implement, from scratch (do not delegate the
   structure itself to a stdlib container):

   - `DynamicArray` — geometric resizing (×2), with `append`, `__getitem__`,
     `__setitem__`, `__len__`.
   - `HashMap` — open addressing with linear probing, load-factor-
     triggered resize at 0.7, with `put`, `get`, `delete`, `__len__`.
   - `MinHeap` — array-backed, with `push`, `pop`, `peek`, `heapify`.
   - `UnionFind` — union-by-rank with path compression.
   - `BloomFilter` — configurable bit array and number of hash functions.
     Include a `false_positive_rate(n_inserted)` method that returns
     the *theoretical* rate from `(1 - exp(-kn/m))^k`.
   - `SkipList` — probabilistic with coin-flip level assignment
     (probability 0.5). `insert`, `search`, `delete`.

   Each implementation: docstring with claimed operation costs and a
   one-paragraph "when to reach for this vs. the stdlib equivalent."

2. `test_implementations.py` — correctness tests against stdlib
   equivalents on random workloads. Use `hypothesis` for property-based
   tests where feasible.

3. `benchmarks.py` — import `harness` from
   `chapters.ch02_algorithm_analysis.harness`. Verify empirically:

   - `HashMap` insert and lookup degradation as load factor approaches
     resize threshold. Plot performance vs. load factor.
   - `MinHeap` push/pop matches O(log n).
   - `UnionFind` near-constant amortized cost.
   - `BloomFilter` empirical false-positive rate vs. theoretical
     prediction at multiple fill levels.
   - `SkipList` insert/search empirically O(log n) with bounded variance.

   For each: use `harness.fit_growth_rate` and assert the empirical
   best-fit matches the claim.

4. `README.md` — six decision cards. "Surprising findings" reports at
   least: the load factor at which your `HashMap` degrades, compared to
   the equivalent measurement on Python's built-in `dict`.

Commit with `ch03: canonical data structures with verified operation
costs`.
```

**What this produces:** Six tested data-structure implementations (~600 lines), six populated decision cards, empirical verification that operation costs match theory, and at least one surprising-finding note grounded in actual benchmark output.

**How to adapt this prompt:**

- *For your own project:* If your domain uses specialized structures (count-min sketch, HyperLogLog, treap), swap one in for the skip list or add a seventh.
- *For ChatGPT / Gemini:* Manageable but slower without Claude Code. Ask for one structure at a time, write the test, then move on.
- *For Claude Code:* Native fit. Let it iterate — write, test, fix, benchmark, repeat — on each structure before moving to the next.
- *For a Claude Project:* Pair with Claude Code. Use the Project to hold the running notebook (Project Option 1) if you want both artifacts.

**Connection to previous chapters:** Imports `harness.time_function` and `harness.fit_growth_rate` from Chapter 2. Each decision card populates the Chapter 1 template.

**Preview of next chapter:** Chapter 4 implements the sort catalog (insertion, merge, quick, heap, Timsort) and finds the empirical crossover where insertion sort beats merge sort on small subarrays — Bear's signal at n ≈ 32.
