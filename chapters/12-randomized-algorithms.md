# Randomized Algorithms

## TL;DR

A randomized algorithm makes random choices during execution. Reach for this chapter when you want to defeat adversarial inputs, when expected-time bounds beat worst-case bounds, when a simple randomized algorithm replaces a complicated deterministic one, or when the problem is genuinely probabilistic. After consulting it, you can choose between Monte Carlo (fixed time, probabilistic correctness) and Las Vegas (probabilistic time, certain correctness), apply linearity of expectation and basic concentration inequalities, recognize the canonical randomized algorithms, and distinguish algorithmic from cryptographic randomness.

## Recognition pattern

Four signals.

*Adversarial input would defeat a deterministic algorithm.* Quicksort with a fixed pivot policy is `O(n²)` on adversarial input; randomized pivot makes the expected case `O(n log n)` regardless of input. Hash tables with deterministic hash functions are vulnerable to collision attacks; per-process random seeds defend. Whenever input might be hostile, randomization is a defense.

*A simpler randomized algorithm replaces a complex deterministic one.* Karger's min-cut is a few lines; the deterministic Stoer-Wagner min-cut is more involved. Randomized quickselect is simpler than median-of-medians for `k`-th order statistic. The randomized algorithm is often easier to write and easier to reason about.

*Concentration of measure makes the expected behavior reliable.* When you run a randomized algorithm on a problem with many independent random choices, the law of large numbers and concentration inequalities (Markov, Chebyshev, Chernoff) make the actual behavior tightly clustered around expectation. "Expected `O(n log n)`" with high concentration is a stronger statement than "average-case `O(n log n)`" — the algorithm's variance is bounded.

*The problem is inherently probabilistic.* Counting approximations (Karger), nearest-neighbor approximation (LSH), Bayesian inference, simulation-based estimation. Some problems are best framed as probabilistic from the start.

A signal randomization is *not* the right tool: the problem is small and deterministic algorithms suffice. Randomization buys robustness, simplicity, or expected-time bounds; if you do not need any of those, the deterministic algorithm is preferable for reproducibility.

## What you need to know first

This chapter assumes basic probability — sample spaces, events, expectation, variance. The probability primer on the companion page covers the prerequisites for any reader rusty on the basics. The chapter cites it once and uses it throughout.

This chapter also assumes Big O (Chapter 2) and graph fundamentals (Chapter 5). Karger's algorithm uses the contraction operation introduced in Chapter 5 §6. Randomized quicksort builds on quicksort (Chapter 4 §3 and Chapter 7).

For randomized rounding as an approximation technique, see Chapter 11 §4. For the algorithmic-complexity-attack defense via universal hashing, see Chapter 3 §6.

## Monte Carlo vs Las Vegas

Two ways to use randomness.

**Monte Carlo.** Fixed running time, probabilistic correctness. The algorithm always finishes within a known time bound; the answer may be wrong with bounded probability. To improve correctness, run the algorithm multiple times. Examples: primality testing (Miller-Rabin), Karger's min-cut. The probability of error decreases exponentially in the number of trials.

**Las Vegas.** Probabilistic running time, certain correctness. The algorithm always returns the right answer; the running time is a random variable, with bounded expectation. Examples: randomized quicksort, randomized quickselect. To improve expected time, restart on slow runs.

The two are duals. Many randomized problems admit both formulations: a Monte Carlo algorithm with bounded error becomes a Las Vegas algorithm by repeated trial until verification succeeds (when verification is cheap). A Las Vegas algorithm becomes Monte Carlo by truncating execution at a time bound and accepting whatever partial result remains.

The choice depends on what you can verify. If correct answers are easy to verify but hard to compute, Las Vegas is natural — keep trying until you get one that verifies. If correct answers are not easy to verify, Monte Carlo with multiple trials and majority vote (or amplification) is the better fit.

## The two essential tools

**Linearity of expectation.** `E[X + Y] = E[X] + E[Y]`, regardless of whether `X` and `Y` are independent. This is the workhorse of randomized analysis. Many bounds that look hard become easy via linearity: decompose the random quantity into a sum of indicator variables, compute the expectation of each, sum.

Example. The expected number of comparisons made by randomized quicksort on `n` elements is `2n H_n − 4n ≈ 1.39 n log n` [verify]. The proof: `E[comparisons]` = sum over pairs `(i, j)` of `P(i and j are compared)`. Each probability is computable; the sum becomes the harmonic number bound.

**Concentration inequalities.** Bound the probability that a random variable deviates from its expectation. Three increasingly strong forms.

*Markov's inequality.* For non-negative `X` and `t > 0`: `P(X ≥ t) ≤ E[X] / t`. Crude but always applicable.

*Chebyshev's inequality.* `P(|X − E[X]| ≥ k σ) ≤ 1/k²`, where `σ` is the standard deviation. Stronger than Markov when variance is bounded.

*Chernoff bounds.* For sums of independent bounded random variables, the tail decays exponentially in the deviation. The bound that justifies "high probability" claims in randomized analysis.

Use Chernoff when summing independent indicators; use Chebyshev when you have variance but not independence; use Markov when you have only non-negativity.

## Canonical randomized algorithms

**Randomized quicksort.** Pick the pivot uniformly at random from the subarray. Expected running time `O(n log n)` regardless of input. Worst case `O(n²)` is reached only with vanishing probability under random pivot choice. The algorithm Las Vegas — always correct, expected time bounded.

**Randomized quickselect.** Find the `k`-th smallest element in expected `O(n)` time. Pivot selection randomized; recurse on the side containing position `k`. The expected linear bound is one of the cleanest applications of linearity of expectation.

**Karger's min-cut.** Worked example below.

**Reservoir sampling.** Select a uniformly random `k`-element sample from a stream of unknown length, in `O(n)` time and `O(k)` space. Algorithm: keep the first `k` items; for the `i`-th item (`i > k`), replace a random sampled item with probability `k / i`. Each item ends up in the final sample with probability `k / n`.

**Locality-sensitive hashing (LSH).** Approximate nearest-neighbor search in high dimensions via hash functions that map nearby points to the same bucket with high probability. The deterministic alternative — building tree-based indexes that work in low dimensions — fails in high dimensions due to the curse of dimensionality. LSH gets sub-linear query time with bounded approximation error.

**Bloom filter.** Probabilistic set membership with one-sided error. Covered in Chapter 3 §7; the analysis is randomized.

**Skip list.** Randomized alternative to balanced search trees. Operations `O(log n)` expected; implementation simpler than red-black or AVL trees. The Linux kernel uses skip lists for some purposes [verify]; Redis uses skip lists in sorted sets [verify].

**Randomized rounding for LP relaxation.** Solve the LP, treat fractional values as probabilities, round each variable to 1 with that probability. Often gives the same approximation ratio as deterministic LP rounding (Chapter 11) with a simpler analysis.

**Miller-Rabin primality test.** Probabilistic test for primality. Composite numbers are detected with probability ≥ 3/4 per round; running `k` rounds reduces the false-positive rate to `4^(−k)`. Deterministic primality testing exists (AKS, 2002) [verify] but Miller-Rabin remains the production choice because its constants are small and `k = 40` rounds gives error rate `≤ 2^(−80)`.

**Treap, randomized BST.** Randomized variants of binary search trees that maintain balance probabilistically rather than via explicit rotations.

## Cryptographic vs algorithmic randomness — the high-stakes distinction

This section is short and high-stakes per the chapter's design. The distinction is consequential and routinely confused.

**Algorithmic randomness.** A pseudo-random number generator (PRNG) seeded with a fixed seed produces a deterministic sequence of bits that *looks* random for statistical purposes. Most language standard libraries provide such PRNGs (Mersenne Twister in Python and many others; xorshift, PCG, etc.). Adequate for randomized algorithms, simulation, sampling.

*Not adequate for cryptography.* Given enough output, the PRNG's seed can be reverse-engineered, and the entire future and past sequence becomes predictable.

**Cryptographic randomness.** A cryptographically secure PRNG (CSPRNG) — `/dev/urandom` on Linux, `BCryptGenRandom` on Windows, `SecureRandom` in Java, `secrets` module in Python — produces output that is computationally indistinguishable from true randomness, even given partial output. Required for keys, nonces, tokens, and any setting where an adversary has access to the algorithm's output and could exploit predictability.

The mistake to never make: using `Math.random()` or `random.random()` for security-sensitive purposes. The mistake destroys the security of the system. Authentication tokens generated with non-cryptographic PRNGs have been broken; production breaches have been traced to this exact failure [verify].

The corrective: when in doubt about a randomness use, ask whether an adversary having access to outputs could exploit predictability. If yes, use a cryptographic source. If no — internal sampling, simulation, randomized algorithm pivots — non-cryptographic is fine and faster.

## Decision rules

| Situation | Approach |
| --- | --- |
| Adversarial input would break a deterministic algorithm | Randomize the algorithm |
| Need always-correct, time may vary | Las Vegas |
| Need always-fast, may be wrong | Monte Carlo (with retry/amplification) |
| Need to bound expected time | Linearity of expectation |
| Need to bound probability of large deviation | Chernoff (independent), Chebyshev (variance), Markov (only non-negativity) |
| `k`-th order statistic | Randomized quickselect |
| Min-cut in a graph | Karger (or Stoer-Wagner deterministic for small graphs) |
| Random sample from a stream | Reservoir sampling |
| Approximate nearest neighbor in high dimensions | LSH |
| Fast set membership, false positives OK | Bloom filter (Chapter 3) |
| Primality test on large numbers | Miller-Rabin |
| Building a randomness-balanced BST | Treap or skip list |
| Cryptographic key, token, nonce | CSPRNG (NEVER `Math.random()`) |
| Simulation, sampling, algorithm pivot | Standard PRNG (Mersenne Twister, etc.) |

## Worked example — Karger's min-cut algorithm

The minimum cut of a graph is the smallest set of edges whose removal disconnects the graph. Deterministic algorithms exist (Stoer-Wagner runs in `O(V·E + V² log V)`), but Karger (1993) [verify] gave a strikingly simple randomized algorithm.

**The algorithm.** Repeatedly pick a random edge and "contract" it — merge its two endpoints into a single vertex, removing self-loops, retaining parallel edges. Stop when only 2 vertices remain. The edges between those 2 vertices form a candidate cut. Output it.

```
while |V| > 2:
    pick a uniformly random edge (u, v)
    contract (u, v): merge u and v, drop self-loops, keep parallel edges
return remaining edges as the cut
```

**The surprise.** This works. The algorithm finds the *minimum* cut with probability at least `1 / C(n, 2) = 2 / (n(n-1))` per run — roughly `2/n²`.

**The analysis (linearity-of-expectation flavor).** Fix any minimum cut `C` with `k` edges. The algorithm fails to find `C` only if some edge of `C` is contracted. At each contraction step, all remaining edges are equally likely to be picked. The minimum cut has `k` edges out of `m` total at the start; the probability that the first contracted edge is in `C` is at most `k / m`. A counting argument (every cut has at most `k` edges, every vertex has degree at least `k`, so `m ≥ kn/2`) gives `k/m ≤ 2/n`. Across `n−2` contractions, the probability `C` survives is bounded below by a product that simplifies to `2 / (n(n-1))`.

**Amplification.** A single run finds the min-cut with probability `≥ 2/n²`. Run the algorithm `T = n²/2` times and take the smallest cut found across runs; the failure probability drops to roughly `(1 − 2/n²)^(n²/2) ≈ 1/e ≈ 0.37`. Run `T = n² log n` times; failure probability is `1/n`. Run `T = n²` times; failure probability is essentially zero. Total time: `O(n² · contraction-time)` per run × `O(n² log n)` runs = `O(n^4 log n)` total [verify], beaten by the Karger-Stein refinement (1996) [verify] at `O(n² log³ n)`.

**Why this is striking.** The algorithm makes no use of any structural property of the graph — no minimum-cut characterization, no flow theory. It just contracts random edges. The fact that this finds the min-cut with non-trivial probability is the kind of result that changes how you think about algorithms.

**Production use.** Karger's is rarely the production min-cut algorithm because Stoer-Wagner has better deterministic bounds. But the *technique* — random contraction — has applications in cluster analysis, image segmentation pre-processing, and randomized approximation. The conceptual lesson exceeds the specific algorithm.

**Why this teaches randomization.** The algorithm illustrates the four moves of randomized analysis: a simple algorithm based on random choices, an expected-correctness bound, amplification by repetition, and a concentration argument that justifies "high probability" outcomes. Read it slowly; it is the cleanest example of how randomization works as a technique.

## Failure modes — when "randomness in algorithms is just statistical noise" misleads

The misconception engaged: "Randomness in algorithms is just statistical noise."

This treats randomization as decoration — adding noise to a deterministic procedure, with results that are the same up to small fluctuation. That reading is wrong on multiple counts.

**Randomization can make impossible problems tractable.** Karger's min-cut runs in `O(n^4 log n)` randomized; finding min-cut without randomization is harder both conceptually and historically. The algorithm is not "noisy deterministic"; it is fundamentally different.

**Randomization defeats adversaries.** A deterministic algorithm has predictable behavior; an adversary who knows the algorithm can construct worst-case input. A randomized algorithm has behavior that depends on the algorithm's internal random choices, which the adversary cannot predict. The algorithmic-complexity attack on hash tables (Chapter 3 §6) is the production failure mode of treating randomization as cosmetic.

**Las Vegas algorithms are always correct.** They are not "approximately correct" or "correct on average." They are correct, every time. The randomness lives in the time budget, not the answer.

**Monte Carlo with amplification gives high-confidence answers.** A 0.99-probability algorithm run 10 times in parallel, with majority vote, gives an error probability bounded by tail probabilities far below 0.01. The composition of randomized procedures with bounded error gives stronger guarantees than the misconception suggests.

**Concentration inequalities are tight.** Chernoff bounds are not back-of-envelope estimates; they are mathematical theorems. When the conditions hold, the deviation probability really does decay exponentially. "Probabilistic" does not mean "vague."

**Cryptographic randomness is not interchangeable with algorithmic randomness.** This failure has shipped to production. Rolling a non-cryptographic PRNG for security-sensitive randomness is a class of bug that has caused real breaches. The failure mode is silent — code runs, tests pass, the security property is gone.

The corrective heuristic: state where the randomness comes from, what assumptions about its quality the algorithm needs, and what the correctness or expected-time bound is. Each piece is mathematical, not vague. The algorithm's behavior is precisely characterized; the precision is what makes randomization a tool rather than a hand-wave.

## Cross-references

For randomized quicksort in production-sort context, see Chapter 4. For hash tables and the algorithmic-complexity attack defense, see Chapter 3 §6. For randomized rounding as approximation technique, see Chapter 11 §4. For Karger contraction in graphs, see Chapter 5 §6. For probability theory used here, see the companion-page primer.

## Companion-page handoffs

Probability primer (Markov/Chebyshev/Chernoff with worked examples); Karger-Stein min-cut walkthrough; LSH demo with high-dimensional nearest-neighbor benchmarks; Miller-Rabin primality test implementation; reservoir sampling examples; treap and skip-list implementations; cryptographic vs algorithmic randomness reading list with security-incident case studies. Available at bearbrown.co/algorithms-by-bear-vol1/chapter-12.

## What this chapter does not enable

This chapter does not give full coverage of derandomization — the art of removing randomness from a randomized algorithm without losing the time bound. The method of conditional probabilities, pairwise-independent sample spaces, expander graphs, and the Nisan-Wigderson generator are research-level topics; for an introduction, see Motwani and Raghavan's *Randomized Algorithms* or Mitzenmacher and Upfal's *Probability and Computing*. The chapter also does not cover quantum algorithms, which are randomized in a fundamentally different sense and live outside the classical canon.

## Capability statement

You can now determine whether a randomized algorithm offers a useful tradeoff for a problem; choose between Monte Carlo (fixed time, probabilistic correctness) and Las Vegas (certain correctness, probabilistic time); apply linearity of expectation and basic concentration inequalities (Markov, Chebyshev, Chernoff); recognize the canonical randomized algorithms (quicksort, quickselect, Karger, reservoir sampling, LSH, Miller-Rabin); and distinguish algorithmic from cryptographic randomness — which is the high-stakes distinction the chapter asks you to never get wrong.


---

## LLM Exercise — Chapter 12: Randomized Algorithms and Concentration

**Project:** *Algorithms by Bear* Code-Along Toolkit.

**What you're building this chapter:** Three randomized algorithms — Karger's min-cut, randomized quickselect, locality-sensitive hashing — with empirical measurement of the concentration of measure that makes expected-time bounds practically useful.

**Tool:** Claude Code.

**The prompt:**

```
Chapter 12 task in the algorithms-by-bear-toolkit. Bear's signature
worked example is Karger's min-cut — the surprise that random edge
contraction works with a computable success probability per run that
amplification turns into near-certainty. The exercise implements three
canonical randomized algorithms and measures both expected time and
concentration around expectation.

In `chapters/ch12_randomized/`:

1. `implementations.py`:

   - `kargers_min_cut(graph, num_trials=None)` — repeated random
     edge contraction until 2 super-vertices remain. With
     `num_trials = O(n^2 log n)`, the success probability of
     finding *the* min cut is ≥ 1 - 1/n. Default to that many
     trials. Reuse `UnionFind` from Chapter 3 for the contractions.
   - `randomized_quickselect(arr, k)` — find the k-th smallest in
     expected O(n) with a random pivot.
   - `reservoir_sampling(stream, k)` — uniform random sample of
     size k from a stream of unknown length. Returns the reservoir
     after consuming the stream.
   - `lsh_minhash(signature_set, num_hash_functions)` — MinHash
     signature for Jaccard similarity estimation.
   - `miller_rabin(n, num_rounds=10)` — primality test (Monte
     Carlo, one-sided error).

2. `test_implementations.py`:

   - Quickselect correctness against `sorted(arr)[k]`.
   - Reservoir sampling distributional check — average over many
     stream replays approaches uniform.
   - MinHash similarity estimate convergence to exact Jaccard.
   - Miller-Rabin correctness against `sympy.isprime` for the
     first 10000 integers.
   - Karger correctness: on small graphs (≤ 20 nodes) where the
     true min cut is computable, the success probability per run
     plus enough trials produces the true min cut with probability
     ≥ 99%.

3. `benchmarks.py` — the concentration study. For each algorithm:

   - Run it `N = 1000` times on the same input.
   - Plot the distribution of outcomes (time or quality).
   - Compute the empirical mean and variance.
   - For quickselect: verify the empirical mean time matches
     O(n) and the worst-case observed time is bounded.
   - For Karger: verify success probability per single trial
     matches the theoretical 1/C(n,2) lower bound. Plot success
     probability against number of trials. Find the empirical
     number of trials needed for 99% success on graphs of size
     50, 100, 200.
   - For Miller-Rabin: confirm zero false positives across
     `N` calls on confirmed-prime inputs.
   - For LSH: vary `num_hash_functions` and measure estimation
     variance for Jaccard similarity.

4. `README.md` — decision cards. "Surprising findings": the
   empirical concentration of `randomized_quickselect` (how close
   to expected-O(n) the worst observed time was), and the number
   of Karger trials needed in practice on your test graphs.

Commit with `ch12: randomized algorithms with concentration study`.
```

**What this produces:** Five randomized algorithms, distributional plots showing concentration, and the per-trial Karger success probability measured against the theoretical lower bound. The Karger study is the most surprising — watching random contraction actually work is the chapter's pedagogical payoff.

**How to adapt this prompt:**

- *For your own project:* If you work with near-duplicate detection (search, fraud, content moderation), expand the LSH study to your real data.
- *For ChatGPT / Gemini:* Karger's contraction logic is fiddly with multigraphs; ask either model to write the contraction step against a `UnionFind` substrate and test on a 5-node graph by hand before scaling.
- *For Claude Code:* Native fit. Run 1000 trials per algorithm — it's fast enough to be illuminating.
- *For a Claude Project:* Skip.

**Connection to previous chapters:** Reuses `UnionFind` from Chapter 3; the quickselect is the randomized counterpart to the quicksort from Chapter 4.

**Preview of next chapter:** Chapter 13 closes the optimization arc with linear programming — formulating a real planning problem, calling a solver, interpreting the dual, and using LP relaxation as an approximation tool that ties back to the work in Chapter 11.
