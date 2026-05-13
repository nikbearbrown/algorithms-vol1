# Research notes — Chapter 11: Approximation Algorithms

## Source folders used
- Primary: `Approximation_Algorithms/Approximation_Algorithms.md`
- Secondary: `Algorithm_Analysis/Algorithm_Analysis.md` approximation section (lines 1905–2013), routed in (TSP and Vertex Cover specifically)

## Source sections kept
- `## Introduction to Approximation` (3–139) — definitions, role
- `## Fundamentals → Ratio, PTAS, FPTAS` (140–309)
- `## Design Techniques → Greedy, DP, LP Rounding` (310–425)
- `## List Scheduling Algorithms` (426–536)
- `## Local Search` (537–783) — brief
- `## Probabilistic and Metaheuristic → Simulated Annealing` (784–855) — kept
- `## Case Studies → Vertex Cover, Set Cover, Network Design` (923–1078) — set cover is worked example anchor
- `## Challenges → Limits of Approximability` (1079–1134)

## Source sections cut
- `## Probabilistic and Metaheuristic → Hopfield Networks` (856–922) — magazine
- `## Future of Approximation Algorithms`
- `## Practical Implementations → Real-World` — used selectively
- `## Exercises and Problems`

## Original Claude content (NOT in source)

### Specific approximation ratios and attributions
- Christofides 1976 1.5-approximation
- Steiner tree 1.39-approximation
- 0/1 knapsack FPTAS
- FFD bin packing 11/9 (Yue 1991)
- List scheduling 2 − 1/m (Graham 1966)
- Maximum clique inapproximability `n^(1−ε)` (Håstad 1996)
- PCP theorem (ALMSS 1998)
- TSP 2020 improvements

All marked [verify]. The set of attributions is widely-canonical but specific dates need cross-check.

### Worked example — sensor placement
Source covers vertex/set cover but not the water-utility sensor-placement framing. Per outline.md, the worked example is "sensor placement, server location, or test-suite reduction" — chose sensor placement as primary, listed others. Specific numbers (10,000 junctions, 1,000 candidates) marked [verify].

### When ratios mislead (§6)
This is the chapter's distinguishing content per outline.md note. The five failures are framed by Claude. The phenomena are widely-canonical; the framing as a coordinated set is original to the chapter.

### Application examples
CDN server location, software test-suite reduction (Coverity, SonarQube reference), feature selection. All marked [verify] for specifics.

## Factual claims preserved from source
- Vertex cover greedy and matched-edge approximations — from source
- Set cover greedy `H_n`-approximation — from source
- LP rounding for vertex cover — from source (Design Techniques section)
- Approximation ratio definition — from source
- PTAS / FPTAS definitions — from source

## [verify] count
9 inline `[verify]` markers:
1. Christofides 1976
2. Steiner 1.39 ratio
3. Arora 1998 / Mitchell 1999 (Euclidean TSP PTAS)
4. Bin packing FPTAS impossibility
5. Håstad 1996 (clique inapproximability)
6. PCP theorem ALMSS 1998
7. Yue 1991 (FFD bin packing 11/9)
8. Graham 1966 (list scheduling)
9. TSP 2020 improvements (Karlin-Klein-Oveis Gharan)
10. Worked example numbers (10,000 junctions, 1,000 candidates)
11. Greedy ratio in practice (1.1× to 1.3× optimal on water networks)
12. ILP solver timing on 10,000-junction instance
13. Coverity/SonarQube test-suite reduction techniques

(Total: ~12 inline `[verify]` tags as written.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: set cover for sensor placement (named in outline.md) ✓
- Failure modes engages "approximation guarantee tells you how good the answer is on this instance" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- The "ratio is worst-case, not per-instance" framing carries through. §6 names it explicitly; §8 engages the misconception built around it.
- Resisted teaching PCP theorem on the page. Reference convention: state the result, name the foundational paper, defer derivation to companion page.
- The 2-approximation matched-edge algorithm for vertex cover is described as a tight algorithm in source but the cleaner bound proof is from a different convention. Chose the cleanest production-relevant version.
- The water-utility worked example is concrete and computational; serves as the practitioner anchor.

## Word count
~2,800 words (target: ~2,750; within +2%)

## Open issues for editor
- All historical attribution dates need verification, especially TSP improvements
- Sensor-placement numbers are illustrative; if real benchmark data available, replace
- The "When ratios mislead" framing in §6 could be moved earlier (before Decision rules) for stronger effect; current structure follows outline.md
- Coverity/SonarQube test-suite reduction is a modern engineering claim; could be replaced with citation to academic test-suite minimization literature
