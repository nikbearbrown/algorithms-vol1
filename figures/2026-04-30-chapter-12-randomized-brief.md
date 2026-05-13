# Figure brief — Chapter 12: Randomized Algorithms

## Recommended figures

### Figure 1 — Monte Carlo vs Las Vegas
**Path:** `../images/chapter-12-monte-carlo-vs-las-vegas.jpg`
**Description:** Two-panel comparison. Left: Monte Carlo with fixed time bound, output may be wrong with bounded probability, showing a histogram of outputs across runs. Right: Las Vegas with always-correct output, time as a random variable, showing a distribution of running times. Caption: "MC fixed time, probabilistic correctness; LV certain correctness, probabilistic time."
**Use:** referenced in §3 (Monte Carlo vs Las Vegas).

### Figure 2 — Karger's contraction sequence
**Path:** `../images/chapter-12-karger-contraction.jpg`
**Description:** Six-panel sequence on a small graph (8 vertices). Each panel shows one contraction step with the chosen edge highlighted. Final panel: 2 vertices remaining, the edges between them highlighted as the candidate cut. Caption: "Random contractions until 2 vertices; edges between are the cut."
**Use:** referenced in §6 (Worked example).

### Figure 3 — Concentration inequality comparison
**Path:** `../images/chapter-12-concentration-inequalities.jpg`
**Description:** Plot showing tail probability `P(X > t · μ)` as a function of `t`. Curves: Markov bound (1/t), Chebyshev (variance/t²), Chernoff (exponentially decaying). Caption: "Stronger assumptions buy tighter tails."
**Use:** referenced in §4 (The two essential tools).

### Figure 4 — Reservoir sampling
**Path:** `../images/chapter-12-reservoir-sampling.jpg`
**Description:** Stream of items arriving sequentially. Reservoir of size 3 holding the current sample. Each new item shown with the probability `k/i` of replacing a reservoir item. Final state: uniform sample of size 3.
**Use:** referenced in §5 (Canonical algorithms).

### Figure 5 — Cryptographic vs algorithmic randomness
**Path:** `../images/chapter-12-prng-vs-csprng.jpg`
**Description:** Two side-by-side flow diagrams. Top: PRNG (seeded, deterministic from seed, output predictable given enough samples). Bottom: CSPRNG (entropy-seeded, computationally indistinguishable from random, output unpredictable even with partial knowledge). Boxes annotated with use cases ("OK for: simulation, sampling" / "Required for: keys, tokens, nonces").
**Use:** referenced in §6 (Cryptographic vs algorithmic randomness).

## Inline figure-call markers
None inserted in current draft.
