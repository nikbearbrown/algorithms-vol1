# Research notes — Chapter 10: NP-Completeness and Intractability

## Source folders used
- Primary: `Intractability/README.md` — **VERY THIN** (only lesson outlines, no body content)

## Composition strategy
Per source-chapter-map.md, this is the highest-risk chapter for fabrication. Source provides only lesson topic outlines. The chapter is composed primarily from canonical NP-completeness knowledge, with [verify] discipline on every reduction, every canonical NP-complete problem, and every complexity-class claim.

The chapter anchors to widely-canonical references throughout:
- Cook 1971 (NP-completeness of SAT)
- Levin 1973 (independent NP-completeness proof)
- Karp 1972 (21 NP-complete problems)
- Garey & Johnson 1979 (Computers and Intractability)

## Original Claude content (essentially the entire chapter)

### Class definitions (P, NP, NP-hard, NP-complete)
Standard textbook definitions. Marked [verify] on Khachiyan 1979 (LP in P).

### Canonical reductions list
Standard textbook reductions — 3-SAT to vertex cover, independent set, clique, graph coloring, ILP, etc. All widely-canonical. The constructions themselves are not given in the chapter (companion page); only the existence is asserted.

### NP-complete problem catalog
- SAT/3-SAT (Cook-Levin)
- Vertex cover, independent set, clique
- Graph coloring (NP-hard for k≥3)
- Hamiltonian cycle/path
- TSP (Christofides 1976 1.5-approximation [verify])
- Subset sum, knapsack, partition
- Set cover (Feige 1998 inapproximability [verify])
- Bin packing
- ILP
- Steiner tree

### "What to do" section (§6)
The seven strategies are widely-discussed in modern texts. The framing here is Claude's, but the content is standard:
1. Exact exponential (Held-Karp, branch-and-bound)
2. Approximation (Chapter 11)
3. Heuristic (local search, SA, GA)
4. Special case (bipartite vertex cover, planar graphs, 4-Color Theorem)
5. FPT
6. SAT/ILP encoding
7. Phase-transition awareness

### Worked example — course scheduling → graph coloring
Standard textbook reduction. The "what universities actually do" section is composed from general knowledge of academic scheduling practice. All specific implementation claims marked [verify].

### Modern SAT/ILP solver names
Glucose, MiniSAT, CaDiCaL, Kissat are real solvers. Gurobi, CPLEX, OR-Tools are real ILP solvers. [verify] specific capabilities and benchmark claims.

### 3-SAT phase transition at clause-to-variable ratio 4.26
Widely-cited number from random 3-SAT studies (Kirkpatrick-Selman, Mitchell-Selman-Levesque, etc.). [verify].

### Held-Karp practical n limit (~25)
[verify].

### FPT vertex cover practical k limit (~30)
[verify].

### Four Color Theorem (planar graph 4-coloring)
Appel-Haken 1976. [verify].

## Source content kept (extremely limited)
- Lesson topics from Intractability/README.md served as a structural skeleton for the chapter:
  - NP-Completeness fundamentals → §3 (P, NP, NP-hard, NP-complete)
  - Reductions and NP-Hard → §4 (Reductions)
  - NP-Completeness Proofs → §4 (mention)
  - Understanding PSPACE → mentioned in §10 anti-capability (deferred to companion page)
  - Extending Tractability → §6 (What to do when a problem is NP-complete)

## [verify] count
17 inline `[verify]` markers — highest in any chapter so far, as expected per the source-chapter-map.md flag:

1. Garey-Johnson 1979 catalog
2. Khachiyan 1979 (LP in P)
3. Cook 1971 / Levin 1973 (NP-completeness of SAT)
4. Karp 1972 (21 NP-complete problems)
5. Christofides 1976 1.5-approximation for metric TSP
6. Feige 1998 set cover inapproximability
7. Garey-Johnson catalog "300+" problems
8. Held-Karp practical n limit (~25)
9. Modern SAT solvers list (Glucose, MiniSAT, CaDiCaL, Kissat)
10. Four Color Theorem (k=4 planar polynomial)
11. FPT vertex cover practical k (~30)
12. Modern ILP solver claims (Gurobi, CPLEX, OR-Tools timing)
13. 3-SAT phase transition at ratio 4.26
14. DSATUR / RLF heuristics for graph coloring
15. ILP solvers handling university scheduling in minutes-to-hours
16. (Inferred — counted multi-mention items separately)

(Final inline tag count: ~14, depending on how multi-mentions are counted. Significantly higher than other chapters in this batch as expected.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: course scheduling → graph coloring (named in outline.md) ✓
- Failure modes engages "NP-complete means impossible" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- The "what to do" section §6 must not be buried per outline.md note. Placed before decision rules and worked example so the reader sees the actionable content as the chapter's spine.
- Resisted writing a complete proof of Cook-Levin theorem on the page. Reference convention: state the result, name the canonical reference, point at the companion page for the proof.
- The misconception engagement is unusually long because "NP-complete means impossible" is widely held and consequential. Made the corrective concrete rather than abstract.
- Chapter may read as more textbook-philosophical than the others because the subject is more conceptual. Held to reference voice (decision-rule-first, named tools, no editorial hand-wringing about open problems).

## Word count
~3,180 words (target: ~3,000; within +6%)

## Open issues for editor
- This chapter has the highest [verify] count and the highest fabrication risk. Every historical attribution, every canonical NP-complete problem assertion, every solver-capability claim should be cross-checked against canonical references — Garey-Johnson, Sipser, Arora-Barak, Papadimitriou.
- The seven-strategy framework in §6 is the chapter's most distinctive content. The framing should be reviewed for completeness — is something missing? (Quantum algorithms? Out of scope per book.md.)
- The course-scheduling worked example uses general knowledge of academic scheduling. Specific claims (DSATUR/RLF use, ILP solver behavior on real instances) should be verified or genericized.
- Phase-transition number 4.26 is widely-cited but the precise value depends on the formula generation model; check against current literature.
- Consider whether to add a one-paragraph note about quantum computing as an explicit-out-of-scope reference (Shor's algorithm, Grover's, BQP) — currently omitted; could be added in revision.
