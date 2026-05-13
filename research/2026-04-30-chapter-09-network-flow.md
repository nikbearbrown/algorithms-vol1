# Research notes — Chapter 09: Network Flow

## Source folders used
- Primary: `Network_Flow/Network_Flow.md`

## Source sections kept
- `## Introduction to Network Flow` (3–159) — definitions, key concepts
- `## Fundamentals` (160–279) — graphs and networks, capacity, problem types
- `## Maximum Flow Theory` (280–416) — max-flow min-cut, augmentation, capacity scaling
- `## Algorithms for Maximum Flow → Ford-Fulkerson, Push-Relabel` (417–1076)
- `## Applications → Assignment, Connectivity` (850–1093)
- `## Advanced → Min-Cost Flow, Multi-commodity` (1094–1341)

## Source sections cut
- `## Software and Tools` (1342–1432) — companion page
- `## Case Studies → Transportation, Telecom, Energy` — used selectively, not exhaustively
- `## Challenges → ML Integration`
- `## Exercises and Problems`
- `Quiz_Questions_Network_Flow.md`

## Original Claude content (NOT in source)

### Image segmentation reduction
Per outline.md, this is the worked example. Source covers max-flow algorithms generally but does not develop the segmentation reduction. Added with citations:
- Greig, Porteous, Seheult 1989 [verify]
- Boykov, Kolmogorov 2004 [verify]

The graph construction (source-pixel edges with negative-log-likelihood capacities, sink-pixel edges, pairwise similarity edges) is canonical for binary submodular MRF inference.

### Project selection reduction
Standard textbook reduction. Source covers max-flow but not this reduction explicitly. Added.

### Baseball elimination
Standard textbook reduction. Added.

### Edmonds-Karp (1972)
[verify] year and attribution. Source covers Edmonds-Karp algorithm but I added the year.

### Dinic 1970
[verify] year.

### Goldberg-Tarjan 1988 push-relabel
[verify].

### Menger's theorem 1927
[verify].

### Boykov-Kolmogorov practical performance
"1 megapixel in seconds" claim marked [verify].

### König's theorem mention
Standard theorem. [verify] precise attribution.

### Production library specifics
NetworkX, OR-Tools, LEMON, Boost Graph Library all real. [verify] specific solver capabilities if cited at depth.

## Factual claims preserved from source
- Max-flow min-cut theorem (Ford-Fulkerson 1956 attribution to founding paper)
- Ford-Fulkerson algorithm structure with augmenting paths and residual graphs — from source
- Capacity scaling — from source
- Min-cost flow problem statement — from source
- Multi-commodity flow problem statement — from source

## [verify] count
9 inline `[verify]` markers:
1. Ford-Fulkerson 1956 (max-flow min-cut)
2. Ford-Fulkerson irrational-capacity termination (Zwick example reference)
3. Edmonds-Karp 1972
4. Dinic 1970
5. Goldberg-Tarjan 1988 push-relabel
6. Greig/Porteous/Seheult 1989
7. Boykov-Kolmogorov 2004
8. Menger 1927
9. Boykov-Kolmogorov practical performance numbers
10. König's theorem
11. Adobe Photoshop graph-cut implementation history

(Total: 11 actual `[verify]` tags.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: image segmentation via min-cut (named in outline.md) ✓
- Failure modes engages "Network flow is a niche topic" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- The chapter's argument — "flow is a modeling vocabulary, not a niche algorithm" — runs through. Held in §1, made concrete in §5 (reductions), engaged in §8 (failure modes).
- Resisted the temptation to derive the max-flow min-cut theorem on the page. Reference convention: state the theorem and its consequences; derivation goes to a textbook reference.
- The reductions section §5 is the chapter's most practically useful content per outline.md note. Made vivid: each reduction named, the graph construction stated, the production application named.
- Image segmentation worked example is concrete and computational; serves as the "this is what flow looks like in production" anchor.

## Word count
~2,720 words (target: ~2,650; within +3%)

## Open issues for editor
- All historical attribution dates need verification
- Image-segmentation production-application claims (Adobe Photoshop history, medical imaging pipelines) are widely-believed but specific implementation details should be verified
- The reductions section could carry more — kidney exchange, ad allocation, scheduling — but length budget constrains
- Boykov-Kolmogorov "seconds for 1 megapixel" is approximate; actual benchmarks vary by hardware
