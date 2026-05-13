# Research notes — Chapter 07: Divide and Conquer

## Source folders used
- Primary: `Divide_and_Conquer_Strategies/Divide_and_Conquer_Algorithms.md`
- Secondary: `Algorithm_Analysis/Algorithm_Analysis.md` D&C section (lines 1261–1430), routed in

## Source sections kept
- `## Introduction to Divide and Conquer` (3–298)
- `## Sorting and Selection → Merge Sort, Quickselect` (299–736) — brief; merge sort full coverage in Ch 4
- `## Integer and Polynomial Multiplication → Karatsuba` (737–1204)
- `## Matrix Multiplication → Strassen` (1205–1495)
- `## Counting Inversions, Closest Pair` (1496–1744) — closest pair is worked example
- `## Quicksort` (1745–1896) — brief; full coverage in Ch 4
- `## Comparative Analysis` (1897–1986)

## Source sections cut
- `## Advanced Topics → Parallelization, Distributed Computing, Recent Advances` (1987–2068)
- `## Practical Applications → Modern Software Development` (vague)
- `## Challenges and Future Directions`
- `## Conclusion`

## Original Claude content (NOT in source)

### FFT (Cooley-Tukey 1965)
Source mentions FFT only in passing or not at all. Added because FFT is the canonical large-scale D&C algorithm and a reference book of D&C algorithms cannot omit it. [verify] Cooley-Tukey 1965 attribution.

### Strassen practical crossover
Source describes Strassen but does not quantify when standard schoolbook wins. Added "below n ≈ 64 to 128" — widely-canonical crossover range. [verify] specific BLAS implementation behavior.

### Closest-pair geometric bound (7 points per 2δ × δ box)
Source describes the strip scan but does not derive the constant. Added: "at most 7, sometimes stated as 6." [verify] The exact constant depends on the specific packing argument used in the proof; classical analyses give 7 or 8.

### Worked example timing numbers
"5 × 10¹¹ for n=10⁶ brute force" and "2 × 10⁷ for D&C" are illustrative. [verify].

## Factual claims preserved from source
- Closest-pair D&C structure (divide along x-median, conquer, strip-scan combine) — from source
- Karatsuba algorithm structure and `n^(log_2 3)` bound — from source
- Strassen algorithm structure and `n^(log_2 7)` bound — from source
- Counting inversions via modified merge sort — from source
- Quicksort and quickselect — from source

## [verify] count
6 inline `[verify]` markers:
1. Karatsuba 1960
2. Strassen 1969
3. Cooley-Tukey 1965
4. Strassen crossover at n ≈ 64 to 128
5. Closest-pair box bound (7 points)
6. Brute-force timing 5 × 10¹¹ for n=10⁶
7. D&C timing 2 × 10⁷ for n=10⁶

(Total: 7 actual inline `[verify]` tags.)

## Structure-drift checks
- Section 1 titled "Recognition pattern" ✓
- Section 2 titled "What you need to know first" ✓
- Decision rules table present ✓
- Worked example: closest pair of points in 2D (named in outline.md) ✓
- Failure modes engages "D&C is just recursion" ✓
- Anti-capability paragraph present ✓
- Capability statement closes the chapter ✓
- No exercises ✓
- No learning objectives ✓
- No chapter-opening hook ✓

## Voice-anchoring notes
- The "three signals → D&C applies" diagnostic move in §1 is the chapter's reference move. Held.
- The recurrence framing `T(n) = aT(n/b) + f(n)` carries through the chapter as the analytic spine.
- Resisted teaching FFT and Strassen at depth — both deferred to companion page per outline note.
- The contrast with DP (Chapter 8) is set up here; DP chapter must complete the diagnostic by showing overlapping subproblems explicitly.

## Word count
~2,650 words (target: ~2,600; within +2%)

## Open issues for editor
- All historical attribution dates need verification
- Closest-pair box constant (7 or 6 or 8) varies by proof; pick one and stick with it; current draft says "7, sometimes 6"
- Strassen practical crossover should be verified against actual BLAS library behavior
- Worked-example timing numbers are illustrative; consider replacing with figures from a real benchmark if available
