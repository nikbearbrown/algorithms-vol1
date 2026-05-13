# Figure brief — Chapter 11: Approximation Algorithms

## Recommended figures

### Figure 1 — Approximation guarantee classes
**Path:** `../images/chapter-11-approximation-classes.jpg`
**Description:** Hierarchy chart of approximation classes from strongest to weakest guarantee: FPTAS → PTAS → constant-factor → log-factor → no constant-factor approximation (inapproximable). Examples in each band.
**Use:** referenced in §3 (Approximation ratio).

### Figure 2 — Set cover greedy worst-case construction
**Path:** `../images/chapter-11-set-cover-worst-case.jpg`
**Description:** Construction showing greedy choosing larger sets that miss small-set targets, achieving close to `H_n` ratio. Universe of `n` elements with sets of decreasing sizes constructed to force greedy mistakes.
**Use:** referenced in §6 (When the ratio misleads).

### Figure 3 — Vertex cover matched-edge algorithm
**Path:** `../images/chapter-11-vertex-cover-matched-edge.jpg`
**Description:** Small graph. Maximal matching `M` highlighted. Output cover = both endpoints of every edge in `M`. Optimal cover annotated for comparison. Caption: "|M| ≤ OPT, output = 2|M|, ratio = 2."
**Use:** referenced in §5 (Canonical approximation algorithms).

### Figure 4 — Worst-case vs typical-case approximation behavior
**Path:** `../images/chapter-11-worst-vs-typical.jpg`
**Description:** Histogram or scatter plot. X-axis: instances. Y-axis: ratio of algorithm's solution to optimum. Worst-case bound (e.g., 2 or `H_n`) drawn as a horizontal line. Distribution of typical performance falls well below the line. Caption: "The bound is the ceiling, not the prediction."
**Use:** referenced in §6 and §8 (misconception engagement).

### Figure 5 — Sensor placement on water network
**Path:** `../images/chapter-11-sensor-placement.jpg`
**Description:** Schematic water-distribution network with junctions and pipes. A subset of junctions colored as "sensor locations" (chosen by greedy). Coverage indicated by shading downstream junctions. Caption: "Greedy: 1.1× to 1.3× optimal in practice; bound is `H_n` ≈ 9.21."
**Use:** referenced in §7 (Worked example).

## Inline figure-call markers
None inserted in current draft.
