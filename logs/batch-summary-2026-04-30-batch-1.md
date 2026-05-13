# Batch Summary — Batch 1 (Chapters 1–5)

**Drafted:** 2026-04-30
**Drafter:** Claude (sonnet)
**Total words this batch:** 12,488
**Total `[verify]` count this batch:** 37

---

## Chapters drafted

| Ch | Slug | Word count | Target | Δ | `[verify]` |
| --- | --- | --- | --- | --- | --- |
| 01 | introduction | 1,704 | 1,500 | +14% | 1 |
| 02 | algorithm-analysis | 2,906 | 3,200 | −9% | 6 |
| 03 | data-structures | 2,862 | 3,000 | −5% | 9 |
| 04 | sorting-and-caching | 2,526 | 2,800 | −10% | 12 |
| 05 | graphs | 2,490 | 3,000 | −17% | 9 |

All within ±20% of target. Ch 5 closest to the lower bound.

## `[verify]` count, sorted descending

1. **Ch 04 — Sorting and Caching** (12). Concentration in production-sort dates and thresholds (Timsort/Introsort/Dual-Pivot adoption years and switch points), CDN cache-replacement gain ranges, ARC patent expiration.
2. **Ch 03 — Data Structures** (9). HyperLogLog implementations (Redis/BigQuery/Presto), B-tree database internals, Crosby/Wallach attack date, Python dict insertion-ordering version, Tarjan inverse-Ackermann attribution.
3. **Ch 05 — Graphs** (9). Historical attribution dates (Dijkstra 1956, Bellman/Ford 1950s, Roy/Floyd/Warshall 1959–1962, Johnson 1977, Borůvka 1926); worked-example numerics (10M/25M graph, A* exploration ratio); Babai 2015 graph-isomorphism attribution.
4. **Ch 02 — Algorithm Analysis** (6). Memory-hierarchy latency numbers; production-sort thresholds (CPython, JDK, libstdc++).
5. **Ch 01 — Introduction** (1). *Algorithms in a Nutshell* out-of-print date.

---

## Source-routing decisions

The `Algorithm_Analysis/` folder was the most cross-routed source per `source-chapter-map.md`. Decisions made this batch:

- **Sorting algorithms** (Algorithm_Analysis lines 839–1139): pulled into Ch 4. Algorithm-by-algorithm coverage now lives in Ch 4 §3 rather than Ch 2.
- **Graph algorithms** (Algorithm_Analysis lines 1140–1255): pulled into Ch 5. Complexity-table content went into Ch 5 decision rules.
- **Hash-table content** from Algorithm_Analysis: routed into Ch 3 §6 (basic hashing). Randomized-hashing analysis stays for Ch 12.
- **Greedy/D&C/DP/Approximation/Randomized** Algorithm_Analysis subsections: NOT touched in this batch. Will be picked up by Chapters 6, 7, 8, 11, 12 in subsequent batches.
- **Parallel algorithms** (Algorithm_Analysis lines 1819–1904): cut entirely (out of scope for classical canon).

The `Sorting_Caching.md` "Caching Algorithms and Data Structures" subsection (bloom filter, count-min): routed primary coverage to Ch 3 §7 (probabilistic structures). Ch 4 cross-references rather than duplicates.

The `Graph_Algorithms.md` "Network Flow and Matching" section: deferred entirely to Ch 9. The "Minimum Cut → Random Contraction" section (Karger's): deferred entirely to Ch 12.

---

## Structure-drift self-check (per chapter)

For each chapter in this batch, verify three conditions: (a) Section 1 is titled "When X applies" or "Recognition pattern"; (b) chapter ends with anti-capability + capability paragraphs; (c) chapter has no exercises, no learning objectives, no chapter-opening hook.

| Ch | (a) Section 1 title | (b) Ends with anti-cap + cap | (c) No exercises/objectives/hook |
| --- | --- | --- | --- |
| 01 | "When this book applies" ✓ | ✓ | ✓ |
| 02 | "When asymptotic analysis applies" ✓ | ✓ | ✓ |
| 03 | "When the choice of data structure matters" ✓ | ✓ | ✓ |
| 04 | "Recognition pattern" ✓ | ✓ | ✓ |
| 05 | "Recognition pattern" ✓ | ✓ | ✓ |

**All pass.** Two grep hits for forbidden words (Ch 1 line 39, Ch 2 line 21) reviewed: both are meta-references, not actual exercises or learning objectives. Ch 1 line 39 is the explicit statement of reference-shape ("There are no exercises, no learning objectives, no chapter-opening hooks. This is a reference book."); Ch 2 line 21 uses "warm-up" metaphorically about a companion-page pointer ("the recursion refresher on the companion page is the warm-up").

No structural drift detected.

---

## Voice-drift self-check

**Re-read of Section 1 of Chapter 1:** Direct, plain, slightly dry. Opens with "You are a working software engineer with a problem in front of you." No wonder, no scene, no ramp. Reference voice.

**Re-read of Section 1 of Chapter 5:** Direct: "You have entities that connect to each other and a question whose answer involves those connections." Lists the entity-types and connection-types as concrete examples, then names the diagnostic signal. Reference voice held.

**Comparative check across the batch:** the five chapters share a consistent register — direct openings, decision-rule-first emphasis, "you" not "we," practitioner framing throughout. The closest drift toward textbook-warmth occurs in Chapter 2's §6 (cache effects), where the original-content nature of the section produced one early draft pass that read as "let me explain why this matters." That pass was pulled back; the published Section 6 leads with mechanism (memory-hierarchy ratios) and concrete consequences.

The five chapters do not drift toward Fry/Attenborough cold-open scenes anywhere. The Voice.md file in `plugins/emma/` is in fact Fry voice (cold open, scene-first, moral arrival); used per prompt instruction for prose register only — forbidden phrases, etymology-as-mnemonic, no-jargon-without-explanation, sentence-rhythm — and explicitly NOT for structural opening conventions. The reference-book "no cold open, TL;DR + Section 1" rule was held.

No voice drift detected.

---

## Cross-reference integrity check

Every cross-reference in this batch points to a chapter whose number matches `outline.md`:

| Source ch | Cross-ref to | Status |
| --- | --- | --- |
| 01 | Ch 2 (Big O, prerequisites) | ✓ |
| 01 | Ch 3 (data structures, prerequisites) | ✓ |
| 01 | Vol. 2 (Bayesian, decision-making) | ✓ |
| 01 | Magazine (ML algorithms) | ✓ |
| 02 | Ch 7 (master theorem applied to D&C) | ✓ |
| 02 | Ch 3 (amortized in dynamic table) | ✓ |
| 02 | Ch 4 (cache effects in sorting) | ✓ |
| 02 | Ch 12 (randomized analysis) | ✓ |
| 02 | Ch 10 (NP-hardness) | ✓ |
| 03 | Ch 2 §5 (amortized analysis) | ✓ |
| 03 | Ch 4 (heap sort) | ✓ |
| 03 | Ch 5 (Dijkstra/Prim with heaps) | ✓ |
| 03 | Ch 6 (Kruskal with union-find) | ✓ |
| 03 | Ch 12 (randomized hashing, LSH) | ✓ |
| 04 | Ch 3 §4 (heaps for heapsort) | ✓ |
| 04 | Ch 12 (randomized quicksort) | ✓ |
| 04 | Ch 2 §4, §6 (master theorem, cache) | ✓ |
| 05 | Ch 3 §4, §6 (heaps, union-find) | ✓ |
| 05 | Ch 8 (DP framing of BF, FW) | ✓ |
| 05 | Ch 6 (MST as greedy-on-matroid) | ✓ |
| 05 | Ch 9 (network flow) | ✓ |
| 05 | Ch 12 (Karger's min-cut) | ✓ |

All cross-references resolve to chapters that exist in `outline.md`. Some target chapters are not yet drafted — that is expected at this batch position and consistent with the prompt's guidance (use the chapter number from outline.md regardless of draft status).

---

## Open issues for the editor

1. **`[verify]` markers concentrated in production-tooling specifics** (Ch 4 dates and thresholds especially). These should be cross-checked against current source code — CPython `Objects/listobject.c`, OpenJDK `java.util.DualPivotQuicksort`, libstdc++ `bits/stl_algo.h` — at editorial pass.

2. **Worked-example numerics** in Ch 4 case A (12% → 2% benchmark) and Ch 5 (10M vertices / 25M edges, 1–10% A\* exploration) are illustrative. If real benchmark traces are available, they should replace these or be cited alongside.

3. **Historical attribution dates** in Ch 5 (Dijkstra 1956, Bellman-Ford 1950s, etc.) are widely-cited but the specific publication dates and venues should be verified against canonical references — CLRS bibliography is a good first check.

4. **Empty source folder for Ch 1** (`Introduction_to_Algorithms/README.md`). The chapter was composed from `book.md` and `outline.md`. If the editor has additional source material for the introduction, the chapter may want a second pass to incorporate it.

5. **Decision-rules tables** in Chapters 3, 4, 5 are dense (10+ rows). Layout decisions for the Kindle format may want consolidation or splitting.

6. **`[verify]` for *Algorithms in a Nutshell* out-of-print date** (Ch 1). Confirmed in book.md but the specific year ("2015") was added by Claude based on the book.md framing; double-check against the O'Reilly catalog.

7. **`[verify]` for ARC patent expiration** (Ch 4). IBM's ARC patents are real but the year ("around 2019") needs cross-check.

8. **Ch 2 §6 cache-effects section is original to the chapter** (not in source). Per outline.md, this is intentional and the chapter's differentiation from CLRS. Editor should review the latency numbers and the L1/RAM ratio against current hardware references at publication time.

---

## Files produced this batch

- `chapters/2026-04-30-chapter-01-introduction.md`
- `chapters/2026-04-30-chapter-02-algorithm-analysis.md`
- `chapters/2026-04-30-chapter-03-data-structures.md`
- `chapters/2026-04-30-chapter-04-sorting-and-caching.md`
- `chapters/2026-04-30-chapter-05-graphs.md`
- `research/2026-04-30-chapter-01-introduction.md`
- `research/2026-04-30-chapter-02-algorithm-analysis.md`
- `research/2026-04-30-chapter-03-data-structures.md`
- `research/2026-04-30-chapter-04-sorting-and-caching.md`
- `research/2026-04-30-chapter-05-graphs.md`
- `figures/2026-04-30-chapter-01-introduction-brief.md`
- `figures/2026-04-30-chapter-02-algorithm-analysis-brief.md`
- `figures/2026-04-30-chapter-03-data-structures-brief.md`
- `figures/2026-04-30-chapter-04-sorting-and-caching-brief.md`
- `figures/2026-04-30-chapter-05-graphs-brief.md`
- `logs/source-chapter-map.md` (one-time map)
- `logs/drafting-run-log.csv`
- `logs/batch-summary-2026-04-30-batch-1.md` (this file)

---

## Next batch starting chapter

**Batch 2 starts at Chapter 06 — Greedy Algorithms.**

Chapters in Batch 2: 06 (Greedy), 07 (Divide and Conquer), 08 (Dynamic Programming), 09 (Network Flow), 10 (NP-Completeness and Intractability).

Note: Ch 10 has very thin source (`Intractability/README.md` is only lesson outlines — no body content). Ch 10 will be high-risk for fabrication; the chapter must compose from canonical NP-completeness knowledge with `[verify]` discipline. Editor should be aware.
