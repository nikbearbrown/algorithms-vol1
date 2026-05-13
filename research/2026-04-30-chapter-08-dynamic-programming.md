# Research notes — Chapter 08: Dynamic Programming

## Source folders used
- Primary: `Dynamic_Programming/Dynamic_Programming.md`
- Secondary: `Dynamic_Programming/Bellman_Ford.md` (briefly cross-referenced; full coverage in Ch 5)
- Tertiary: `Algorithm_Analysis/Algorithm_Analysis.md` DP section (lines 1431–1507), routed in

## Source sections kept
- `## Introduction to DP` (3–341) — principles, history, basic techniques, memoization vs tabulation
- `## The Knapsack Problem` (342–418)
- `## Sequence Alignment` (419–576) — informs edit-distance worked example
- `## Optimal Binary Search Trees` (577–674)
- `## Shortest Paths in Graphs → BF, FW` (675–750) — DP framing, cross-ref Ch 5
- `## DP for NP-Complete → Faster Exact` (751–841) — Held-Karp TSP
- `## Advanced DP → State Space Reduction` (948–1060)
- `## Comparative Analysis → DP vs Greedy, DP vs D&C` (1061–1124)
- `## Practical Implementations → Common Pitfalls` (1145–1170)

## Source sections cut
- `## DP for NP-Complete → Approximation Algorithms` (842–947) — routed to Ch 11
- `## Applications → CV/NLP` — speculative
- `Quiz_Questions.md` — out of scope
- `## Conclusion`

## Original Claude content (NOT in source)

### Production applications of edit distance
Spell-check, BK-trees, Levenshtein automata, BLAST/HMMER, pg_trgm, Myers' diff algorithm, code-completion ranking. Source covers sequence alignment in bioinformatics; the broader production catalog is Claude's. All marked [verify].

### Hirschberg's 1975 algorithm for O(m+n) space alignment
Source covers full-table sequence alignment but not Hirschberg. Added because space optimization is the load-bearing topic and Hirschberg is the canonical answer to "but I need traceback." [verify] 1975 attribution.

### Knuth's optimization (1971) for OBST
Source covers OBST DP but not Knuth's `O(n²)` reduction. [verify].

### Held-Karp practical n limit (~20–25)
[verify] specific limit; depends on memory and constants.

### Levenshtein 1965 / Wagner-Fischer 1974
Standard attribution. [verify].

### Needleman-Wunsch 1970 / Smith-Waterman 1981
Standard attribution. [verify].

### Myers diff 1986
[verify].

## Factual claims preserved from source
- Optimal substructure + overlapping subproblems framing — from source
- Memoization vs tabulation — from source
- 0/1 knapsack recurrence and `O(nW)` bound (pseudo-polynomial framing) — partly source, partly Claude
- Sequence alignment recurrence — directly from source
- Floyd-Warshall recurrence over `(i, j, k)` — from source

## [verify] count
8 inline `[verify]` markers:
1. Hirschberg 1975
2. Levenshtein 1965
3. Wagner-Fischer 1974
4. Needleman-Wunsch 1970
5. Smith-Waterman 1981
6. Myers diff 1986
7. Knuth OBST optimization 1971
8. Held-Karp practical n limit
9. pg_trgm extension specifics
10. Spell-check engine specifics

(Total: 10 actual inline `[verify]` tags.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: edit distance in production (named in outline.md) ✓
- Failure modes engages "DP is the rigorous one; greedy is sloppy" (continued from Ch 6) ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- Section 3 (Designing a DP recurrence) is load-bearing per outline.md. The four-step process is the chapter's reference move. The hardest step (subproblem definition) is named explicitly.
- Resisted the temptation to teach by example only. The chapter's structure first, examples later. Reference convention.
- Edit-distance worked example uses production applications (per outline.md), not a pedagogical hypothetical. Mobile keyboards, DNA alignment, database fuzzy matching, diff, code completion.
- The DP-vs-greedy contrast bookends Chapter 6's misconception engagement. Both chapters point at each other; the practitioner sees "use the matched tool" as the reading.

## Word count
~3,170 words (target: ~3,200; within −1%)

## Open issues for editor
- All historical attribution dates need verification
- pg_trgm and BLAST/HMMER citations could be made more specific
- Production application examples could be replaced with citations to specific implementation docs
- Decision rules table has 9 rows; layout decision for editor
