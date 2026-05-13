# Chapter 3 — Data Structures

*The question is never "does this work." The question is "what does this make cheap."*

---

Here is a puzzle I want you to sit with before we do anything else.

Suppose you have a billion web URLs — pages a crawler has already visited — and for every new URL the crawler discovers, you need to answer one question: *have we been here before?* That's it. Just yes or no. You need the answer fast, billions of times, and you cannot afford to store the full URL list in memory because you don't have that much memory.

A hash table answers the question in expected constant time. It also costs you memory proportional to the number of URLs stored, which at a billion entries starts to hurt.

A bloom filter answers the question in constant time with a fraction of the memory. Occasionally it says "yes, we've been here" when the honest answer is "no, actually." It never says "no" when the answer is "yes."

Which structure do you reach for?

If your instinct was "hash table, obviously," I want to examine that instinct. Not because it's wrong — for most problems, it's right. But because the moment you said "obviously," you stopped asking the question that actually matters: *what operations do I need, and what does each structure make cheap?*

That question is what this chapter is about.

---

## The idea that holds everything else together

<!-- → INFOGRAPHIC: radial "operation cost map" — each data structure as a node, labeled operations radiating from it, colored by O(1)/O(log n)/O(n); makes the "every structure optimizes for a mix" claim visually scannable at a glance -->

A data structure is a deal. You give it data. It gives you fast access to some operations and slow access to others. Every structure optimizes for a particular mix of operations, and there is no structure that's fast at everything. If there were, we'd have one data structure and call it a day.

The art of choosing data structures is the art of reading a workload — listing the operations you actually need, their relative frequencies, and what "fast" has to mean — and then matching that profile to the structure whose cheap operations line up with yours.

This sounds obvious when stated. It fails constantly in practice, because we have defaults. The default is the hash map. The hash map is a fine default. It is not a universal default. I'll show you exactly where it breaks, and why.

---

## Arrays: the structure that wins on physics

Start with arrays, because everything else is built in reaction to them.

A static array is a contiguous block of memory. You ask for element `i`, you get it in one operation: start address plus `i` times element size. The CPU doesn't deliberate. It reaches in and takes it. That's `O(1)` access by index, and the constant factor is as small as it gets.

The catch: the array is fixed size. You allocated ten slots; you cannot have eleven without allocating a new block and copying everything over.

Dynamic arrays — Python's `list`, Java's `ArrayList`, C++'s `std::vector` — solve this by doubling when full. You insert an eleventh element, the structure notices it's out of room, allocates a block of twenty, copies the ten elements over, and continues. That single insert cost `O(n)`. The next nine inserts cost `O(1)` each. Then you double again.

The key insight — and this is worth pausing on — is that the expensive inserts happen rarely enough that they don't ruin the average. The total cost of `n` appends is `O(n)`, because each element gets copied at most once per doubling, and doublings are geometrically spaced. Spread that total cost across `n` operations and you get `O(1)` amortized per append.

Amortized doesn't mean each operation is fast. It means the expensive operations are rare enough that they don't change the average. When you need a hard real-time guarantee — every single operation must complete in bounded time — amortized analysis tells you nothing useful. When you care about throughput, it tells you everything.

Now here is something worth understanding about why dynamic arrays dominate linked lists in almost every production setting. A linked list also supports `O(1)` append. It also supports `O(1)` insert at a known position. By the asymptotic analysis alone, they're tied.

They are not tied.

Dynamic array elements sit in contiguous memory. When you scan them, your CPU prefetches the next element while you're still processing the current one. Cache miss rate is low. Linked list elements are scattered across the heap, allocated wherever `malloc` found space. Traversing a linked list means following a pointer from one arbitrary memory location to another, which is almost guaranteed to be a cache miss. The constant factor swamps the theoretical tie.

<!-- → IMAGE: side-by-side memory layout diagram — top row shows dynamic array as a contiguous colored strip, bottom row shows linked list as boxes with pointer arrows leaping to random addresses; caption should note that the pointer jumps are the cache misses -->

Linked lists earn their place at one workload: arbitrary-position insertion and deletion when you already have a pointer to the node you're modifying. Nowhere else does a linked list outperform an array in practice.

---

## Heaps: the structure for "give me the best one"

Suppose the only operations you need are: insert an item with a priority, and always give me the minimum-priority item.

This comes up constantly. A task scheduler runs the highest-priority job next. Dijkstra's shortest-path algorithm always explores the closest unvisited node next. A hospital triage system always treats the most critical patient next.

The heap is built for exactly this.

Picture a binary tree that's completely filled level by level, left to right. Every node satisfies one rule: it's smaller than both its children. So the smallest element is always at the root.

Insert: put the new element at the next open leaf position, then "sift it up" — swap it with its parent until the heap property is restored. This takes at most `O(log n)` steps, because the tree has height `log n`.

Extract-min: take the root (which is the minimum), move the last leaf to the root, then "sift it down" — swap it with its smaller child until the heap property is restored. Also `O(log n)`.

The clever part is that you can store this tree in an array. The root is at index 0. Its children are at indices 1 and 2. The children of index `i` are at `2i+1` and `2i+2`. You never need pointer arithmetic or dynamic allocation. The array layout also gives you the same cache benefits as a plain array.

<!-- → IMAGE: dual-panel diagram — left panel shows the heap as a labeled binary tree (nodes numbered 0–6), right panel shows the same heap as a flat array with index annotations and arrows connecting tree positions to array slots; student should see the arithmetic relationship at a glance -->

One counterintuitive result: if you have an unordered array and want to turn it into a heap, the naive approach — insert each element one at a time — costs `O(n log n)`. But there's a smarter approach: sift down from every internal node, starting at the bottom. This costs `O(n)`. The proof is a geometric series: most nodes are near the bottom, where sifting down costs almost nothing. The savings add up.

This `O(n)` construction is what makes heap sort possible: heapify the array in `O(n)`, then repeatedly extract the minimum in `O(log n)` each, for a total of `O(n log n)`. Same asymptotic cost as merge sort, but fully in-place.

A binary heap handles the standard priority-queue workload. When you also need to merge two independent priority queues, a binomial heap supports `O(log n)` merge. When you need to decrease the priority of an existing element — which Dijkstra's algorithm does — a Fibonacci heap supports `O(1)` amortized decrease-key. The theoretical improvement to Dijkstra's running time is real. The constants on a Fibonacci heap are large enough that in practice, a binary heap is usually competitive or better on actual hardware. Theory and implementation sometimes disagree; measure before you trust the bound.

---

## Balanced search trees: the structure for ordering

A binary search tree maintains a simple invariant: every node's key is larger than all keys in its left subtree and smaller than all keys in its right subtree. Lookup, insert, and delete all follow the same path from root to leaf, taking `O(h)` time where `h` is the tree's height.

The problem: `h` depends on insert order. Feed a sorted sequence to a naive binary search tree and it degenerates into a linked list — every node hangs to the right, `h = n`, every operation costs `O(n)`. The elegant invariant fails to protect itself from adversarial input.

Self-balancing trees add rebalancing operations — rotations and color flips — that enforce `h = O(log n)` no matter what order you insert. The result: guaranteed `O(log n)` for all three operations.

Red-black trees are the dominant implementation. Each node is colored red or black; a set of color invariants implies the tree height stays within a factor of 2 of optimal. The C++ standard library's `std::map` and `std::set` are red-black trees. Java's `TreeMap` and `TreeSet` are red-black trees. The Linux kernel's process scheduler uses a red-black tree to track runnable processes. The structure has earned its place.

<!-- → IMAGE: two-panel tree diagram — left panel shows a degenerate BST built from sorted input (a right-leaning chain of height n), right panel shows the same keys in a balanced red-black tree of height ~log n; caption should make the height difference the focus, not the coloring -->

AVL trees maintain a tighter balance — height difference between any node's subtrees is at most 1. Lookups are marginally faster; inserts do more work to maintain the tighter invariant. The practical difference is small and workload-dependent.

B-trees change the question entirely. Instead of binary branching (each node has at most 2 children), B-tree nodes hold many keys and many children. A B-tree node with `t` as the minimum degree holds between `t-1` and `2t-1` keys and between `t` and `2t` children. This makes the tree very shallow. Lookup, insert, and delete still cost `O(\log n)$, but `log` base `t` rather than base 2 — far fewer levels.

Why does this matter? Because on disk, reading a node means fetching a page. You want as few page fetches as possible. A shallow tree means few levels means few page fetches. Databases and filesystems use B-trees and B+ trees for exactly this reason — PostgreSQL, MySQL's InnoDB storage engine, the ext4 and NTFS filesystems. These are not textbook exercises. They are the structures holding your data right now.

The discriminator between hash tables and balanced trees is one capability the hash table lacks entirely: **range queries**. "Give me all keys between `a` and `b`" requires an ordered structure. A balanced tree answers this in `O(\log n + k)` where `k` is the number of results. A hash table cannot answer it at all without scanning every element.

---

## Hash tables: the structure for lookup

A hash table maps a key to a value through a hash function. The hash function transforms the key into an index in an array (the bucket array). The value lives at that index. Lookup: hash the key, go to the index, retrieve the value. Expected `O(1)`.

Two elements can hash to the same index — a collision. **Separate chaining** handles collisions by storing a linked list at each bucket; colliding elements join the list. **Open addressing** stores everything in the bucket array itself; collisions probe forward to the next open slot via some probe sequence.

Open addressing has better cache behavior — all lookups stay within the contiguous bucket array. Separate chaining handles high load factors (fraction of buckets occupied) more gracefully; it doesn't degrade sharply as the table fills. Most practical implementations use open addressing with a load factor threshold — resize (double the array, rehash everything) when load factor exceeds roughly 0.7.

The `O(1)` bound is *expected* under the assumption that keys hash uniformly across buckets. That assumption can fail. If an attacker knows your hash function, they can craft keys that all hash to the same bucket, driving every operation to `O(n)` — a realistic attack that took down web servers in the early 2000s. Modern language runtimes defend against this by seeding hash functions with a per-process random value; the attacker doesn't know the seed and can't craft the collision pattern. Check whether your environment provides this protection; configurations exist where it doesn't.

**Universal hashing** is the principled defense: pick a hash function at random from a family where no adversarial input can force systematic collisions. The family $h_{a,b}(k) = ((ak + b) \bmod p) \bmod m$ for random $a$ and $b$ and a prime $p$ larger than the key universe is the canonical construction. Each key independently has a $1/m$ probability of landing in any bucket, regardless of what other keys are present.

The hash table's limits: no range queries, no ordered iteration (unless the implementation explicitly maintains insertion order, as Python's `dict` has done since version 3.7), and a cache penalty when the bucket array grows beyond last-level cache. A table with ten million entries is unlikely to fit in L3; random-access lookups will miss cache and pay the full RAM latency penalty. At that scale, a sorted array with binary search — which has worse asymptotic cost but contiguous access — often wins on wall-clock time.

---

## Probabilistic structures: approximate is enough

The bloom filter deserves its own discussion because it illustrates a principle that matters more as systems scale: sometimes you don't need an exact answer, and accepting a bounded approximation buys you an order-of-magnitude resource reduction.

A bloom filter uses a bit array of size `m` and `k` independent hash functions. To insert an element: hash it `k` times, set those `k` bits to 1. To test membership: hash it `k` times, check those `k` bits. If any bit is 0, the element is definitely absent. If all bits are 1, the element is *probably* present — some earlier element might have set all those same bits by coincidence.

False negatives are impossible. A false positive means the filter claims an element is present when it isn't. The false positive rate is:

$$p \approx \left(1 - e^{-kn/m}\right)^k$$

for `n` elements inserted. You can tune this. More bits per element, lower false positive rate. The optimal number of hash functions is $k = \frac{m}{n} \ln 2$. For 1% false positive rate, you need roughly 9.6 bits per element — orders of magnitude less than storing the elements themselves.

<!-- → CHART: line chart with false positive rate (y-axis, log scale 0.001%–10%) vs. bits-per-element (x-axis, 4–20), one line per k value (k=1,2,3,5,7,10); student should see that the optimal k traces a curve and that going below ~8 bits per element makes 1% FPR impossible at any k -->

The crawler problem from the opening of this chapter is exactly where a bloom filter belongs. You have a billion URLs. Each URL might be 50-100 bytes. Storing them in a hash set costs 50-100 GB. A bloom filter at 1% false positive rate costs roughly 1.2 GB. What does a 1% false positive rate cost you? The crawler re-visits 1% of URLs it should have skipped. Almost certainly acceptable.

Two related structures worth knowing: **HyperLogLog** estimates the count of distinct items in a stream using kilobytes of memory regardless of stream size, with bounded relative error — used by Redis and most analytics platforms for `COUNT DISTINCT` queries on billions of items. **Count-min sketch** estimates the frequency of each item in a stream using a two-dimensional array of counters and `d` independent hash functions; the estimate is always an overcount, but bounded. Both structures accept bounded error in exchange for constant memory. Use them when memory is the constraint and the error bound is acceptable to your application.

---

## Union-Find: the structure for connectivity

Suppose you have a set of items, and operations arrive that say "these two things are in the same group." After many such merges, you want to answer: "are these two things in the same group?" This is the connected-components problem.

Union-Find (also called Disjoint Set Union) solves it. Each item belongs to a set, represented by a tree rooted at a representative element. `Find(x)` returns the representative of `x`'s set; `Union(x, y)` merges `x`'s set and `y`'s set.

Naive implementation: every element points to its parent, root points to itself. `Find` climbs to the root. `Union` makes one root point to the other. Cost per operation: `O(n)` in the worst case if the tree becomes a chain.

Two optimizations together bring this to nearly constant time.

**Union by rank**: when merging two trees, attach the shorter tree's root under the taller tree's root. This keeps trees shallow.

**Path compression**: when `Find` climbs from `x` to the root, make every node on that path a direct child of the root. Future `Find` calls on any of those nodes go directly to the root.

<!-- → IMAGE: before/after diagram of path compression — left shows a chain of five nodes (deep tree), right shows the same nodes all pointing directly to root after one Find call; caption should note that the restructuring happens as a side effect of the lookup, not as a separate pass -->

With both optimizations, the amortized cost per operation is $O(\alpha(n))$ where $\alpha$ is the inverse Ackermann function. $\alpha(n)$ grows so slowly that it never exceeds 4 for any `n` smaller than the number of atoms in the observable universe. For practical purposes, every operation is constant time.

Kruskal's minimum spanning tree algorithm uses Union-Find to efficiently check whether adding an edge would create a cycle: if both endpoints are already in the same component, skip the edge. Dynamic graph connectivity queries — "are these two nodes currently connected, given that edges are being added?" — are a natural fit.

---

## The decision table

State the operations you need. State the working-set size relative to cache. State whether range queries are required. The structure falls out.

<!-- → TABLE: decision table as a formatted reference card — three columns: "Workload signature", "Recommended structure", "Key reason" (the one-phrase justification for the choice); the "Key reason" column is what's missing from the prose table and makes this a self-contained reference -->

| Workload signature | Recommended structure |
|---|---|
| Index by position, append-heavy | Dynamic array |
| Lookup by key, no ordering, no range queries | Hash table |
| Lookup, insert, delete, ordered iteration, range queries | Balanced BST |
| On-disk storage, page-level access | B-tree / B+ tree |
| Insert + extract-min only | Binary heap |
| Membership test, large set, false positives acceptable | Bloom filter |
| Count distinct over a stream | HyperLogLog |
| Frequency estimate over a stream | Count-min sketch |
| Connected components, dynamic graph | Union-Find with path compression + union by rank |

The hash map is not on every row. This is intentional.

---

## The default trap

The misconception this chapter engages is specific: *a hash map's `O(1)` lookup makes it the right default for any keyed access.*

Three places this breaks.

**Range queries.** A hash map cannot answer "all keys between `a` and `b`" in sub-linear time. The keys have no order. If you need range queries — timestamps, numeric IDs, lexical prefixes — you need a tree. Using a hash map forces a full scan every time.

**Cache pressure at scale.** At ten million entries, a hash map's bucket array is unlikely to fit in L3. Random-access lookups miss cache, fetch from RAM, pay 100× the latency of a cache hit. A sorted array over the same data, accessed by binary search, has asymptotically worse lookup — `O(\log n)` versus `O(1)` — but each access lands in contiguous memory, prefetches well, and often wins on actual wall-clock time. Asymptotic bounds describe behavior as `n` grows. Hardware is fixed. When the structure is sized larger than your last-level cache and access is random, measure both before deciding.

**Adversarial input.** In any setting where user-supplied data keys the hash table, the `O(1)` expected bound depends on the attacker not knowing your hash function. If they do, they construct inputs that all hash to one bucket, every operation becomes `O(n)`, and you have a denial-of-service vulnerability. Keyed hash functions and universal hashing are the defense.

**Memory overhead at low load factor.** A hash table at 50% load factor uses memory for twice as many slots as elements. A balanced tree uses memory for exactly the nodes it holds, plus pointer overhead. Neither is universally cheaper; the comparison depends on element size and load factor. The hash table's memory cost is not zero.

The corrective is to ask the question before reaching for the default: what operations do I actually need, and what does each structure make cheap? The hash map answers that question correctly most of the time. The habit of asking is what prevents the cases where it doesn't.

---

## What you can do now that you couldn't before

You can read a workload description — its operations, their frequencies, its scale relative to cache — and name the structure or combination of structures that makes those operations cheap. You can explain why the hash map is a fine default and precisely where it fails. You can describe the trade-off a bloom filter makes and the workloads where that trade-off is correct. You can implement union-find with both optimizations and explain why the amortized cost is effectively constant.

The next chapter applies several of these structures in the context of sorting: the heap earns its place in heap sort, comparison-based lower bounds explain why certain structures can't be sorted in sub-linear time, and the relationship between a sorted array and a balanced tree becomes concrete.

---

## Exercises

### Warm-up

**1.** A dynamic array currently holds 8 elements in a capacity-8 backing array. You append 3 more elements, one at a time, using geometric (×2) doubling. Trace each append: for each one, state whether a resize occurs, the capacity before and after, and the cost of that single append. Then state the total number of element-copy operations across all three appends.
*(Tests: amortized analysis of dynamic array resizing.)*

**2.** Draw the min-heap that results from inserting the keys 9, 3, 7, 1, 5 into an initially empty heap, in that order. Show the heap after each insertion. Then perform one `extract-min` and show the resulting heap.
*(Tests: heap insert and sift-up, extract-min and sift-down.)*

**3.** A hash table uses open addressing with linear probing and a load factor threshold of 0.7. The table currently has capacity 10 and holds 6 elements. You insert 2 more elements. Does a resize trigger? If so, what is the new capacity? State your reasoning.
*(Tests: load factor mechanics and resize threshold.)*

---

### Application

**4.** For each workload below, name the data structure you would reach for first and give one sentence explaining the decisive operation that rules out the hash map:

   - (a) A log-aggregation service that needs to answer "how many distinct IP addresses have we seen in the last hour?" over a stream of 500 million log entries per hour, with 2% relative error acceptable.
   - (b) A flight-booking system that must efficiently retrieve all flights departing between 8:00 AM and 11:00 AM on a given date.
   - (c) A spell-checker that needs to answer "is this word in the dictionary?" for a dictionary of 200,000 words, with a 0.1% false-positive rate acceptable and minimal memory use required.

*(Tests: workload-to-structure matching for HyperLogLog, balanced BST range queries, and bloom filter.)*

**5.** You are implementing Kruskal's minimum spanning tree algorithm on a graph with 10,000 nodes and 50,000 edges. The edges are sorted by weight. You process them in order, adding each edge to the spanning tree if its two endpoints are in different components.

   - (a) Which data structure do you use to track connected components?
   - (b) What are the two optimizations required for near-constant amortized cost, and what failure does each one prevent?
   - (c) Without those optimizations, what is the worst-case cost of processing all 50,000 edges?

*(Tests: Union-Find application and the necessity of both optimizations.)*

**6.** A hash table with separate chaining holds `n` elements across `m` buckets, with load factor $\alpha = n/m$. An adversary knows your hash function and has crafted `n` keys that all map to bucket 0. What is the cost of a single lookup in this table? What defense does universal hashing provide, and why can the adversary not reproduce the attack after you deploy it?
*(Tests: hash table collision analysis and the universal hashing defense.)*

---

### Synthesis

**7.** A real-time leaderboard for an online game must support three operations: (1) update a player's score, (2) retrieve the top-10 players at any time, (3) retrieve a player's current rank (their position in the sorted order). A classmate proposes using a single max-heap. Identify the operation this design handles well and the two operations where it fails or becomes expensive. Then propose a structure or combination of structures that handles all three efficiently, and justify each choice.
*(Tests: recognizing heap limitations; multi-structure design reasoning.)*

**8.** You are designing storage for a URL shortener service (think `bit.ly`). The service has two access patterns: (A) given a short code like `xK3p`, retrieve the full URL — happens 100,000 times per second; (B) given a full URL, check whether we have already shortened it — happens 500 times per second, and it is acceptable to occasionally shorten the same URL twice. The full URL corpus is 2 billion entries.

   Choose a data structure for each access pattern. For pattern (A), justify why a hash table is correct. For pattern (B), justify why a bloom filter is correct and calculate the approximate memory cost at a 1% false positive rate (assume average URL length of 80 bytes, and that the bloom filter requires ~9.6 bits per element).

*(Tests: applying the hash table vs. bloom filter trade-off in a concrete system design scenario.)*

---

### Challenge

**9.** The inverse Ackermann function $\alpha(n)$ is described in this chapter as "effectively constant for any `n` you will encounter." Explain what this claim means precisely — and what it does *not* mean. Specifically: is the amortized cost of Union-Find operations `O(1)`? If not, why do we say it is "effectively" constant? What would it mean for $\alpha(n)$ to exceed 5 for an input you could actually construct?
*(Tests: precise understanding of asymptotic claims vs. practical guarantees; distinguishes "effectively constant" from "actually constant.")*

**10.** Design a data structure for the following workload: a stream of (user\_id, event\_type) pairs arrives at 1 million events per second. At any moment, you must be able to answer: (a) approximately how many distinct users have triggered event type X in the last 5 minutes? and (b) which event type has the highest estimated count in the last 5 minutes? Memory is constrained to 10 MB total. Exact answers are not required; bounded error is acceptable.

   Describe the structures you would use for each query, how you would handle the sliding 5-minute window, and what error guarantees your design provides.

*(Tests: combining probabilistic structures — HyperLogLog for count-distinct, count-min sketch or similar for frequency — with sliding window mechanics; open-ended design with no single correct answer.)*

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
