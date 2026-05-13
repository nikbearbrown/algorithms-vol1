# Research notes — Chapter 02: Algorithm Analysis

## Source folders used
- Primary: `Algorithm_Analysis/Algorithm_Analysis.md`
- Secondary: book.md (chapter notes), outline.md (capabilities, worked example, misconception)

## Source sections kept
- Lines 3–321: Introduction to Algorithm Analysis (definitions, complexity measures, run-time evaluation)
- Lines 322–565: Big O, Omega, Theta notations + comparison
- Lines 566–755: Asymptotic analysis, recurrence relations, master theorem, amortized analysis
- Lines 756–838: Binary search as a brief Big O example (compressed; not used at length)

## Source sections routed out
Per source-chapter-map.md:
- Sorting algorithms → Ch 4
- Graph algorithms → Ch 5
- Greedy algorithms → Ch 6
- Divide and conquer → Ch 7
- Dynamic programming → Ch 8
- Approximation → Ch 11
- Randomized → Ch 12
- Parallel algorithms → cut (out of scope)

## Source sections cut entirely
- End of Chapter Exercises (lines 2256–2487)
- Chapter Summary (lines 2488+)
- Future Trends in Algorithm Analysis (speculative)
- Further Reading and Resources (companion page handles)

## Original Claude content (NOT in source)

### Section 6 — Cache effects
This section is original to the chapter; the source does not address memory-hierarchy effects on asymptotic bounds at all. Per outline.md, this section is the chapter's differentiation from CLRS and is the load-bearing original content of the chapter.

Specific claims marked [verify]:
- Memory hierarchy latency numbers (L1 ~1 ns, L2 ~4 ns, L3 ~12–40 ns, RAM ~100 ns, SSD μs, HDD ms) — these are widely-canonical but vary by architecture. Should be verified against a reference like Hennessy & Patterson or current Intel/AMD optimization manuals at publication time.
- 100× latency ratio L1 to RAM — canonical, but specific multiplier varies.
- Insertion sort wins to roughly n=32 to n=64 — canonical, but the exact crossover depends on implementation and hardware.

### Worked example — insertion sort vs merge sort crossover
- 32 as run threshold for CPython's Timsort: [verify] against current CPython source (`Objects/listobject.c`, `MIN_RUN` calculation; canonical reference is Tim Peters' original notes).
- INSERTION_SORT_THRESHOLD = 47 in JDK's Dual-Pivot Quicksort: [verify] against current OpenJDK source (`java.util.DualPivotQuicksort`).
- Introsort threshold = 16 in libstdc++: [verify] against current libstdc++ `bits/stl_algo.h`.

### Algorithmic complexity attack on hash tables
- Cited as a real failure mode. Klink and Wälde 2003 paper, Crosby and Wallach 2003 USENIX paper. [verify] specific dates if cited explicitly; in this draft only the general phenomenon is cited.

## Factual claims preserved from source
- Definitions of Big O, Omega, Theta — straight from source
- Master theorem statement and three cases — from source, lightly compressed
- Recursion tree method as alternative — from source
- Substitution method as alternative — from source
- Amortized analysis dynamic array example — from source

## [verify] count
6 inline `[verify]` markers:
1. L1/L2/L3/RAM latency numbers
2. CPython Timsort MIN_RUN cutoff
3. JDK INSERTION_SORT_THRESHOLD = 47
4. libstdc++ introsort threshold = 16
5. JDK threshold (second mention)
6. libstdc++ threshold (second mention)

## Voice-anchoring notes
- The chapter is the most-cited in the book and the longest in this batch. Maintaining reference voice over 3,200 words required vigilance — three places in the cache-effects section drifted toward "let me explain why this matters" textbook register; pulled back.
- The §6 cache-effects section is original and intellectually the chapter's differentiator. Resisted the temptation to lecture. Voice held: direct, mechanism-first, concrete numbers.
- The worked example section uses real production cutoffs (CPython, JDK, libstdc++) rather than a pedagogical sketch. Per outline.md instruction.
- Failure modes section engages the misconception "O(n log n) is always good enough" by working through three concrete cases (cache, constants, framing).
- Decision rules table fits scan-and-go reference shape.

## Structure-drift checks
- Section 1 titled "When asymptotic analysis applies" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules section present ✓
- Worked example uses the named real-practitioner case (insertion vs merge crossover at n≈32) ✓
- Failure modes section engages the named misconception ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Word count
~3,250 words (target: ~3,200; within +2%)

## Open issues for editor
- All [verify] tags need cross-check against current sources (CPython, JDK, libstdc++)
- Consider whether to add a small ASCII-art figure showing the merge sort recursion tree (currently embedded in prose)
- The "Algorithmic complexity attack" reference in §10 could carry a citation; left bare to avoid speculation about which paper to cite without verification
- Master theorem reference card is a major companion-page deliverable; flag for design pass
