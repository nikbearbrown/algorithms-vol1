# Figure brief — Chapter 09: Network Flow

## Recommended figures

### Figure 1 — Flow network with capacities and a flow
**Path:** `../images/chapter-9-flow-network.jpg`
**Description:** Small directed graph (~6 vertices) with source and sink labeled. Each edge labeled `f/c` (current flow / capacity). Conservation visible at each non-terminal vertex. Caption notes the flow value at the sink.
**Use:** referenced in §3 (Flow networks).

### Figure 2 — Residual graph and augmenting path
**Path:** `../images/chapter-9-residual-augmentation.jpg`
**Description:** Same graph as Figure 1 with the residual graph overlaid: forward edges showing remaining capacity, backward edges showing current flow. Augmenting path highlighted, bottleneck capacity annotated.
**Use:** referenced in §4 (Ford-Fulkerson).

### Figure 3 — Bipartite matching reduction
**Path:** `../images/chapter-9-bipartite-matching-reduction.jpg`
**Description:** Original bipartite graph (workers L, tasks R, edges between) on the left. Reduced flow network with source connected to L (unit capacities), R connected to sink (unit capacities), original edges as unit-capacity edges. Caption: "Max flow = max matching."
**Use:** referenced in §5 (Reductions).

### Figure 4 — Image segmentation graph construction
**Path:** `../images/chapter-9-segmentation-graph.jpg`
**Description:** A small image (4x4 pixels). Source `s` connected to each pixel with foreground-likelihood edge. Each pixel connected to sink `t` with background-likelihood edge. Adjacent pixels connected pairwise with similarity edges. Min-cut highlighted; pixels colored by which side of the cut they fall on.
**Use:** referenced in §7 (Worked example).

### Figure 5 — Project selection min-cut
**Path:** `../images/chapter-9-project-selection.jpg`
**Description:** Project graph with prerequisites (DAG). Positive-profit projects connected from source; negative-profit connected to sink. Prerequisite edges with infinite capacity. Min-cut partitioning into selected/rejected. Profit annotation.
**Use:** referenced in §5 (Reductions).

## Inline figure-call markers
None inserted in current draft.
