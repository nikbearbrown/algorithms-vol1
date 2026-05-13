# Sorting and Caching

## TL;DR

Sorting orders elements; caching keeps frequently-accessed elements close. The two appear in one chapter because both are dominated by the memory hierarchy in production, both have a small canonical algorithm catalog, and both have a "use the standard library" answer that hides the more interesting question. Reach for this chapter when you need to choose a sorting algorithm under specific constraints — size, memory, distribution, stability — or pick a cache replacement policy for a workload. After consulting it, you can match algorithm to constraint and explain why your standard library's choice is usually right.

## Recognition pattern

You have data to sort, or a working set to keep in fast memory, and the question is *which strategy*.

For sorting, the signal is *constraint*. Constraints come in five flavors. Size — does the data fit in RAM? Memory — can you afford `O(n)` auxiliary space? Distribution — is the data nearly sorted, uniformly random, or pathological? Stability — must equal-key elements preserve relative order? Adversarial input — is the order under attacker control? Each constraint rules out a candidate. The right algorithm is the one that survives.

For caching, the signal is *access pattern*. The reuse-distance distribution determines which replacement policy wins. Workloads with strong recency win on LRU. Workloads with strong frequency win on LFU. Workloads that mix both win on ARC or its descendants. Workloads with neither lose on every policy and need a different intervention.

A signal that this chapter is *not* the right one: the sort or cache is provided by the platform — your database does the sort, the CPU does the cache — and you have no control. In that case the chapter helps you reason about what the platform is doing; it does not help you tune what you cannot tune.

## What you need to know first

This chapter assumes Big O, recurrences, and the master theorem (Chapter 2 §3–4). The cache discussion assumes the memory-hierarchy reasoning from Chapter 2 §6. The heap-sort section refers back to heaps (Chapter 3 §4). For randomized quicksort's analysis, see Chapter 12.

## The classical sort catalog

Six algorithms cover almost every working programmer's needs. The table at the end consolidates; the prose below names the use case each one earns.

**Bubble sort**, **selection sort**, **insertion sort** — the `O(n²)` family. All three build the sorted result one element at a time. Bubble sort sweeps repeatedly, swapping adjacent out-of-order pairs; selection sort scans for the minimum and places it; insertion sort takes each element and shifts it leftward into the sorted prefix. Of the three, insertion sort is the only one that earns a place in production: it has `O(n)` best-case on already-sorted input, low constants, excellent cache behavior, and is what every standard library calls for small subarrays. Bubble sort is a teaching tool; selection sort is rarely the right call.

**Merge sort** — `O(n log n)` worst-case, `O(n)` auxiliary space, stable. Divide the array in half, recursively sort each half, merge. The merge step touches each element once per level and there are `log n` levels, giving the bound. Merge sort wins when stability is required (database sorts, ordered iteration over equal-key records) and memory is available. It is the dominant external-memory sort because the merge step is sequential and streams nicely from disk.

**Quicksort** — `O(n log n)` expected, `O(n²)` worst-case, `O(log n)` stack space, not stable. Pick a pivot, partition the array around it (smaller on left, greater on right), recurse on both sides. The worst case occurs on already-sorted input with a fixed pivot policy; randomized pivot selection (Chapter 12) makes the worst case essentially never hit. Quicksort wins when memory is tight (in-place partitioning) and stability does not matter. It is the dominant in-memory sort for primitives.

**Heap sort** — `O(n log n)` worst-case, `O(1)` auxiliary space, not stable. Build a max-heap (Chapter 3 §4), then repeatedly extract the maximum into the back of the array. Heap sort wins when worst-case bounds matter and memory is constrained. It loses to quicksort on average because cache behavior is worse — heap operations jump across the array — but the worst case is guaranteed.

**Counting sort, radix sort, bucket sort** — non-comparison sorts. Counting sort runs in `O(n + k)` for `n` elements and `k` distinct keys; radix sort runs in `O(d(n + k))` for `d` digits per key. Both beat the `Ω(n log n)` lower bound for comparison sorts because they exploit structure (bounded key range, fixed-width keys). They win when keys are integers in a known small range. They lose when keys are arbitrary objects, when the key range is huge, or when the cache penalty of multiple passes over the data dominates.

## Why your standard library hybridizes

Three production sorts dominate the major language ecosystems. Each is a hybrid; none is one of the catalog algorithms in pure form.

**Timsort** is CPython's `list.sort()` since version 2.3 (2002) [verify], Java's `Arrays.sort()` for objects since 7 (2011) [verify], and the basis for many other implementations. Timsort detects "natural runs" — already-sorted ascending or descending subsequences — and merges them. On real-world data, which is often partially sorted (timestamps, IDs, log entries), it approaches `O(n)` rather than `O(n log n)`. On unsorted input it falls back to merge sort. Subarrays below `MIN_RUN` (computed from input length, typically between 32 and 64) [verify] are sorted with binary insertion sort — the cache-locality argument from Chapter 2 §6.

**Introsort** is the C++ standard library's `std::sort` and similar implementations. Introsort starts as quicksort, switches to heap sort when recursion depth exceeds `2 log₂ n` (preventing the `O(n²)` worst case), and switches to insertion sort for subarrays smaller than 16 [verify]. Three algorithms, one wrapper.

**Dual-Pivot Quicksort** is the JDK's `Arrays.sort()` for primitive arrays since Java 7 (2011) [verify]. Two pivots partition the array into three regions, with insertion sort below 47 elements [verify]. Two pivots reduce the average comparison count by about 5% and improve cache behavior.

The pattern: production sorts pay attention to constants, cache locality, and constraint-specific optimization. Asymptotic dominance is a necessary condition, not a sufficient one.

## Cache eviction strategies

When fast memory is full and a new item arrives, an eviction policy decides which item leaves. The right choice depends on the workload's reuse pattern.

**LRU (Least Recently Used).** Evict the item accessed least recently. Implemented as a doubly-linked list plus a hash map: the map points to list nodes, the list keeps recency order, both updates are `O(1)`. LRU is the workhorse policy. It assumes recent access predicts future access — temporal locality — and most workloads honor that assumption.

**LFU (Least Frequently Used).** Evict the item with the lowest access count. Useful when popularity is more stable than recency: a CDN serving long-tail content has a stable set of popular items that should stay cached even if a recent burst of unique requests pushes them out under LRU.

**FIFO (First-In, First-Out).** Evict the oldest item regardless of recent access. Simpler than LRU; performs worse on most workloads. Earns its place when implementation simplicity matters more than hit rate.

**Random replacement.** Evict an item chosen uniformly at random. Surprisingly competitive on many real workloads — within a few percent of LRU — because the variance averages out at scale. Used in some hardware caches because the randomness defeats pathological access patterns and the implementation cost is near zero.

**ARC (Adaptive Replacement Cache).** Maintains two LRU-like lists, one for recently-once-accessed items and one for repeatedly-accessed items, plus ghost lists tracking recently-evicted items. The split adapts based on which kind of access is dominating. ARC outperforms LRU on workloads with mixed recency/frequency patterns. The patent situation — IBM held patents that expired around 2019 [verify] — slowed adoption; today, ARC and its descendants (CAR, CLOCK-Pro) appear in storage systems, including ZFS [verify] and PostgreSQL's experimental work.

The eviction policy is one knob. Cache size is another. The relationship between them is non-linear — doubling the cache often improves hit rate non-trivially up to a knee, then plateaus. A workload that misses by 30% with a 1 GB cache may miss by 5% with a 4 GB cache and 4.8% with a 16 GB cache. Tune the size before tuning the policy.

## Decision rules

| Workload signature | Sort choice |
| --- | --- |
| Subarray smaller than ~32 | Insertion sort |
| Stability required, memory available | Merge sort or Timsort |
| Memory tight, stability irrelevant, average case matters | Quicksort (with randomized pivot) |
| Worst-case bound required, memory tight | Heap sort or introsort |
| Keys are integers in a small known range | Counting sort or radix sort |
| Real-world partially-sorted data | Timsort |
| Don't know — use language default | Standard library sort (almost always right) |

| Workload signature | Cache policy |
| --- | --- |
| Strong temporal locality | LRU |
| Stable popularity, long-tail | LFU |
| Hardware-level cache, simplicity matters | Random or FIFO |
| Mixed recency/frequency | ARC or CLOCK-Pro |
| Don't know — measure before choosing | LRU as default; profile before tuning |

## Worked example — two real cases

### Case A — Why Timsort beats classical merge sort on real Python data

A backend service ingests log entries arriving roughly in timestamp order, with occasional out-of-order events from clock drift. The service sorts batches of 100,000 entries before writing to long-term storage. The first implementation called classical merge sort. Profiling showed sort time was ~12% of batch runtime [verify].

Switching to CPython's `list.sort()` cut sort time to ~2% [verify]. The reason: real log data is dominated by long natural ascending runs. Timsort detects runs of length ≥ `MIN_RUN` (typically 32–64) and merges them rather than splitting and re-sorting. On nearly-sorted input, Timsort approaches `O(n)`; on random input, it falls back to standard merge-sort behavior at `O(n log n)`.

Three properties combine to make Timsort the right call. First, *natural-run detection*: it does not pay the `log n` factor on data it does not need to. Second, *stability*: equal timestamps preserve their original ordering, which matters when log entries within the same millisecond should retain ingestion order. Third, *cache behavior*: small subarrays use binary insertion sort, which streams through L1.

The lesson: the asymptotic bound on classical merge sort was correct. The real-world workload had structure the bound did not exploit. The right algorithm exploited it.

### Case B — Cache replacement in a CDN

A CDN edge server caches static assets. A typical workload mixes long-tail popularity (a stable set of widely-requested objects) with bursts of unique requests (one-off hits, long URL tails, crawler traffic). With a 1 TB edge cache and a 5 GB working set of "always-warm" objects [verify], the policy choice has measurable consequences.

Under LRU, a burst of unique requests evicts warm items. The next request for a warm item misses and is re-fetched. Hit rate is meaningfully lower than the working-set ratio would suggest.

Under LFU, the burst increments counters for unique items but does not displace warm items unless the unique items are accessed repeatedly. Warm items stay cached. Hit rate improves on workloads where popularity is stable.

Under ARC, the two-list structure separates "items accessed once recently" from "items accessed multiple times." Bursts populate the first list; warm items live in the second. Hit rate improves further on mixed workloads.

The reported gains in published CDN literature are 10–30% in hit rate when moving from LRU to a recency+frequency-aware policy on appropriately-shaped workloads [verify]. The intervention pays back in reduced origin fetch traffic, lower bandwidth costs, and faster time-to-first-byte for end users.

The lesson: the caching policy is not a one-size-fits-all decision. Profile the access pattern, identify the recency-frequency split, choose accordingly. A misconfigured LRU on a long-tail workload is a real source of cost.

## Failure modes — when "quicksort is always fastest" misleads

The misconception engaged: "Quicksort is always fastest."

Quicksort has the best average-case constants among comparison sorts on random integer data fitting in RAM. That is the reading the misconception is built on. It is wrong as a universal claim.

**Worst-case `O(n²)` on adversarial input.** Naive quicksort with a fixed pivot policy (first element, last element, middle element) can be driven to `O(n²)` by an adversary who controls the input. Already-sorted input with first-element pivot is the canonical example. A public web service that accepts user-supplied lists and sorts them is vulnerable. Mitigation: randomize the pivot (Chapter 12) or use introsort (which falls back to heap sort when recursion goes too deep).

**No stability.** Equal keys do not preserve relative order. Database sorts on a secondary key require stability to honor a primary-key tiebreaker. Quicksort cannot be used directly; merge sort or Timsort is the right call.

**Cache penalty on small data.** For subarrays below ~32 elements, the recursive overhead of quicksort exceeds the linear scan of insertion sort. Every production sort calls insertion sort at the leaves for this reason.

**Loss to non-comparison sorts on bounded-range keys.** Sorting one billion 32-bit integers, radix sort runs in `O(n)` per pass with a small number of passes. Quicksort runs in `O(n log n)` with worse cache behavior. Radix sort wins on this specific shape.

**Loss to merge sort on disk-resident data.** External-memory sorting prizes sequential access. Merge sort streams; quicksort jumps. The right external sort is multi-way merge.

The corrective heuristic: state the constraint, identify the algorithm that survives the constraint, then verify with a benchmark on representative data. The standard library's choice is the right default; departures from the default need justification.

## Cross-references

For heap operations underlying heapsort, see Chapter 3 §4. For randomized quicksort and the analysis that justifies expected `O(n log n)`, see Chapter 12. For the master theorem applied to merge sort's recurrence, see Chapter 2 §4. For the cache-effects argument underlying every "production sort hybridizes" claim, see Chapter 2 §6.

## Companion-page handoffs

Benchmark suite comparing sorts on representative workloads (random integers, partially-sorted timestamps, near-duplicate strings, adversarial inputs); LRU, LFU, ARC, and CLOCK-Pro implementations with hit-rate harnesses; real cache-strategy tuning examples on workload traces; profiler-driven crossover analysis. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-4.

## What this chapter does not enable

This chapter does not enable hardware-level cache optimization. CPU cache tuning — line size selection, associativity tuning, prefetcher behavior — is architecture-specific and lives in the optimization manuals from Intel and AMD, not in an algorithms reference. The chapter also does not cover external-memory algorithms in depth (multi-way merge, B-tree-based sorts, parallel external sort); for that, consult Vitter's *Algorithms and Data Structures for External Memory* or modern database internals texts.

## Capability statement

You can now choose an appropriate sorting algorithm given size, memory, distribution, and stability constraints; distinguish comparison sorts from non-comparison sorts and know when each wins; explain why your language's standard library hybridizes the way it does; pick a cache replacement strategy from access-pattern signals; and recognize when a workload deserves something more than the LRU default. The next time a sort or cache becomes the bottleneck, the diagnosis and the fix are in your hands.


---

## LLM Exercise — Chapter 4: Sort Catalog and Cache Simulator

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** The sort catalog (insertion, merge, quick, heap, plus a hybrid Timsort-style sort) and a cache-replacement simulator covering LRU / LFU / ARC, with the empirical crossover findings the chapter signals.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 4 task in the algorithms-by-bear-toolkit. The chapter's
signature findings are (a) the insertion-sort/merge-sort crossover at
small n (Bear notes ~32) and (b) the 10–30% hit-rate impact of choosing
the right cache replacement strategy. The exercise reproduces both
empirically.

In `chapters/ch04_sorting_and_caching/`:

1. `implementations.py`:

   - Sorts: `insertion_sort`, `merge_sort`, `quicksort` (with random
     pivot — note that the analysis lives in Chapter 12), `heapsort`
     (using the `MinHeap` from Chapter 3 — import it), and
     `hybrid_sort` that delegates to `insertion_sort` below a
     threshold parameter and `merge_sort` above. The threshold is a
     constructor parameter so we can sweep it.
   - Caches: `LRUCache`, `LFUCache`, `ARCCache` (Adaptive Replacement
     Cache — split it into T1/B1/T2/B2 lists as in Megiddo & Modha
     2003). Each cache exposes `get`, `put`, and a `hit_rate` property.

2. `test_implementations.py` — sort correctness against Python's
   `sorted()` on random, sorted, reverse, and near-sorted inputs.
   Cache correctness verified against a brute-force reference.

3. `benchmarks.py` — two studies:

   **Study A: the insertion/merge crossover.** Use `harness.time_function`
   to time `insertion_sort` and `merge_sort` on input sizes
   `[8, 16, 32, 64, 128, 256, 512, 1024]`. Plot both curves on the
   same axes. Identify the empirical crossover point. Then sweep
   `hybrid_sort` threshold values `[8, 16, 32, 64, 128]` on a large
   input and find the threshold that minimizes total time. Report
   both findings.

   **Study B: cache-replacement impact.** Simulate three workloads —
   (i) recency-dominant: a Zipf distribution favoring recent items;
   (ii) frequency-dominant: a static Zipf with no recency drift;
   (iii) mixed: alternating phases. For each workload, run LRU / LFU /
   ARC at multiple cache sizes (1%, 5%, 10%, 20% of working set) and
   plot hit rate vs. cache size. Report each cache's win region.

4. `README.md` — six decision cards (five sorts + a "caching policies"
   card). "Surprising findings" must include: (a) the empirical
   crossover number you found and how it compares to Bear's ~32 hint;
   (b) the magnitude of the cache-policy hit-rate spread on your
   mixed workload.

Commit with `ch04: sort catalog with crossover study and cache
simulator`.
```

**What this produces:** Five sort implementations + three cache implementations (~700 lines), two empirical studies with plots, decision cards, and surprising-finding notes grounded in your actual numbers.

**How to adapt this prompt:**

- *For your own project:* Replace the Zipf workloads with traces from your real system if you have them — production cache logs make this study far more interesting than synthetic Zipf.
- *For ChatGPT / Gemini:* The cache implementations are subtle (ARC especially); ask for it in isolation and run it through a test suite before integrating.
- *For Claude Code:* Native fit. Have it verify cache correctness against a brute-force reference *before* running the policy comparison — otherwise you may be comparing buggy implementations.
- *For a Claude Project:* Skip; this is code-heavy.

**Connection to previous chapters:** Imports `harness` from Chapter 2 and `MinHeap` from Chapter 3.

**Preview of next chapter:** Chapter 5 implements graph representations and the four classical shortest-path algorithms, with a worked routing example you can compare across Dijkstra, Bellman-Ford, and A*.
