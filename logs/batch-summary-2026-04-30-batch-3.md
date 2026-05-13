# Batch Summary — Batch 3 (Chapters 11–13, FINAL BATCH)

**Drafted:** 2026-04-30
**Drafter:** Claude (sonnet)
**Total words this batch:** 8,730
**Total `[verify]` count this batch:** 30

---

## Chapters drafted

| Ch | Slug | Word count | Target | Δ | `[verify]` |
| --- | --- | --- | --- | --- | --- |
| 11 | approximation | 2,989 | 2,750 | +9% | 12 |
| 12 | randomized | 2,680 | 2,800 | −4% | 9 |
| 13 | linear-programming | 3,061 | 3,200 | −4% | 9 |

All within ±20%. Ch 11 ran slightly over target — the "when ratios mislead" §6 and the failure-modes section together earned the extra space.

## `[verify]` count, sorted descending

1. **Ch 11 — Approximation** (12). Algorithm attribution dates (Christofides 1976, Steiner 1.39, Arora 1998, Mitchell 1999, Yue 1991, Graham 1966); inapproximability results (Håstad 1996, Feige 1998, PCP theorem ALMSS 1998); worked-example numerics; production-application claims (Coverity, SonarQube).

2. **Ch 12 — Randomized** (9). Algorithm attribution (Karger 1993, Karger-Stein 1996, AKS 2002); skip-list production usage (Linux kernel, Redis); randomized-quicksort comparison count; PRNG-related production breach claims.

3. **Ch 13 — Linear Programming** (9). Algorithm attribution (Dantzig 1947, Klee-Minty 1972, Karmarkar 1984, Khachiyan 1979, Spielman-Teng 2001); Goemans-Williamson 1995; refinery LP history ("since 1950s"); production-solver scale claims.

---

## Source-routing decisions (final batch)

The cross-routing initiated in Batches 1–2 completes in this batch. All Algorithm_Analysis subsections now routed to their target chapters.

- **Ch 11 (Approximation) imported** from Algorithm_Analysis: approximation section (lines 1905–2013). TSP and Vertex Cover specifically. Source-chapter-map.md commitment fully realized.
- **Ch 12 (Randomized) imported** from two sources: Algorithm_Analysis randomized section (lines 1677–1818) and Graph_Algorithms.md Karger contraction section (lines 1096–1406). The Karger material is the chapter's worked example.
- **Ch 13 (Linear Programming)** had no Algorithm_Analysis import; Linear_Programming primary source was sufficient.

All thirteen chapters now have full source attribution accounted for. The source-chapter-map.md plan executed without significant deviation.

---

## Structure-drift self-check (per chapter)

| Ch | (a) Section 1 title | (b) Ends with anti-cap + cap | (c) No exercises/objectives/hook |
| --- | --- | --- | --- |
| 11 | "Recognition pattern" ✓ | ✓ | ✓ |
| 12 | "Recognition pattern" ✓ | ✓ | ✓ |
| 13 | "Recognition pattern" ✓ | ✓ | ✓ |

**All pass.** One forbidden-word grep hit (Ch 11, "test exercises a branch") was a verb form, not the exercise-noun — false positive.

No structural drift detected.

---

## Voice-drift self-check

**Re-read of Section 1 of Chapter 11 (first):** "The signal: you have an NP-hard problem (Chapter 10) and you need a polynomial-time algorithm with a quality guarantee." Direct, names the diagnostic move (NP-hard → approximation), states three concrete signals, names the misconception trap. Reference voice held.

**Re-read of Section 1 of Chapter 13 (last):** "The signal: a problem with quantitative decisions, costs that scale linearly with the decisions, and constraints that are inequalities or equalities in those decisions." Direct, lists examples concretely (refinery, network flow, blending, scheduling), names duality structure as second signal, names LP-relaxation as third. Reference voice held.

**Comparative check across the batch:** the three chapters share a consistent register. Ch 13 carries the "capstone" framing per outline.md and book.md, with explicit cross-references back to Ch 6 (matroid LPs), Ch 9 (max-flow min-cut as LP duality), and Ch 11 (LP relaxation). The capstone closing sentence — "This chapter closes the optimization arc the volume opened with" — earns its place.

The chapter most at risk for drift was Ch 12 (Randomized). The cryptographic-vs-algorithmic randomness section is "short but high-stakes" per outline.md, and the temptation is to either gloss it (too short) or lecture (too long). Held to two paragraphs of mechanism plus the corrective. The Karger worked example is the chapter's pedagogical anchor — concrete, surprising, clean linearity-of-expectation analysis.

No voice drift detected.

---

## Cross-reference integrity check

All in-batch cross-references resolve to chapters in `outline.md`:

- Ch 11 → Ch 6, 8, 10, 12, 13: ✓
- Ch 12 → Ch 3, 4, 5, 11: ✓
- Ch 13 → Ch 6, 9, 10, 11, 12: ✓

The capstone cross-references in Ch 13 close the volume's optimization arc explicitly, as required by book.md.

---

## Open issues for the editor (this batch)

1. **Ch 11**: The "When ratios mislead" §6 is the chapter's most practitioner-discriminating content per outline.md. Per current structure it appears between Canonical algorithms and Decision rules; reviewers should confirm placement is right. Could move earlier for stronger framing effect.

2. **Ch 12**: The cryptographic vs algorithmic randomness section is high-stakes; the "production breaches" claim is generic. Specific incident citations (Debian SSH key vulnerability 2008, Sony PS3 ECDSA 2010, various crypto wallet incidents) could be added if editorial wants concrete examples.

3. **Ch 13**: The refinery LP scale numbers ("5 crude types, 8 paths, 6 products → 40 variables / 19 constraints") are illustrative. Real refinery LPs have hundreds of variables. Replace with citation to specific industry reference if available.

4. **Ch 13**: Production LP scale claims ("major airlines solve LPs with millions of variables daily", "power grid LP solved every few minutes") should carry specific citations (American Airlines crew scheduling literature, ISO-NE / PJM / ERCOT economic dispatch documentation) before publication.

---

## Files produced this batch

- `chapters/2026-04-30-chapter-11-approximation.md`
- `chapters/2026-04-30-chapter-12-randomized.md`
- `chapters/2026-04-30-chapter-13-linear-programming.md`
- `research/2026-04-30-chapter-11-approximation.md`
- `research/2026-04-30-chapter-12-randomized.md`
- `research/2026-04-30-chapter-13-linear-programming.md`
- `figures/2026-04-30-chapter-11-approximation-brief.md`
- `figures/2026-04-30-chapter-12-randomized-brief.md`
- `figures/2026-04-30-chapter-13-linear-programming-brief.md`
- `logs/drafting-run-log.csv` (appended)
- `logs/batch-summary-2026-04-30-batch-3.md` (this file)
- `logs/drafting-run-final-summary.md` (the volume-wide final report)

---

## End of drafting run

All thirteen chapters drafted. The final summary in `logs/drafting-run-final-summary.md` carries the volume-wide checks (cross-reference integrity, structure conformance, misconception engagement, anti-capability, worked-example, systemic issues) per Section 10 of the drafting prompt.

After this batch, every chapter's status in `outline.md` is `drafted`. The next stage is editorial review.
