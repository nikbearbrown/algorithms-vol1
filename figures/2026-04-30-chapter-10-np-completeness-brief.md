# Figure brief — Chapter 10: NP-Completeness and Intractability

## Recommended figures

### Figure 1 — Complexity class hierarchy
**Path:** `../images/chapter-10-class-hierarchy.jpg`
**Description:** Venn-style diagram showing P, NP, NP-complete, NP-hard, and undecidable. P inside NP. NP-complete is the intersection of NP and NP-hard. NP-hard extends beyond NP. Undecidable problems sit outside all of them. Examples annotated in each region.
**Use:** referenced in §3 (Class definitions).

### Figure 2 — 3-SAT to graph coloring reduction
**Path:** `../images/chapter-10-3sat-to-coloring.jpg`
**Description:** Small 3-SAT formula on the left. Constructed graph on the right showing the canonical reduction (variable gadgets, clause gadgets, color-class structure). Caption: "Each colored vertex corresponds to a truth assignment; valid coloring iff formula satisfiable."
**Use:** referenced in §4 (Reductions).

### Figure 3 — Course scheduling → graph coloring
**Path:** `../images/chapter-10-course-scheduling-graph.jpg`
**Description:** Six courses, eight student-shared edges between them. The graph annotated with a 3-coloring. Schedule on the right showing time slots = colors.
**Use:** referenced in §7 (Worked example).

### Figure 4 — Phase transition for random 3-SAT
**Path:** `../images/chapter-10-3sat-phase-transition.jpg`
**Description:** Plot. X-axis: clause-to-variable ratio. Y-axis (left): probability of satisfaction (drops from 1 to 0 across the transition). Y-axis (right): median solver runtime (peaks at the transition). Critical region around 4.26 highlighted.
**Use:** referenced in §6 (Phase-transition awareness).

### Figure 5 — Seven strategies decision tree
**Path:** `../images/chapter-10-strategies-decision-tree.jpg`
**Description:** Decision tree starting from "Problem is NP-hard" → branches by question (n small? structured instance? need bounded quality? want best practical?) → leaves at the seven strategies (exact exponential, approximation, heuristic, special case, FPT, SAT/ILP encoding, phase-transition exploit).
**Use:** referenced in §6 and Decision rules.

## Inline figure-call markers
None inserted in current draft.
