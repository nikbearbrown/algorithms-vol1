# Chapter 4 — Sorting and Caching

*The memory hierarchy is the physics. The algorithm is the engineering.*

---

Here is a question that seems simple until you think about it: why does your standard library sort faster than the algorithm your algorithms course taught you?

The textbook answer to sorting is merge sort or quicksort — both `O(n log n)`, both correct, both reasonable. And yet CPython's `list.sort()` routinely outperforms a clean merge sort implementation on real data by a factor of five or six. Java's `Arrays.sort()` for objects is faster still. Neither of them is pure merge sort. Neither of them is pure quicksort. They are hybrids — three or four algorithms stitched together, each taking over when it has the advantage.

Understanding why that is true requires understanding two things: the constraint that makes each algorithm win, and the physical reality that every algorithm runs inside. The physical reality is the memory hierarchy. The constraint determines which algorithm survives it.

This chapter covers sorting and caching together because they are, at bottom, the same problem viewed from two angles. Sorting asks: in what order should I arrange these elements? Caching asks: which elements should I keep close? Both questions are dominated by the same physics — the cost of moving data between slow memory and fast memory — and both have a canonical set of strategies that differ in which constraints they handle. Learn the constraints, and you learn when each strategy wins.

---

## The lower bound and what it does not tell you

Before naming algorithms, it is worth understanding what the theory actually says and what it leaves open.

Any algorithm that sorts by comparing elements must make at least `Ω(n log n)` comparisons in the worst case. The argument is clean: there are `n!` possible orderings of `n` elements, and each comparison eliminates at most half of them. To distinguish all `n!` cases requires at least `log₂(n!)` comparisons, which by Stirling's approximation is `Ω(n log n)`. Merge sort and heapsort achieve this bound in the worst case. Quicksort achieves it in the expected case.

What the lower bound does not tell you: which algorithm is fastest on your specific data. The bound is for the worst case over all possible inputs. Real data has structure — partial sortedness, bounded key ranges, repeated values, predictable access patterns — and algorithms that exploit that structure can beat the worst-case bound on inputs that do not trigger it.

This is the gap between the theory and the engineering. The theory tells you no comparison sort can do better than `O(n log n)` in the worst case. The engineering asks what structure your data actually has, and finds an algorithm that exploits it. Timsort is the canonical answer to that question for general-purpose sorting.

![Two curves on the same axes ](images/04-sorting-and-caching-fig-01.png)
*Figure 4.1 — Two curves on the same axes *

---

## The classical algorithms and what each one is actually for

Six algorithms cover almost every working programmer's needs. The organizing principle is constraint: which properties of the problem does the algorithm require, and which does it sacrifice?

**Insertion sort** is the algorithm that belongs at the base of every other sort. Take each element in sequence and shift it leftward into the sorted prefix until it lands in the right position. Worst case `O(n²)` on reverse-sorted input, best case `O(n)` on already-sorted input. For small subarrays — below 32 elements, roughly — insertion sort is faster than merge sort or quicksort in practice, because the constants are tiny and the access pattern is sequential and cache-friendly. No production sort ignores this. Every standard library calls insertion sort on small subarrays. It is not a toy algorithm; it is the algorithm that wins when the array is small enough that the `O(n log n)` factor has not yet dominated the constant.

**Merge sort** divides the array in half, recursively sorts each half, then merges the two sorted halves into a single sorted sequence. Every element is touched once per level of recursion, and there are `log n` levels, giving `O(n log n)` in all cases — best, average, and worst. Merge sort requires `O(n)` auxiliary space for the merge buffer. It is stable: elements with equal keys preserve their original relative order. Stability is not a nicety when you are sorting database records by a secondary key and need the primary-key order preserved among ties. For that use case, merge sort — or a merge-based hybrid — is the right choice, and stability rules out quicksort.

**Quicksort** picks a pivot, partitions the array around it (elements smaller than the pivot on the left, larger on the right), and recurses on both sides. Expected `O(n log n)`, worst case `O(n²)` when the pivot repeatedly lands at the extreme — as it does on already-sorted input with a fixed first-element pivot. Quicksort does the partitioning in-place, requiring only `O(log n)` stack space. It is not stable. On random data in RAM, quicksort's constants are better than merge sort's: the partition step has excellent cache behavior, touching the array sequentially from both ends toward the middle. The expected case is fast enough that production systems reaching for an in-place sort with no stability requirement default to quicksort or a quicksort-based hybrid.

**Heapsort** builds a max-heap from the input (Chapter 3 §4) and then repeatedly extracts the maximum, placing it at the back of the array. `O(n log n)` worst case, `O(1)` auxiliary space, not stable. Heapsort's theoretical properties are hard to beat: worst-case optimal, in-place. Its practical performance is worse than quicksort because heap operations jump across the array in a pattern that is hostile to the CPU's prefetcher. The useful case for heapsort is when you need a guaranteed bound and cannot tolerate quicksort's worst case — or when you need to extract the top-k elements without sorting everything, which is just k heap extractions: `O(n + k log n)`.

**Counting sort and radix sort** break the `Ω(n log n)` comparison-sort lower bound by not comparing elements. Counting sort builds a frequency table of key values, then reconstructs the sorted output by walking the table. It runs in `O(n + k)` for `n` elements with `k` distinct key values. Radix sort processes keys digit by digit, stable-sorting on each digit from least significant to most. It runs in `O(d(n + k))` for keys of `d` digits in base `k`. Both algorithms win when keys are integers in a known bounded range. Both lose when keys are arbitrary objects — strings of unbounded length, composite keys, anything you cannot decompose into a fixed-width integer representation. The other failure mode is when `k` is large relative to `n`: counting sort on a billion-element array with ten billion distinct keys requires a ten-billion-slot table, and the memory access pattern for a large table destroys the cache advantage.

| algorithm | best case | average case | worst case | auxiliary space |
| --- | --- | --- | --- | --- |
| , wins when | reader sees the constraint matrix at a glance and uses it alongside the decision rules table below | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Why production sorts hybridize

The three dominant production sorts are all hybrids. Understanding each one reveals what the engineers were solving for.

**Timsort** is CPython's `list.sort()` since 2002 and Java's `Arrays.sort()` for objects since Java 7. The core insight: real data is not random. Log files arrive roughly in timestamp order. User records were inserted in roughly alphabetical order. Sensor readings arrive in time order with occasional late arrivals. Data that humans or automated systems generate tends to have partially-sorted structure, because the processes that generated it had temporal or alphabetical ordering themselves.

Timsort detects natural runs — contiguous subsequences that are already sorted ascending or descending — and merges them. On data with long natural runs, the effective input size to the merge step is the number of runs, not the number of elements. In the extreme case of already-sorted data, Timsort runs in `O(n)`: one run, nothing to merge. On genuinely random data, run detection finds only short runs and falls back to standard merge behavior at `O(n log n)`. For subarrays below a threshold (computed from input size, typically 32–64 elements), Timsort calls binary insertion sort, getting the small-array advantage. The result is a sort that is fast on the inputs humans actually produce, competitive on random input, and stable throughout.

**Introsort** is the algorithm behind C++'s `std::sort`. Three algorithms, one wrapper. It starts as quicksort — best-case and average-case behavior, good cache utilization. When the recursion depth exceeds `2 log₂ n`, it detects that the pivot selection has gone pathological and switches to heapsort, getting worst-case `O(n log n)` at the cost of slightly worse constants. For subarrays smaller than 16 elements, it switches to insertion sort. The result is an algorithm that captures quicksort's average-case speed, heapsort's worst-case guarantee, and insertion sort's small-array advantage, without any algorithm having to handle cases it is bad at.

**Dual-pivot quicksort** is the JDK's `Arrays.sort()` for primitive arrays since Java 7. Instead of one pivot dividing the array into two regions, two pivots divide it into three. The average comparison count decreases by roughly 5%, and the three-region structure improves cache behavior slightly. For subarrays below 47 elements, it calls insertion sort. The engineering motivation was straightforward: the JVM is fast enough for object sorts to use Timsort, but primitive array sorts need better constants, and dual-pivot quicksort's constant is measurably better than single-pivot on random data.

The pattern is consistent across all three. Asymptotic dominance is a necessary condition for being considered; it is not sufficient for winning in practice. What wins is a hybrid that matches each subproblem to the algorithm whose strengths fit that subproblem's profile.

![Diagram showing Timsort, Introsort, and Dual-Pivot Quicksort as](images/04-sorting-and-caching-fig-02.png)
*Figure 4.2 — Diagram showing Timsort, Introsort, and Dual-Pivot Quicksort as*

---

## Caching and the same problem in different clothing

Sorting and caching seem like different problems. They share a structure: both are strategies for managing the gap between slow memory and fast memory. Sorting manages it by arranging data so it can be accessed in order. Caching manages it by keeping the right data in the fast layer.

A cache is a fixed-size buffer sitting between a slow data store — disk, network, remote memory — and a fast consumer. When the consumer asks for an item, the cache checks whether it has it (a hit) or needs to fetch it from the slow store (a miss). When the cache is full and a new item arrives, something must leave. The eviction policy is the algorithm.

The theory here has an elegant result: the optimal eviction policy, if you know the future access sequence, is to evict the item that will be accessed farthest in the future (Bélády's algorithm, 1966). This is provably optimal and completely impractical — you do not know the future. All real policies are approximations, and they differ in what they use as a proxy for "likely to be needed soon."

**LRU (Least Recently Used)** evicts the item accessed least recently. The proxy: recent access predicts future access — temporal locality. The implementation is a doubly-linked list plus a hash map, both updated in `O(1)` per access. LRU is the default and the right call for most workloads: it handles temporal locality well, it is `O(1)`, and its behavior is predictable.

**LFU (Least Frequently Used)** evicts the item with the lowest access count. The proxy: frequent past access predicts future access — frequency locality. LFU wins on workloads with stable popularity distributions — a CDN serving video assets where the top 10% of content accounts for 90% of requests and that distribution changes slowly. LFU loses when a burst of unique items floods the cache and inflates their counts, displacing legitimately popular items.

**FIFO (First-In, First-Out)** evicts the oldest item. No proxy about future access — just expiration by age. Simpler than LRU and worse on most workloads. The case for FIFO is implementation simplicity in constrained environments, or workloads where time-to-live matters more than access pattern.

**Random replacement** evicts a uniformly random item. Surprisingly competitive — within a few percent of LRU on many real workloads — because the variance in miss rates averages out at scale. It is also immune to adversarial access patterns that deliberately cycle through items in a pattern that defeats LRU. Some hardware caches use randomized replacement because the implementation cost is near zero and it resists the pathological inputs that degrade deterministic policies.

**ARC (Adaptive Replacement Cache)** maintains four lists: recently-once-accessed items (T1), recently-repeatedly-accessed items (T2), and two ghost lists of recently evicted items from each (B1, B2). When an evicted item in B1 is accessed, it signals that recency was more important; the cache grows T1 and shrinks T2. When an evicted item in B2 is accessed, it signals that frequency was more important; the cache grows T2 and shrinks T1. ARC self-tunes to the workload's recency/frequency balance without knowing the balance in advance. It consistently outperforms both LRU and LFU on mixed workloads. IBM held patents on ARC until around 2019; its descendants — CAR, CLOCK-Pro, W-TinyLFU (used in Caffeine, the default Java in-process cache) — are widely deployed in storage systems and application caches.

![ARC four-list diagram showing T1, T2, B1, B2](images/04-sorting-and-caching-fig-03.png)
*Figure 4.3 — ARC four-list diagram showing T1, T2, B1, B2*

---

## The cache-size interaction

Eviction policy is one knob. Cache size is another. The two interact non-linearly, and getting the size wrong makes the policy choice irrelevant.

The relationship between cache size and hit rate follows a characteristic shape: steep improvement up to a knee, then diminishing returns. A workload with a 5 GB hot working set may miss heavily at 1 GB, accept 80% of requests at 4 GB, and show almost no improvement from 8 GB to 16 GB. The knee is determined by the working set size, not by the cache size. If the cache is below the knee, more cache buys a lot. Above it, more cache buys almost nothing.

The practical implication: profile the working set size before tuning the policy. A well-tuned LRU on a correctly-sized cache outperforms a perfectly-tuned ARC on an undersized one. The hierarchy of interventions is: (1) identify the working set size; (2) size the cache to cover it; (3) tune the policy to the access pattern. Engineers who skip to step 3 without doing steps 1 and 2 are optimizing the wrong thing.

![Hit rate vs](images/04-sorting-and-caching-fig-04.png)
*Figure 4.4 — Hit rate vs*

---

## Failure modes: when "quicksort is always fastest" misleads

The misconception is common enough to name directly: quicksort is always fastest. It is not.

Quicksort has the best average-case constants among comparison sorts on random integer data fitting in RAM. That narrow statement is true. The general claim fails in four ways.

First, adversarial input drives quicksort to `O(n²)`. Already-sorted input with a fixed first-element pivot is the canonical case — the pivot always lands at the extreme, partitions are maximally unbalanced, and every level of recursion processes one fewer element than the last. A web service that accepts user-supplied lists and calls a naive quicksort is vulnerable to a denial-of-service attack. Randomize the pivot (Chapter 12) or use introsort.

Second, quicksort is not stable. A sort on a secondary key that uses quicksort destroys the primary-key ordering among equal secondary keys. Use merge sort or Timsort when stability is required.

Third, on data that fits the non-comparison sort criteria — integer keys in a bounded range — radix sort beats quicksort decisively. Sorting one billion 32-bit integers, radix sort runs in a small number of linear passes. Quicksort runs in `O(n log n)` with a worse memory access pattern. The asymptotic argument does not save it.

Fourth, on disk-resident data, quicksort's random access pattern is catastrophic. The right external-memory sort is multi-way merge, which keeps all its I/O sequential. Quicksort's pivoting across the array triggers random seeks on rotational storage and even on SSDs has worse throughput than streaming.

The corrective: state the constraint, identify the algorithm that survives the constraint, verify with a benchmark on representative data. The standard library's choice is right for the default case. Depart from it when you have a specific constraint the standard library was not optimized for, and when you have measured that the departure helps.

---

## Decision rules

These tables consolidate the chapter. They are for fast lookup after you have read the sections.

| Constraint | Sort choice |
|---|---|
| Subarray smaller than ~32 elements | Insertion sort |
| Stability required, memory available | Merge sort or Timsort |
| Memory tight, stability irrelevant | Quicksort with randomized pivot |
| Worst-case bound required, memory tight | Heapsort or introsort |
| Integer keys in a small known range | Counting sort or radix sort |
| Real-world partially-sorted data | Timsort |
| No special constraint | Standard library sort |

| Access pattern | Cache policy |
|---|---|
| Strong temporal locality | LRU |
| Stable popularity, long-tail distribution | LFU |
| Hardware-level, simplicity required | Random or FIFO |
| Mixed recency and frequency | ARC or W-TinyLFU |
| Unknown — start here, then profile | LRU as default |

---

## What this chapter does not enable

This chapter does not enable hardware-level cache tuning. CPU cache line optimization, prefetcher behavior, and NUMA-aware memory placement are architecture-specific and covered in vendor optimization manuals, not here. The chapter also does not cover external-memory sorting in depth — multi-way merge, B-tree-based sorts, parallel external sort. For that, Vitter's *Algorithms and Data Structures for External Memory* is the right reference.

---

## Capability statement

You can now match a sorting algorithm to a constraint — size, memory, distribution, stability, adversarial input — and explain why the match is right. You know why your standard library hybridizes the way it does and what each component of the hybrid is contributing. You can select a cache replacement policy from access-pattern signals, size the cache before tuning the policy, and recognize when a workload calls for something beyond the LRU default. When sorting or caching becomes the bottleneck, the diagnosis and the fix are in your hands.

---

## Exercises

**Warm-up**

1. Insertion sort has `O(n²)` worst-case complexity but `O(n)` best-case complexity. Describe the input that triggers each case, and explain why every production sort calls insertion sort on small subarrays despite the worst-case bound. *(Tests: understanding how best-case structure interacts with the small-array argument.)*

2. Merge sort and quicksort both run in `O(n log n)` on average. Name two constraints under which you would choose merge sort over quicksort, and two under which you would choose quicksort over merge sort. For each, explain what property of the algorithm makes it the right match. *(Tests: reading the decision table through the lens of the sections, not just the table itself.)*

3. The chapter says the `Ω(n log n)` lower bound applies to comparison sorts but not to counting sort or radix sort. What property of counting sort and radix sort allows them to escape the bound? Name a workload where counting sort wins and one where it loses despite the theoretically favorable bound. *(Tests: understanding the distinction between comparison-based and distribution-based sorting.)*

**Application**

4. You are building a service that sorts user-submitted lists of integers before storing them. A security researcher warns you that a naive sort could be exploited for a denial-of-service attack. Identify which sorting algorithm is vulnerable, describe the input that triggers the attack, and name two algorithmic fixes. *(Tests: applying the adversarial-input failure mode to a real production context.)*

5. A team is sorting timestamped log entries arriving roughly in order, with occasional out-of-order events from clock drift. They are currently using a clean merge sort implementation and are considering switching to Python's `list.sort()`. Explain what Timsort does differently on this specific workload, and predict whether the switch will improve or degrade performance. *(Tests: applying the natural-run detection argument to a concrete workload.)*

6. Use the decision tables to select an appropriate cache policy for each of the following workloads. For each, name the policy and explain which row of the table applies. (a) A social media feed renderer that caches the last 100 posts a user viewed. (b) A CDN where the top 5% of video assets account for 80% of traffic and that ratio is stable across weeks. (c) A CPU's L1 instruction cache, which must make eviction decisions in nanoseconds with no software overhead. *(Tests: translating access-pattern descriptions into policy selections using the decision framework.)*

**Synthesis**

7. The chapter argues that eviction policy and cache size interact non-linearly, and that engineers should tune size before policy. Construct a scenario in which a team tunes the policy without addressing size and gets a misleading result — a situation where their policy change appears to help when the underlying problem is sizing. Describe what measurement they would need to take to distinguish the two causes. *(Tests: understanding the cache-size/policy interaction as a diagnosis problem, not just a tuning problem.)*

8. Introsort, Timsort, and dual-pivot quicksort each hybridize a different set of algorithms for a different reason. For each of the three, identify the single constraint it is solving that its primary algorithm cannot handle alone, and explain which component of the hybrid addresses that constraint. *(Tests: reading hybridization as intentional design, not accidental complexity.)*

**Challenge**

9. Bélády's optimal algorithm evicts the item accessed farthest in the future. It is provably optimal and completely impractical. Design a restricted real-world scenario in which an approximation to Bélády's algorithm becomes practical — a situation where partial knowledge of future accesses is available — and describe what eviction policy you would derive from that knowledge. Identify the failure mode your policy would inherit from the limitations of that partial knowledge. *(Tests: reasoning about the theory-practice gap in cache policy design; applying the structure of the chapter's theoretical framing to a novel case.)*

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
