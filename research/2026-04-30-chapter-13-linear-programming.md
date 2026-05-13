# Research notes — Chapter 13: Linear Programming

## Source folders used
- Primary: `Linear_Programming/Linear_Programming.md`

## Source sections kept
- `## Introduction to LP` (3–80)
- `## Theoretical Foundations` (81–229)
- `## The Simplex Algorithm` (230–358)
- `## Linear Programming Duality` (359–465)
- `## The Ellipsoid Algorithm` (466–628) — brief
- `## Interior Point Methods` (719–822)
- `## Cutting Plane Methods` (823–907) — brief, in MIP context
- `## Computational Aspects → Numerical Stability, Scaling` (908–1127)
- `## Advanced Topics → Sensitivity Analysis` (1128–1228)
- `## Case Studies → Resource Allocation, Production Planning` (1272–1409) — production planning is worked example anchor

## Source sections cut
- `## Stochastic LP` (1182–1228) — flagged as Vol. 2
- `## Multi-objective LP` — out of scope
- `## Conclusion → Future`
- `## Case Studies → Financial Portfolio` — brief reference only

## Original Claude content (NOT in source)

### LP / IP / MIP / QP / SOCP / SDP family framing
Source covers LP and briefly IP. The full family hierarchy is added by Claude, including:
- 0-1 IP / Binary IP
- QP (convex vs non-convex)
- SOCP
- SDP — Goemans-Williamson MAX-CUT 0.878-approximation [verify]
- General convex programming

### Dantzig 1947 simplex attribution
[verify].

### Klee-Minty 1972 worst-case
[verify] specific publication date.

### Karmarkar 1984
[verify].

### Khachiyan 1979 ellipsoid
[verify] (also referenced in Ch 10).

### Spielman-Teng 2001 smoothed analysis
[verify].

### Refineries used LP since 1950s
[verify] — widely-canonical, but specific date should be verified.

### Production solver names
Gurobi, CPLEX, FICO Xpress, COIN-OR CBC, OR-Tools, SciPy linprog, PuLP, CVXPY, Pyomo are all real software. [verify] specific solver capabilities.

### Goemans-Williamson 1995 MAX-CUT 0.878
[verify].

### Refinery LP scale
"5 crude types, 8 paths, 6 products → 40 variables / 19 constraints" is illustrative. Real refineries have hundreds. [verify].

### Pooling problem mention
Standard non-convex extension; widely-known difficulty.

### Production reality of LP industries
Airline scheduling, refinery, power grid, supply chain, telecom, finance. All are real LP/MIP applications. [verify] specifics if cited at depth.

## Factual claims preserved from source
- LP standard form — from source
- Simplex geometric and algorithmic intuition — from source
- LP duality theorem — from source
- Complementary slackness — from source
- Economic interpretation of duality (shadow prices) — from source
- Interior-point methods overview — from source

## [verify] count
8 inline `[verify]` markers:
1. Dantzig 1947
2. Klee-Minty 1972
3. Spielman-Teng 2001 smoothed analysis
4. Karmarkar 1984
5. Khachiyan 1979 ellipsoid
6. Production simplex implementations handle millions of variables
7. Interior-point iteration count bound
8. Refineries used LP since 1950s
9. Goemans-Williamson 1995 MAX-CUT 0.878

(Total: ~9 actual `[verify]` tags as written.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: production planning at refinery (named in outline.md) ✓
- Failure modes engages "LP is for academics; production systems use heuristics" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- Capstone framing — explicitly closes threads opened in Ch 6, 9, 11 per outline.md note ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- The capstone framing required deliberate care. Per outline.md: "LP at Chapter 13 carries the 'capstone optimization method' framing. The chapter explicitly closes threads opened in earlier chapters." Cross-references to Ch 6 (matroid LPs), Ch 9 (max-flow min-cut as LP duality), Ch 11 (LP relaxation) are explicit in the chapter and section, not just in §11 (Cross-references).
- The duality section §4 is the chapter's intellectual centerpiece. Held the framing tight: economic interpretation, sensitivity analysis, bounds on integer programs, reformulation. Each is a use case, not just a theory point.
- Resisted teaching the simplex pivot operation step-by-step. Reference convention: state the geometric and algorithmic ideas, defer derivation to companion page or textbook reference.
- The misconception engagement is one of the chapter's most assertive sections. "LP is for academics" is a real practitioner block; the corrective is industrial fact, not philosophy.
- Final sentence: "This chapter closes the optimization arc the volume opened with." This is the capstone close per book.md and outline.md framing.

## Word count
~3,330 words (target: ~3,200; within +4%)

## Open issues for editor
- All historical attribution dates need verification
- Refinery LP scale (40 variables, 19 constraints for the small example) is illustrative; if a specific industrial reference is preferred, replace
- The LP/IP/MIP/QP/SOCP/SDP family table could be a separate figure
- Goemans-Williamson MAX-CUT mention is brief; consider whether to expand or move to a footnote
- "Major airlines solve LPs with millions of variables daily" claim should be cited specifically (e.g., crew scheduling at American Airlines, fleet assignment at Delta)
- Power-grid LP claim ("solved every few minutes") should be cited (security-constrained economic dispatch, ISO-NE / PJM / ERCOT references)
