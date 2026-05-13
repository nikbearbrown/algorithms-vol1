# Research notes — Chapter 03: Data Structures

## Source folders used
- Primary: `Data_Structures/Data_Structures.md`
- Hash-table content: routed from `Algorithm_Analysis/` per source-chapter-map.md

## Source sections kept
- `## Introduction to Data Structures` (3–98) — classification and applications
- `## Amortized Analysis` (99–220) — referenced; full case study cross-referenced to Ch 2
- `## Heaps` (221–487) — binary, binomial, Fibonacci heaps
- `## Union-Find` (488–687) — with path compression and union by rank
- `## Balanced Search Trees` (688–1074) — BSTs, red-black, AVL, B-trees (B-tree note added beyond source)
- `## Hash Tables` (1075–1184) — operations, collisions, universal hashing
- `## Bloom Filters` (1185–1292) — basics, formula for false positive rate
- `## Conclusion → Choosing the Right Data Structure` (1422–1489) — informed decision rules

## Source sections cut
- `## Exercises and Problems` (1490–1599)
- `## Further Reading and Resources` (1600+)

## Original Claude content (NOT or only partially in source)

### Count-min sketch (§7)
Source mentions count-min sketch only in the caching chapter source (Sorting_Caching.md). I included it in Ch 3's probabilistic-structures section as a streaming companion to bloom filter and HyperLogLog. The basic description is accurate but the inclusion-decision was Claude's. Routed appropriately for chapter 3's "specialized structures" framing.

### HyperLogLog (§7 + worked example)
Not present in source as a named structure. Added because count-distinct is a canonical real-world need and the worked example calls for it. [verify] specific implementations cited (Redis, BigQuery, Presto).

### B-tree disk-storage paragraph
Source mentions BSTs and red-black trees but does not specifically call out B-trees for on-disk storage. Added because production storage choice cannot be discussed without B-trees. [verify] specific database internals (PostgreSQL, MySQL InnoDB).

### Worked example (§8) — real-time analytics
Source's worked example for data structures was generic. Per outline.md, this chapter's worked example is "real-time analytics system: count-distinct queries, top-k tracking, range queries on timestamped events." Built fresh. The 100K events/second figure marked [verify].

### Failure modes (§9)
Source's "limitations" sections are scattered and brief. The four failure modes engaging "use a hash map by default" are framed by Claude. The phenomena (range queries, cache pressure, adversarial input, low-load memory overhead) are widely-canonical.

### Algorithmic complexity attack
Cited Crosby and Wallach 2003. Source mentions universal hashing but does not name the attack. Date marked [verify].

### Fibonacci heap practical caveat
Source describes Fibonacci heaps positively. Practical caveat about constants making them rarely used in production added by Claude. [verify].

## Factual claims preserved from source
- Heap definitions and shape/heap properties — direct from source
- Binary heap operations and complexity — from source
- Fibonacci heap properties — from source
- Union-Find operations and rank/path compression — from source
- Hash table collision strategies — from source
- Universal hashing construction — from source verbatim
- Bloom filter formula — from source verbatim

## [verify] count
8 inline `[verify]` markers:
1. Fibonacci heap implementation comparisons on real graphs
2. Linux kernel CFS (red-black tree)
3. Database B-tree internals
4. Crosby/Wallach algorithmic complexity attack date
5. Python dict insertion-ordered since 3.7
6. Union-Find inverse Ackermann attribution (Tarjan 1975)
7. HyperLogLog implementations in Redis, BigQuery, Presto
8. 100K events/second in worked example
9. HyperLogLog 12 KB per event type / ≤2% error

(Counted 9 above. Final count: 9.)

## Structure-drift checks
- Section 1 titled "When the choice of data structure matters" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: real-time analytics (named in outline.md) ✓
- Failure modes engages "Use a hash map by default" misconception ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- The chapter uses more inline lists than typical reference prose because workload signatures and operation tables benefit from them. Held under control: lists earn their place via decision-rule and signature-matching purposes.
- Resisted "implementation tutorial" drift in heaps and trees sections. Outline note explicit: "Resist the temptation to teach implementation. This is a reference for choice, not a textbook for building structures from scratch." The heap section in particular drifted toward describing heapify-down step-by-step on first pass; pulled back to the operation+complexity framing.
- Bloom filter section had a temptation to derive the false-positive formula on the page. Reference convention: state the formula, link to derivation. Held.

## Word count
~3,030 words (target: ~3,000)

## Open issues for editor
- HyperLogLog claims (size, error rate, Redis/BigQuery use) need cross-check
- B-tree database internals citations need verification
- Crosby/Wallach 2003 paper citation could be made explicit if desired (currently bare reference)
- Decision rules table has 10 rows; consider whether to consolidate or split
