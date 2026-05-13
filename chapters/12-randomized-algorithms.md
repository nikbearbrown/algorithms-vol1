# Chapter 12 — Randomized Algorithms

*The flip of a coin, taken seriously, turns out to be one of the most powerful tools in all of algorithm design.*

---

Here is an algorithm for finding the minimum cut of a graph — the smallest set of edges whose removal disconnects it.

Pick a random edge. Merge its two endpoints into one vertex. Throw away any self-loops. Repeat until only two vertices remain. The edges between them are your cut. Output them.

That's it. That's the whole algorithm. It fits in four lines of code. The only thing it does deliberately is pick edges randomly. It has no clever data structures. It does not look at the structure of the graph. It does not reason about which edges matter. It just contracts random edges until nothing is left.

And it finds the minimum cut.

Not every time — that's the part worth sitting with. It finds the minimum cut with probability at least $2/n^2$, where $n$ is the number of vertices. On a 100-vertex graph, that's a 0.02% chance per run. Run it often enough, and you find the minimum cut with any probability you want.

The fact that this works at all is the kind of result that should make you stop and think about what randomness is actually doing in an algorithm.

---

## Two kinds of randomized algorithms

Before the analysis, a distinction that matters.

A **Las Vegas** algorithm uses randomness but always produces the correct answer. The running time is a random variable, but correctness is certain. Randomized quicksort is Las Vegas: whatever pivot it picks, however the recursion plays out, the array ends up sorted. You might wait longer or shorter depending on random choices, but you never get the wrong answer.

A **Monte Carlo** algorithm uses randomness and always finishes in bounded time, but may produce an incorrect answer with bounded probability. Karger's min-cut is Monte Carlo: it always finishes in $O(n^2)$ time, but it might output a cut that isn't minimum. The error probability is controlled — and small — but it's there.

These are duals. To turn a Monte Carlo algorithm into a Las Vegas algorithm, run it repeatedly until you get an answer you can verify. To turn a Las Vegas algorithm into a Monte Carlo algorithm, cut it off at a time bound and return whatever you have. The choice between them depends on whether you can cheaply verify a correct answer.

The important thing: Las Vegas algorithms are *always correct*. The word "randomized" does not mean "approximately correct" or "probably correct on most inputs." A Las Vegas algorithm is as correct as any deterministic one — the randomness lives in the time budget, not the answer.

<!-- → INFOGRAPHIC: two-column comparison card — left column "Las Vegas": fixed correctness, variable time, example = randomized quicksort; right column "Monte Carlo": fixed time, variable correctness, example = Karger's min-cut; each column includes a one-line summary of the dual conversion ("run until verified" vs. "truncate at time bound"); makes the classification scannable without re-reading the prose -->

---

## Why Karger's algorithm works

Fix a minimum cut $C$ — any minimum cut, with $k$ edges. Karger's algorithm fails to find $C$ if and only if it contracts some edge in $C$ at some point during the $n-2$ contractions. So ask: what is the probability that $C$ survives all the way to the end?

The key observation is a counting argument. If the minimum cut has $k$ edges, then every vertex must have degree at least $k$ — otherwise, the edges incident to a low-degree vertex would form a cut smaller than the minimum, which is a contradiction. If every vertex has degree at least $k$ and there are $n$ vertices, the total degree is at least $kn$. Since the total degree is twice the number of edges, there are at least $kn/2$ edges in the graph.

At the first contraction step, the probability of picking an edge in $C$ is at most $k$ divided by the number of edges, which is at most $k / (kn/2) = 2/n$.

So the probability that the first step does *not* contract an edge in $C$ is at least $1 - 2/n$.

After the first contraction, there are $n-1$ vertices, and the same argument applies. The minimum cut still has $k$ edges (you didn't touch any of them), and the graph still has at least $k(n-1)/2$ edges. The probability the second step avoids $C$ is at least $1 - 2/(n-1)$.

Multiply these probabilities across all $n-2$ contractions:

$$P(\text{C survives}) \geq \prod_{i=0}^{n-3} \left(1 - \frac{2}{n-i}\right) = \frac{2}{n(n-1)}$$

<!-- → CHART: bar chart or step plot showing the per-step survival probability at each of the n-2 contraction steps — x-axis is contraction step number (1 to n-2), y-axis is the conditional probability of not contracting a min-cut edge at that step; the bars decrease from (1 - 2/n) toward (1 - 2/3) as the graph shrinks; the product of all bars is the total survival probability; student should see that the final steps are the most dangerous, not the first -->

For $n = 100$, this is about $0.0002$. Not great for a single run. But each run is independent, and the probability structure is precise. Run the algorithm $T$ times and take the smallest cut found. The probability that $C$ is never found across $T$ trials is at most:

$$\left(1 - \frac{2}{n(n-1)}\right)^T$$

Set $T = n^2 \ln n$ and the failure probability drops below $1/n$.

<!-- → CHART: line chart showing failure probability (y-axis, log scale) vs. number of trials T (x-axis) for n = 20, 50, 100 — three curves, each showing exponential decay; vertical reference lines at T = n^2/2 and T = n^2 ln n; student should see how quickly amplification reduces the per-trial ~2/n^2 success rate to near-certainty --> The algorithm now finds the minimum cut with high probability, in $O(n^4 \log n)$ total work. The Karger-Stein refinement brings this to $O(n^2 \log^3 n)$ by being smarter about when to branch the recursion — but the conceptual payoff is already here.

What makes this striking is not the bound, which is modest, but the method. The algorithm has no idea what the minimum cut is. It picks edges at random. It doesn't reason about structure. It just contracts, and the combinatorics work out such that any specific minimum cut has a guaranteed nonzero survival probability per run. Amplification does the rest.

---

## The two analytical tools

Karger's analysis used a counting argument, but two tools handle the broad sweep of randomized analysis.

**Linearity of expectation** is the workhorse. The expected value of a sum of random variables equals the sum of their expected values — regardless of whether the variables are independent. This seems modest. In practice it's powerful because it lets you decompose complicated random quantities into simple parts.

Here is the classic application: how many comparisons does randomized quicksort make, in expectation?

Instead of tracking the recursion, track each pair of elements. Fix elements $a_i$ and $a_j$ where $i < j$ in sorted order. They are compared if and only if one of them is chosen as a pivot before any element between them is chosen as a pivot. Under uniform random pivot selection, each of the $j - i + 1$ elements is equally likely to be the first among them selected. So $P(\text{$a_i$ and $a_j$ compared}) = 2/(j - i + 1)$.

The expected total number of comparisons is:

$$E[\text{comparisons}] = \sum_{i < j} \frac{2}{j - i + 1} = 2 \sum_{k=1}^{n-1} \frac{n-k}{k+1} \approx 2n \ln n$$

The key step was linearity of expectation: the expected total is the sum of expected contributions from each pair. No independence required.

**Concentration inequalities** tell you how tightly a random variable clusters around its expectation. You need three, in increasing strength.

*Markov's inequality* says that for any non-negative random variable $X$ and any $t > 0$:

$$P(X \geq t) \leq \frac{E[X]}{t}$$

If the expected value is small, large deviations are unlikely. This is very crude — it only uses the mean — but it applies to anything non-negative.

*Chebyshev's inequality* uses the variance. If $X$ has mean $\mu$ and variance $\sigma^2$:

$$P(|X - \mu| \geq k\sigma) \leq \frac{1}{k^2}$$

Twice the standard deviation away from the mean: at most 25% probability. Three standard deviations: at most 11%. Stronger than Markov when variance is bounded.

*Chernoff bounds* are the sharp tool. For a sum $X = X_1 + X_2 + \cdots + X_n$ of independent zero-one random variables with mean $\mu = E[X]$:

$$P(X \geq (1 + \delta)\mu) \leq e^{-\mu \delta^2 / 3}$$

The tail decays *exponentially* in the deviation. This is what "high probability" means in algorithms — not "fairly likely" but "probability falling off like $e^{-n}$." When the conditions hold, Chernoff is tight and the guarantee is strong.

The practical rule: use Chernoff when summing independent indicators; use Chebyshev when you have variance but not independence; use Markov when all you have is non-negativity.

<!-- → TABLE: three-row reference card for concentration inequalities — columns: "Inequality", "What you need to know about X", "What it bounds", "Tail decay", "When to use"; rows: Markov (only mean, one-sided, 1/t, always applicable), Chebyshev (mean + variance, two-sided, 1/k², variance bounded), Chernoff (independent indicators + mean, one-sided upper, exponential, sum of indicators); makes the selection rule scannable -->

---

## What randomization actually buys you

There are four distinct reasons to reach for a randomized algorithm. Confusing them is the source of most confusion about what randomization is.

**Defeating adversarial input.** A deterministic algorithm has predictable behavior — any adversary who knows the algorithm can construct input that drives it to worst case. Randomization removes this vulnerability. Quicksort with a fixed pivot strategy runs in $O(n^2)$ on sorted input; an adversary who knows your strategy can always hand you sorted input. Quicksort with a random pivot runs in expected $O(n \log n)$ *regardless of the input* — the adversary cannot control your random choices. The randomness is a defense mechanism.

This is the same reason that hash functions in production systems use per-process random seeds. Without them, an adversary who knows the hash function can craft inputs that all collide, turning every $O(1)$ lookup into $O(n)$. The defense is randomness the adversary cannot observe.

**Getting simple algorithms.** Karger's min-cut is four lines. The deterministic Stoer-Wagner min-cut algorithm is substantially more involved. Randomized quickselect — find the $k$-th smallest element in expected $O(n)$ time — is a few lines. The deterministic median-of-medians that achieves worst-case $O(n)$ requires careful construction and larger constants.

Often the simplest correct algorithm for a problem is randomized. This is not incidental. Deterministic algorithms frequently need to work around adversarial inputs by being clever about structure. Randomized algorithms can ignore structure entirely and let probability do the work.

**Expected-time bounds that beat worst-case.** Randomized quicksort runs in expected $O(n \log n)$. No comparison sort can do better in the worst case — the $\Omega(n \log n)$ lower bound applies to all deterministic comparison sorts. But "expected $O(n \log n)$" under random pivots with good concentration is often better in practice than $O(n \log n)$ worst-case algorithms with larger constants.

The distinction between average-case and expected-case matters. Average-case analysis assumes something about the input distribution. Expected-case (Las Vegas) analysis holds *for all inputs* — the expectation is over the algorithm's random choices, not over the inputs. Randomized quicksort's expected $O(n \log n)$ bound holds for sorted input, reverse-sorted input, all-equal input, adversarial input. The randomness is internal.

**Intrinsically probabilistic problems.** Some problems are naturally framed probabilistically: approximate counting, nearest-neighbor search in high dimensions, streaming algorithms where you cannot store the whole input. These call for randomization not as a workaround but as the right frame from the start.

---

## A handful of canonical algorithms

**Randomized quickselect.** Finding the $k$-th smallest element in an array. Pick a random pivot, partition the array, recurse on the side containing position $k$. Expected $O(n)$ time by linearity of expectation applied to the expected work at each level. The deterministic median-of-medians achieves worst-case $O(n)$ with more complicated pivot selection; quickselect is faster in practice and simpler to implement.

**Reservoir sampling.** You have a stream of $n$ items, $n$ unknown in advance, and want to draw a uniform random sample of $k$ of them using $O(k)$ memory. Algorithm: keep the first $k$ items. For the $i$-th item (when $i > k$), replace a uniformly random currently-held item with probability $k/i$. By induction, after processing the first $i$ items, each of them is in the reservoir with probability exactly $k/i$. When the stream ends, the reservoir is a uniform random $k$-sample. This is used everywhere streaming data is sampled: analytics pipelines, database query optimizers, online learning.

**Locality-sensitive hashing.** Nearest-neighbor search in high dimensions is hard for deterministic tree-based methods because of the curse of dimensionality — all points become equidistant and trees lose their structure. LSH uses hash functions that map nearby points to the same bucket with high probability and far-away points to different buckets with high probability. You build an index of hash buckets; at query time, you hash the query and examine only the points in its bucket. Sub-linear query time at the cost of bounded approximation error. The method that powers similarity search in large-scale recommendation systems, document near-duplicate detection, and image retrieval.

**Miller-Rabin primality testing.** Is this large number prime? The deterministic AKS algorithm (2002) runs in polynomial time, but Miller-Rabin has been the production choice for decades. It is Monte Carlo: each round detects a composite with probability at least 3/4. Running 40 independent rounds gives a false-positive probability of at most $4^{-40} \approx 10^{-24}$ — far below any reasonable threshold. You call it on a candidate prime, get a yes/no answer, and can calculate the residual uncertainty precisely.

**Skip lists.** A randomized alternative to balanced binary search trees. Each element is inserted into a hierarchy of linked lists, with each level included with probability 1/2. Expected $O(\log n)$ for search, insert, and delete — the same asymptotic bound as a red-black tree, but with a simpler implementation that doesn't require explicit rebalancing logic. Redis uses skip lists for its sorted set data structure.

---

## The distinction you cannot afford to confuse

One thing in this chapter is high-stakes in a way the rest is not.

Algorithmic randomness and cryptographic randomness are not the same thing.

A standard pseudo-random number generator — Python's `random`, Java's `Math.random()`, the Mersenne Twister — is a deterministic function. It takes a seed and produces a sequence of bits that pass statistical tests for randomness. This is adequate for algorithm pivots, simulations, sampling, and everything in this chapter so far.

It is *not* adequate for anything security-sensitive: generating authentication tokens, producing session keys, creating nonces, picking cryptographic parameters. A standard PRNG is deterministic; given enough output, an adversary can recover the seed and predict all future and past outputs. Systems that generate authentication tokens with non-cryptographic PRNGs have been broken. Real breaches have been traced to exactly this mistake.

A cryptographically secure PRNG — `/dev/urandom` on Linux, `BCryptGenRandom` on Windows, the `secrets` module in Python, `SecureRandom` in Java — produces output that is computationally indistinguishable from true randomness even given partial output. This is what you use for anything where an adversary could exploit predictability.

The test to apply: would an adversary who sees some outputs of this random source gain an advantage? If your answers to that question are "yes" or "maybe" or "I'm not sure," use a cryptographic source. If the answer is "no — these are internal algorithm choices that the adversary has no access to," a standard PRNG is fine and faster.

This is a short section in the chapter but a long section in incident reports. The mistake is silent: the code runs, tests pass, and the security property is gone. Do not confuse the two.

<!-- → TABLE: two-column reference card "Algorithmic vs. Cryptographic Randomness" — columns: "Property", "Algorithmic PRNG", "Cryptographic PRNG"; rows covering: examples (Mersenne Twister / /dev/urandom), predictability (recoverable from output / computationally indistinguishable), speed (fast / slower), use cases (pivots, sampling, simulation / tokens, keys, nonces), failure mode (adversary exploits predictability / N/A); caption should reinforce: when in doubt, use cryptographic -->

---

## What you can do now that you couldn't before

You can classify a randomized algorithm as Monte Carlo or Las Vegas and know what each classification means about correctness and time. You can apply linearity of expectation to analyze expected running time by decomposing into indicator variables. You can choose among Markov, Chebyshev, and Chernoff based on what information you have about the random variable. You can explain why Karger's algorithm finds the minimum cut with probability $2/n(n-1)$ per run and how amplification makes this useful. You can recognize when randomization defends against adversarial input, when it simplifies the algorithm, and when the problem is inherently probabilistic.

And you will never use `Math.random()` to generate an authentication token.

The next chapter closes the optimization arc with linear programming — a framework that generalizes many of what we've built in the book, provides the dual of max-flow min-cut, and gives the LP relaxation technique that ties back to the approximation work in Chapter 11.

---

## Exercises

### Warm-up

**1.** Karger's algorithm is run once on a graph with $n = 10$ vertices. The minimum cut has $k = 3$ edges.

   - (a) What is the probability that the minimum cut survives all 8 contractions? Use the product formula and compute the exact value.
   - (b) How many independent trials are needed so that the probability of finding the minimum cut at least once exceeds 99%? Use the bound $T \geq n^2 \ln n / 2$ as your estimate, then check it: compute $(1 - 2/n(n-1))^T$ for your chosen $T$ and verify the failure probability is below 0.01.

*(Tests: mechanical application of the Karger survival probability and amplification formulas.)*

**2.** Classify each of the following as Las Vegas or Monte Carlo. For each, state what is fixed (time or correctness) and what is random.

   - (a) Randomized quicksort
   - (b) Karger's min-cut (single run)
   - (c) Miller-Rabin primality test with 40 rounds
   - (d) Reservoir sampling
   - (e) An algorithm that runs Miller-Rabin until it gets two agreeing "prime" answers, then outputs "prime"

*(Tests: Las Vegas vs. Monte Carlo classification; recognizing that amplification can convert one form to the other.)*

**3.** Apply linearity of expectation to compute the expected number of comparisons made by randomized quicksort on an array of $n = 5$ elements (with values 1, 2, 3, 4, 5). For each of the $\binom{5}{2} = 10$ pairs $(a_i, a_j)$ with $i < j$, state the probability they are compared and compute the total expected comparisons. Then compare to the $2n \ln n$ approximation.
*(Tests: linearity of expectation applied concretely; indicator variable decomposition on a small enough case to trace by hand.)*

---

### Application

**4.** A deterministic quicksort always uses the first element as pivot. An adversary knows this.

   - (a) Construct the worst-case input for this strategy on an array of 6 elements (hint: already-sorted input is one; what invariant does it satisfy?).
   - (b) A randomized pivot is chosen uniformly at random. Explain precisely why an adversary who knows the algorithm — but not the random seed — cannot construct a worst-case input in the same way.
   - (c) Randomized quicksort still has a worst case of $O(n^2)$. Why is this not a contradiction of the adversary argument?

*(Tests: the adversarial input defense; understanding that "expected $O(n \log n)$ for all inputs" is different from "worst-case $O(n \log n)$".)*

**5.** You are using reservoir sampling to maintain a uniform random sample of $k = 5$ items from a data stream. The stream so far has delivered 20 items, and the reservoir contains items at positions 3, 7, 11, 14, 19. The 21st item now arrives.

   - (a) What is the probability that the 21st item is included in the reservoir?
   - (b) If it is included, which item does it replace and with what probability?
   - (c) After the replacement (if any), what is the probability that any specific item from the first 21 is in the final reservoir?

*(Tests: reservoir sampling mechanics; the invariant that each of the first $i$ items is in the reservoir with probability $k/i$.)*

**6.** Miller-Rabin primality test with $r$ rounds has a false-positive rate of at most $4^{-r}$ per composite number tested.

   - (a) You need to generate 10,000 prime candidates for a cryptographic application. You run 20 rounds per candidate. What is an upper bound on the expected number of composites incorrectly declared prime?
   - (b) Is this algorithm Las Vegas or Monte Carlo? Explain.
   - (c) Why does the production community use Miller-Rabin rather than the deterministic AKS primality test, even though AKS is provably correct?

*(Tests: Monte Carlo error analysis and amplification; the practical trade-off between bounded-error probabilistic and deterministic algorithms.)*

---

### Synthesis

**7.** The chapter claims that "average-case analysis" and "expected-case (Las Vegas) analysis" are different claims about an algorithm's performance. Explain the distinction precisely using randomized quicksort as the example. Specifically: what does "average-case $O(n \log n)$" mean, what does "expected $O(n \log n)$" mean, and which is a stronger guarantee? For which kinds of inputs does the stronger guarantee matter?
*(Tests: the average-case vs. expected-case distinction; understanding that expected-case bounds hold for all inputs, not just "typical" ones.)*

**8.** You want to find the minimum cut of a graph with $n = 50$ vertices. You have two options: Karger's algorithm with $T = n^2 \ln n$ trials, or Stoer-Wagner (a deterministic algorithm running in $O(VE + V^2 \log V)$).

   - (a) Estimate the number of Karger trials. How does this compare to one Stoer-Wagner run?
   - (b) On what grounds would you choose Karger over Stoer-Wagner for a production system? On what grounds Stoer-Wagner over Karger?
   - (c) The chapter says "the technique — random contraction — has applications beyond the specific algorithm." Name one application domain where Karger's contraction idea generalizes beyond exact min-cut, and explain why the probabilistic structure transfers.

*(Tests: algorithmic trade-off reasoning; recognizing that the value of a randomized technique can exceed the specific algorithm it was designed for.)*

---

### Challenge

**9.** The Chernoff bound applies to sums of *independent* zero-one random variables. Karger's analysis used a *product* of conditional probabilities instead. Explain why Chernoff cannot be directly applied to bound the probability that Karger's algorithm fails, and why the product-of-conditionals argument is the right structure instead. (Hint: are the indicator variables "does contraction $i$ hit the min-cut?" independent across steps?)
*(Tests: understanding when Chernoff applies and when it doesn't; the structural reason Karger's analysis takes a different form.)*

**10.** Design a randomized streaming algorithm for the following problem, or prove it cannot be solved in sub-linear space.

   *Majority element.* A stream of $n$ items arrives one at a time. You must determine, at the end of the stream, whether any item appears more than $n/2$ times. If yes, output it. If no, output "none." You may use $O(\log n)$ memory. You may err with probability at most $1/3$.

   Boyer-Moore majority vote is a deterministic $O(1)$-space algorithm but requires two passes. Design a single-pass randomized algorithm using $O(\log n)$ space. State the algorithm, analyze its error probability, and explain which concentration inequality governs its correctness guarantee.

*(Tests: applying randomized algorithm design to a streaming problem; connecting concentration inequalities to a specific algorithm's correctness argument; open-ended design with evaluation criteria.)*

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
