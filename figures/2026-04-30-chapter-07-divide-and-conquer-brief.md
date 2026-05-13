# Figure brief — Chapter 07: Divide and Conquer

## Recommended figures

### Figure 1 — D&C anatomy diagram
**Path:** `../images/chapter-7-dc-anatomy.jpg`
**Description:** Three-panel illustration. Left: "Divide" with input split into two halves. Middle: "Conquer" with each half being recursed. Right: "Combine" with the two solutions merged into one. Below each panel, the corresponding term in `T(n) = aT(n/b) + f(n)`.
**Use:** referenced in §3 (Anatomy).

### Figure 2 — Closest-pair strip
**Path:** `../images/chapter-7-closest-pair-strip.jpg`
**Description:** 2D plane with points scattered, vertical dividing line at median x. Recursive δ_L and δ_R highlighted. Strip of width 2δ around the line shaded. Within strip, a 2δ × δ box showing the geometric bound on points-per-box.
**Use:** referenced in §6 (Worked example).

### Figure 3 — Karatsuba algebra
**Path:** `../images/chapter-7-karatsuba-algebra.jpg`
**Description:** Visualization of `(x₁B + x₀)(y₁B + y₀)`. Schoolbook: 4 multiplications. Karatsuba: 3 multiplications via the identity. Recurrence comparison: T(n) = 4T(n/2) + n vs T(n) = 3T(n/2) + n.
**Use:** referenced in §4 (Canonical algorithms).

### Figure 4 — Strassen 7-product trick (overview only)
**Path:** `../images/chapter-7-strassen-overview.jpg`
**Description:** Two 2×2 block matrices, with arrows indicating that 7 multiplications of n/2 × n/2 blocks (instead of 8) suffice. Recurrence T(n) = 7T(n/2) + n² annotated. Caption notes that the algebraic recombination lives on the companion page.
**Use:** referenced in §4.

### Figure 5 — Brute force vs D&C asymptotic comparison
**Path:** `../images/chapter-7-brute-vs-dc-curves.jpg`
**Description:** Log-log plot. X-axis n. Y-axis operations. Curves: O(n²), O(n log n), O(n^1.585). Crossover annotations.
**Use:** referenced in §6 (Worked example) and §5 (When D&C wins).

## Inline figure-call markers
None inserted in current draft.
