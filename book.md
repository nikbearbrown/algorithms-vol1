# Algorithms by Bear, Vol. 1

A 13-chapter Kindle reference handbook for the classical algorithms canon, written for working software engineers who consult the book by lookup rather than reading sequentially. Volume 1 of the *Algorithms by Bear* series. Published by Bear Brown LLC, edited by Nik Bear Brown.

## Identity

**Type:** Practitioner handbook / reference. NOT a course textbook, NOT a theoretical CS reference, NOT competitive programming preparation, NOT a popular-audience book.

**Distribution:** Kindle at $0.99 plus Kindle Unlimited, with promotional free-distribution windows.

**Edition cycle:** Snapshot editions every 4–5 years, with errata pushes via KDP between editions, and a live companion page that grows continuously.

## Reason for existing

> *Algorithms by Bear, Vol. 1* is the practitioner-shaped, Kindle-priced, plain-language reference for the classical algorithms canon — for the working developer who wants the canonical content of CLRS without the textbook overhead and the price tag, written for fast lookup rather than sequential study.

The book exists because no current, maintained, Kindle-priced reference fills this slot. *Algorithms in a Nutshell* (O'Reilly) was the closest match and has been out of print since 2015. CLRS is academic; Sedgewick is Java-anchored and textbook-priced; Skiena is folksy and uneven; *Algorithm Design Manual* is interview-prep-shaped. Each occupies a real shelf position. This volume occupies the empty slot below them.

## Audience

**Primary reader (Reader A):** Working software engineer, 28+, four years into the role, bootcamp graduate or self-taught CS, no formal algorithms course. Knows how to build production systems but the formal vocabulary of the field has gaps. Pays $0.99 to have the book on every device, or reads it on Kindle Unlimited.

**Secondary readers:**
- Career-switcher self-learner (Reader B): former biologist, ML engineer, or quant filling in CS theory gaps.
- CS student (Reader C): undergraduate using the book as the cheap, plain-language secondary reference alongside an assigned CLRS or Kleinberg-Tardos textbook.

**Non-targets:** Course adopters as a primary text. Theoretical CS students. Competitive programmers. Non-technical popular readers.

## Architecture

13 chapters in four topic clusters. Foundations first, then methods, then optimization, then the probabilistic outlier:

- **Foundations** (1–3): Introduction, Algorithm Analysis, Data Structures.
- **Core Methods** (4–8): Sorting and Caching, Graphs and Graph Search, Greedy, Divide and Conquer, Dynamic Programming.
- **Optimization** (9–11, 13): Network Flow, NP-Completeness and Intractability, Approximation Algorithms, Linear Programming.
- **Probabilistic** (12): Randomized Algorithms.

Linear Programming sits at chapter 13, after Randomized Algorithms, as the capstone optimization method.

## Series architecture

This book is one of three vehicles in the *Algorithms by Bear* line:

1. **Algorithms by Bear, Vol. 1** (this book) — classical canon, 4–5 year refresh.
2. **Algorithms by Bear, Vol. 2** — decision-making algorithms (Bayesian methods, optimal stopping, game theory, scheduling, social networks); 2-year refresh.
3. **[Magazine Name]** — quarterly deep-dive serial covering modern AI methods (RL, LLMs, generative models) and student-cohort case studies.

The companion page indexes magazine issues by chapter, hosts the errata page per edition, and provides extended code, benchmarks, and visualizations.

## Preface — explicit scope statement

> Modern machine learning algorithms — gradient descent, attention mechanisms, transformer architectures — are out of scope for this classical reference and are covered in the *Algorithms by Bear* magazine series.

The preface also names the load-bearing chapters for sequential readers — Chapter 2 (Algorithm Analysis) and Chapter 3 (Data Structures) — and recommends them first for readers who want to use the rest confidently.

## Implicit arguments

The book carries five implicit arguments through structure and emphasis. None are stated as theses; all should be visible in the chapter shapes:

1. Decision rules matter as much as algorithms.
2. Asymptotic analysis is necessary but not sufficient.
3. Worked examples should be real, not pedagogical.
4. Failure modes are first-class content.
5. The classical canon is durable.