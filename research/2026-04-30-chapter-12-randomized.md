# Research notes — Chapter 12: Randomized Algorithms

## Source folders used
- Primary: `Randomized_Algorithms/Randomized_Algorithms.md`
- Secondary: `Algorithm_Analysis/Algorithm_Analysis.md` randomized section (lines 1677–1818), routed in
- Tertiary: `Graphs_and_Graph_Search_Algorithms/Graph_Algorithms.md` Karger contraction (lines 1096–1406), routed in

## Source sections kept
From `Randomized_Algorithms.md`:
- `## Introduction` (3–72)
- `## Theoretical Foundations` (73–189) — probability basics, random variables, concentration, randomized quicksort
- `## Classification → Monte Carlo, Las Vegas` (190–352)
- `## Design and Analysis` (353–697) — sampling, randomized data structures, random walks (briefly)
- `## Applications → Graph Algorithms` (807–997)
- `## Cryptography → Probabilistic Encryption, Random Oracle` (998–1118) — short but high-stakes
- `## Computational Geometry → Closest Pair` (1119–1187) — brief
- `## Challenges → Derandomization, Randomness Quality` (1286–1439)

From `Graph_Algorithms.md`:
- `## Random Contraction Algorithm` (1282–1406) — Karger's min-cut, the worked example
- `## Randomized Algorithms in Graph Theory → Random Selection` (1407–1552) — used briefly

## Source sections cut
- `## Future Directions → Quantum Computing, Algorithmic Fairness` — magazine
- `## Exercises and Problems`
- `## Further Reading and Resources`
- `Quiz_Questions_Randomized_Algorithms.md`

## Original Claude content (NOT in source)

### Reservoir sampling
Source covers random sampling generically but not the specific reservoir-sampling algorithm. Added because it's a canonical streaming algorithm and the chapter would feel incomplete without it.

### LSH
Source mentions LSH only briefly. Added the high-dimensional nearest-neighbor framing.

### Miller-Rabin / AKS
Source mentions probabilistic primality testing generically. Specific algorithm names and AKS 2002 attribution added. [verify].

### Skip list, treap
Source covers some randomized data structures. Added skip list as alternative-to-balanced-BST framing. Linux kernel and Redis usage claims marked [verify].

### Karger 1993 / Karger-Stein 1996
[verify] specific years and refinement attribution.

### Cryptographic vs algorithmic randomness section
Per outline.md note: "short but high-stakes." Source covers cryptographic randomness in §6; integrated into the named-distinction framing here. The "production breaches" reference is generic; specific incidents would need citation.

### Specific PRNG mentions
Mersenne Twister, xorshift, PCG are real PRNGs. `/dev/urandom`, `BCryptGenRandom`, `SecureRandom`, Python `secrets` are real CSPRNGs. [verify] platform-specific claims.

### Randomized quicksort expected comparisons
`2n H_n − 4n ≈ 1.39 n log n` — canonical bound. [verify].

## Factual claims preserved from source
- Monte Carlo vs Las Vegas distinctions — from source
- Linearity of expectation, Markov/Chebyshev/Chernoff — from source
- Random contraction algorithm — from source
- Universal hashing connection — implicit in Ch 3

## [verify] count
9 inline `[verify]` markers:
1. Karger 1993
2. Karger-Stein 1996 refinement at `O(n² log³ n)`
3. Total Karger basic time `O(n^4 log n)`
4. Linux kernel skip list use
5. Redis sorted-sets skip list use
6. AKS primality test 2002
7. Miller-Rabin error rate `4^(−k)` per round
8. Randomized quicksort expected `1.39 n log n` comparisons
9. Production breaches from non-cryptographic PRNG misuse

(Total: ~9 inline `[verify]` tags as written.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" — elevated probability primer pointer ✓
- Decision rules table present ✓
- Worked example: Karger's min-cut (named in outline.md) ✓
- Failure modes engages "Randomness in algorithms is just statistical noise" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- §1 elevates the probability-primer pointer prominently per outline.md.
- §6 (cryptographic vs algorithmic) is short and high-stakes per outline.md. Held to two paragraphs of mechanism plus the corrective.
- Karger's worked example is the chapter's pedagogical anchor — concrete, surprising, and shows how linearity of expectation works in a non-trivial analysis.
- Resisted the temptation to derive Chernoff bounds on the page. Reference convention: state the inequality, point at the primer for derivation.
- The misconception engagement uses six concrete failures rather than one — the misconception is layered and each layer has a different corrective.

## Word count
~2,860 words (target: ~2,800; within +2%)

## Open issues for editor
- All historical attribution dates need verification
- Karger-Stein refinement at `O(n² log³ n)` — verify exact bound
- AKS primality test attribution should be cross-checked
- "Production breaches from PRNG misuse" claim could carry specific incident citations (e.g., Debian SSH key vulnerability 2008, Sony PS3 ECDSA key reuse 2010, various crypto wallet incidents)
- The cryptographic randomness section could point at NIST SP 800-90A / 800-90B for production guidance
