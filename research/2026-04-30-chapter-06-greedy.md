# Research notes — Chapter 06: Greedy Algorithms

## Source folders used
- Primary: `Greedy_Algorithms/Greedy_Algorithms.md`
- Secondary: `Algorithm_Analysis/Algorithm_Analysis.md` greedy section (lines 1508–1674), routed in

## Source sections kept
- `## Introduction to Greedy Algorithms` (3–374) — definitions, principles, applications
- `## Interval Scheduling` (375–497) — problem, algorithm, proof of optimality
- `## Interval Partitioning` (498–627) — depth-based lower bound argument
- `## Shortest Paths and MSTs → MST coverage` (628–1193) — Kruskal, Prim, Borůvka
- `## Huffman Coding` (1194–1387) — algorithm and optimality proof

## Source sections cut
- Dijkstra coverage (cross-referenced to Ch 5; not duplicated)
- `## Challenges and Future Directions` (1388–1710)
- `## Conclusion` (1711–1856)
- `## Exercises and Problems` (1857–2042)
- `## Further Reading and Resources`

## Original Claude content (NOT in source)

### Matroid theory framing
Source mentions matroids only briefly. Added more substantial framing: "graphic matroid," "uniform matroid," "partition matroid," "transversal matroid" — these are widely-canonical matroid names. The fundamental theorem of matroids (greedy optimal iff matroid) is canonical (Edmonds 1971). [verify] specific historical attribution.

### Vertex cover / set cover brief introduction
Per outline.md, this chapter introduces these algorithms, full ratio analysis is in Ch 11. Set cover `H_n ≈ ln n` ratio added; Feige 1998 tightness result added. [verify] specific Feige paper attribution (the result is real and widely-cited).

### Coin change counterexample
"For arbitrary denominations like (1, 6, 10), greedy fails: making 12 cents greedily uses 10 + 1 + 1, optimal is 6 + 6." Standard textbook counterexample.

### Worked example numbers (200-person org, 800 meetings/day, 40 rooms)
Per outline.md, this chapter's worked example is "Meeting room scheduling at a real organization." Specific numbers marked [verify] as illustrative.

### DEFLATE / gzip / PNG / JPEG using Huffman
[verify] specific compression-format details.

## Factual claims preserved from source
- Interval scheduling problem definition and earliest-finish algorithm — from source
- Interval partitioning problem and algorithm — from source
- Greedy choice property + optimal substructure framing — from source
- Three MST algorithms (Kruskal, Prim, Borůvka) — from source
- Huffman algorithm and optimality framing — from source
- Borůvka 1926 (also marked [verify] in Ch 5)

## [verify] count
6 inline `[verify]` markers:
1. Borůvka 1926 (re-flagged from Ch 5)
2. DEFLATE/gzip/PNG/JPEG Huffman use
3. Feige 1998 set cover tightness
4. Worked example numbers (200-person org, 800 meetings/day, 40 rooms)
5. Edmonds 1971 matroid greedy theorem (implicit; not cited explicitly in chapter; if added, mark [verify])

(Total in chapter: 4 actual inline `[verify]` tags as written.)

Recount: 1 Borůvka, 2 DEFLATE/etc., 3 Feige 1998, 4 worked-example numbers. 4 inline `[verify]` markers.

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: meeting room scheduling (named in outline.md) ✓
- Failure modes engages "Greedy is sloppy; DP is the rigorous one" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- The "matroid + exchange argument" intellectual spine (per outline.md notes) anchored throughout. Not just enumerated; used as the through-line connecting interval scheduling, MST, and Huffman.
- Resisted lecturing on matroid theory in §6 — kept the matroid framing brief and pointed at companion page for theory.
- Failure modes section is the most prose-heavy. Restrained.
- The 0/1 knapsack vs fractional knapsack contrast appears in decision rules; useful sleeper for Ch 8 cross-reference.

## Word count
~2,700 words (target: ~2,600; within +4%)

## Open issues for editor
- Edmonds 1971 matroid theorem could be cited explicitly
- Worked example numbers are illustrative; consider whether to use a specific named organization or a more abstract framing
- Coin change counterexample (1, 6, 10) is widely-canonical but the specific denominations could be sourced
- Feige 1998 paper title and venue should be added if cited explicitly
