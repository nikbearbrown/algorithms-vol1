# Research notes — Chapter 04: Sorting and Caching

## Source folders used
- Primary: `Sorting_and_Caching/Sorting_Caching.md`
- Secondary: `Algorithm_Analysis/Algorithm_Analysis.md` lines 839–1139 (sorting algorithms section, routed in)

## Source sections kept
From `Sorting_Caching.md`:
- `## Introduction to Caching` (3–82) — definitions, role, benefits, challenges
- `## Theoretical Foundations of Caching` (83–168) — coherence, locality, eviction policies overview
- `## Cache Eviction Strategies` (313–445) — LRU, FIFO, Random, ARC
- `## Caching in Distributed Systems → CDNs` (500–599)

From `Algorithm_Analysis.md`:
- Sorting algorithms section (839–1139): bubble, selection, insertion, merge, quick, heap

## Source sections cut entirely
- `## Caching Algorithms and Data Structures` (600–720) — bloom filter and count-min coverage routed to Ch 3
- `## Emerging Trends → Machine Learning for Cache Management` (792–823) — out of scope (magazine)
- `## Exercises and Problems`
- `## Further Reading and Resources`
- `## Future Directions in Caching Technology`

## Original Claude content (NOT in source)

### Hybrid sort details (Timsort, Introsort, Dual-Pivot)
Source describes the basic sort algorithms but does not address production hybrid sorts. Per outline.md, Timsort is the chapter's signature; introsort and Dual-Pivot Quicksort were added for completeness. All specific dates and thresholds marked [verify]:
- Timsort in CPython 2.3 (2002)
- Timsort in JDK 7 (2011)
- MIN_RUN typically 32–64
- Introsort recursion-depth switch at 2 log₂ n
- Introsort insertion-sort threshold = 16 (libstdc++)
- Dual-Pivot Quicksort INSERTION_SORT_THRESHOLD = 47

### Worked example case A — Timsort on log data
Specific percentages (12% → 2%) marked [verify]. The phenomenon (Timsort's natural-run detection winning on partially-sorted data) is widely documented.

### Worked example case B — CDN cache replacement
Specific 1 TB / 5 GB working set numbers marked [verify]. The 10–30% hit-rate gain claim aligns with published CDN literature but the specific range is approximate.

### ARC patent expiration
"Around 2019" marked [verify]; IBM's ARC patents are real (US 6,996,676 and related) but the precise expiration year should be verified.

### ZFS uses ARC
[verify] specific filesystem/database adoption claims.

## Factual claims preserved from source
- Sort algorithm pseudo-code and complexity bounds — from source
- LRU, FIFO, ARC descriptions — from source
- Cache coherence and consistency definitions — from source
- Temporal and spatial locality definitions — from source

## [verify] count
12 inline `[verify]` markers:
1. Timsort in CPython 2.3 (2002)
2. Timsort in JDK 7 (2011)
3. MIN_RUN 32–64 range
4. Introsort threshold = 16
5. Dual-Pivot in JDK 7 (2011)
6. Dual-Pivot threshold = 47
7. ARC patent expiration ~2019
8. ZFS uses ARC
9. Worked example A: 12% → 2% percentages
10. Worked example A: MIN_RUN 32–64 (second mention)
11. Worked example B: 1 TB / 5 GB working set numbers
12. Worked example B: 10–30% hit-rate range

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules (two tables: sort + cache) ✓
- Worked example: Timsort on Python data + CDN cache (named in outline.md) ✓
- Failure modes engages "Quicksort is always fastest" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- Combining sorting and caching in one chapter creates a structural risk: the chapter could fall into two halves with no through-line. The opening connects them by the memory-hierarchy theme. Held.
- Resisted the temptation to provide full pseudo-code for each sort. Reference convention: name the algorithm, name the bound, name the use case. Implementation goes to the companion page.
- The ARC discussion drifted toward patent-history narrative on first pass; pulled back to one parenthetical.

## Word count
~2,840 words (target: ~2,800; within +1.5%)

## Open issues for editor
- All [verify] tags need cross-check, especially production-sort dates and thresholds
- Worked example numbers are illustrative — could be replaced with figures from actual published benchmarks if available
- ARC patent claim should be confirmed before publication
- The two-table format for decision rules (one for sort, one for cache) is ergonomic but unusual; check that the layout works
