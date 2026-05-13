# Introduction to Algorithms

## TL;DR

*Algorithms by Bear, Vol. 1* is a Kindle-priced reference handbook covering the classical algorithms canon — analysis, data structures, sorting, graphs, greedy, divide-and-conquer, dynamic programming, network flow, NP-completeness, approximation, randomized algorithms, and linear programming. Reach for this book when you need to identify the right algorithm, recall a complexity bound, or recognize when a problem fits a canonical pattern. After consulting it, you can choose an algorithm, justify the choice, and route past the canonical failure modes.

## When this book applies

You are a working software engineer with a problem in front of you. You suspect there is a named algorithm or analysis technique for it, but you cannot remember which one, or you can name it but cannot remember what it actually does, or you can do both but want to confirm a complexity bound or a failure mode before you ship. This book is for those moments.

The shape of the question matters. *Algorithms by Bear, Vol. 1* answers questions of the form: "Which sort do I use here?" "What does it cost?" "When does this break?" "Is this approach provably optimal or just usually good enough?" It does not answer "How do I implement a red-black tree in C++ from memory?" — your standard library or a textbook with full code listings does that. It does not answer "What is the gradient of attention with respect to query weights?" — the *Algorithms by Bear* magazine and a deep-learning textbook do that.

If the question is decidable from a chapter's decision rules, this book is the right book. If the question requires implementation-ready code, consult the companion page. If the question concerns a method outside the classical canon — Bayesian decision theory, online learning, reinforcement learning, transformers — consult Vol. 2 or the magazine.

The book is not a course textbook and is not a theoretical CS reference. It is a handbook. The chapters are designed for lookup. You are expected to skip around. The sequential dependencies are marked.

## What you need to know first

This book assumes you can read pseudo-code, can run a basic complexity argument, and have written software professionally. No formal algorithms course is assumed. If you have not read Chapter 2 (Algorithm Analysis) or Chapter 3 (Data Structures) yet, read them first. Every later chapter cites Big O, recurrences, heaps, and hash tables; those two chapters carry the vocabulary the rest of the book uses. The recursion refresher and probability primer live on the companion page.

## How the book is shaped

Thirteen chapters in four clusters.

**Foundations (Chapters 1–3).** Introduction. Algorithm Analysis. Data Structures. These chapters establish the vocabulary — Big O, master theorem, amortized analysis, heaps, hash tables, balanced trees, bloom filters — that the rest of the book uses without re-explaining.

**Core methods (Chapters 4–8).** Sorting and Caching. Graphs and Graph Search. Greedy. Divide and Conquer. Dynamic Programming. The five method families that solve most algorithmic problems a working engineer encounters.

**Optimization (Chapters 9–11, 13).** Network Flow. NP-Completeness and Intractability. Approximation Algorithms. Linear Programming. The methods you reach for when the problem looks combinatorial-hard or has the structure of a constraint system.

**Probabilistic (Chapter 12).** Randomized Algorithms. Placed deliberately between optimization chapters: randomized methods are tools the optimization chapters reach for, and the probabilistic outlier deserves its own chapter rather than being scattered.

Linear Programming sits at Chapter 13 — last — as the capstone optimization method that closes threads opened in Chapters 6, 9, 10, and 11.

## How chapters are shaped

Every chapter follows the same anatomy. TL;DR for orientation. A "When [method] applies" or "Recognition pattern" section: what does the problem look like that calls for this chapter? A pointer to prerequisites. Three to five sections covering the method. A "Decision rules" section — usually a table or short flowchart — for fast in-chapter lookup. One worked example drawn from a real practitioner domain, never a pedagogical hypothetical. A "Failure modes" section listing where this chapter's method goes wrong, including the named misconception the chapter engages. Cross-references to other chapters. Companion-page handoffs naming what extended material lives at bearbrown.co. An anti-capability paragraph naming what the chapter does *not* enable. A capability statement closing the chapter.

There are no exercises, no learning objectives, no chapter-opening hooks. This is a reference book.

## How this book sits next to others

Four other books occupy the same territory. Each fits a different reading position.

**CLRS** (Cormen, Leiserson, Rivest, Stein, *Introduction to Algorithms*) is the academic canon. Definitive coverage, formal proofs, course-textbook scaffolding, hardcover price. Use CLRS when you need a proof, a theoretical bound, or a graduate-level treatment.

**Sedgewick** (*Algorithms*, 4th edition) is Java-anchored, with full source listings and visualizations. Use Sedgewick when you want implementation-ready code in Java, with tightly integrated theory.

**Skiena** (*Algorithm Design Manual*) is the interview-prep classic. The "war stories" sections are excellent. Use Skiena when you want practitioner anecdotes and a problem-shaped catalog.

***Algorithms in a Nutshell*** (Heineman, Pollice, Selkow, O'Reilly) was the closest match to this book. Out of print since 2015. [verify] No current Kindle-priced practitioner reference fills the slot it left. This volume occupies that slot.

This book is not a substitute for any of those. It is the cheap, plain-language, fast-lookup reference that complements them.

## Decision rules — which chapter to consult

| You are asking | Start at |
| --- | --- |
| What does this cost? | Chapter 2 (Algorithm Analysis) |
| What structure should I use? | Chapter 3 (Data Structures) |
| Which sort? Which cache policy? | Chapter 4 (Sorting and Caching) |
| Shortest path? Connectivity? Search? | Chapter 5 (Graphs) |
| Is greedy provably optimal here? | Chapter 6 (Greedy) |
| Self-similar subproblems? | Chapter 7 (Divide and Conquer) |
| Overlapping subproblems? | Chapter 8 (Dynamic Programming) |
| Bipartite matching? Min-cut segmentation? | Chapter 9 (Network Flow) |
| Suspect this is NP-hard? | Chapter 10 (NP-Completeness) |
| Need a guarantee but not optimal? | Chapter 11 (Approximation) |
| Randomization seems to help? | Chapter 12 (Randomized) |
| Linear constraints, linear objective? | Chapter 13 (Linear Programming) |

When the diagnosis is unclear, start at Chapter 2 or 3 and follow cross-references.

## Worked example — three readers using the book

**Reader A — picking a sort for a streaming workload.** A backend engineer is building a service that ingests time-stamped events and emits a sorted feed downstream. Throughput is bounded by sort cost. Reader A opens Chapter 4 (Sorting and Caching) and goes to "Decision rules" first. The table directs them to consider the data distribution: events arrive nearly sorted by timestamp with occasional out-of-order bursts. The Timsort discussion in §3 explains why standard library sorts (Python's `list.sort()`, Java's `Arrays.sort()` for objects) detect natural runs and exploit them. Reader A closes the chapter knowing the right call is the language-standard sort, not a hand-rolled quicksort, and has a one-line answer to a code reviewer who asks why.

**Reader B — looking up Bayesian inference and being redirected.** A former biologist now working as a quant opens the book searching for "Bayes." The introduction's scope statement names the boundary: classical algorithms here, decision-making algorithms — Bayesian methods, optimal stopping, game theory — in Vol. 2. Reader B closes the book and orders Vol. 2. The redirect saved them an hour. The scope statement did its job.

**Reader C — cramming for an exam.** An undergraduate has CLRS Chapter 16 (greedy algorithms) assigned for a midterm in three days. They have read the chapter twice and the proofs are not landing. Reader C opens Chapter 6 (Greedy). The "Recognition pattern" section names the matroid and exchange-argument templates in plain English. The interval scheduling worked example walks through the exchange argument step by step. Reader C re-reads the CLRS chapter the next morning and the proofs land. The book did not replace CLRS. It made CLRS legible.

## Failure modes — when this book is the wrong reference

This book is the wrong reference when you need formal proofs (use CLRS), implementation-ready code in a specific language (use Sedgewick or the companion page), competitive-programming patterns at depth (use Skiena or *Competitive Programming Handbook*), or modern machine learning algorithms (use the *Algorithms by Bear* magazine).

The book is also wrong when you have not read Chapter 2. Every later chapter assumes you can interpret a complexity bound. If you skip Chapter 2 and open Chapter 8 cold, the recurrences will not parse. The fix is short: read Chapter 2 first.

A third failure mode is treating decision rules as substitutes for understanding. Decision tables let you choose fast. The chapter sections explain *why* the choice is correct. When the rule disagrees with your judgment on a real problem, the section explains who is right.

## Cross-references

The "How the book is shaped" architecture above doubles as a cross-reference map. Foundations feed core methods; core methods feed optimization; randomization sits across the optimization cluster. Specific cross-references appear at the end of every chapter.

For ML-era algorithms, see the *Algorithms by Bear* magazine. For decision-making algorithms, see Vol. 2.

## Companion-page handoffs

This chapter has no companion-page handoffs. Chapter 1 is purely book-resident — the orientation and decision rules need to be in the book itself or they fail their purpose.

## What this chapter does not enable

This chapter does not teach any algorithm. No method has been explained. No complexity bound has been derived. The chapter is orientation only — what the book is for, how it is shaped, when to consult it. If you came here looking for an answer to a specific algorithmic question, the answer lives in one of the next twelve chapters; consult the decision table above.

## Capability statement

You can now navigate this book efficiently. You know what kind of reference it is, how its chapters are structured, when to consult it versus CLRS / Sedgewick / Skiena / the companion page / Vol. 2 / the magazine, and which chapter to open first for a given algorithmic question. The next time a problem lands on your desk that smells algorithmic, you know where to start.


---

## LLM Exercise — Chapter 1: Initialize the Toolkit

**Project:** *Algorithms by Bear* Code-Along Toolkit — a public repository where each chapter contributes implementations, tests, and benchmarks, building over 13 chapters into a portfolio-grade reference codebase.

**What you're building this chapter:** The repository skeleton — directory structure, README, CI scaffold, and the decision-card template every later chapter populates.

**Tool:** Claude Code (file creation, test scaffolding, and git from one tool).

**The prompt:**

```
I am working through *Algorithms by Bear, Vol. 1* and building a companion
repository where each chapter contributes implementations, tests, and
benchmarks. This is Chapter 1's setup task — pure scaffolding.

Please scaffold a new repository called `algorithms-by-bear-toolkit` with
this structure:

- A top-level `README.md` explaining the project: one module per chapter,
  each with implementations, tests, and a benchmark harness. Note that
  Chapter 2 will build the shared harness all later chapters use.
- A `chapters/` directory with 13 subdirectories named
  `ch01_introduction` through `ch13_linear_programming`.
- Each chapter directory contains: a `README.md` (one-paragraph statement
  of what the chapter covers), an `__init__.py`, an `implementations.py`
  (placeholder), `test_implementations.py` (placeholder), and a
  `benchmarks.py` (placeholder).
- A top-level `pyproject.toml` for Python 3.11+ with `pytest`,
  `pytest-benchmark`, `numpy`, `matplotlib`, `hypothesis` as
  dependencies.
- A `.github/workflows/test.yml` that runs `pytest -q` on push.
- A `.gitignore` for Python.
- A `Makefile` with `test`, `bench`, `lint` targets.

For Chapter 1 specifically — the only chapter without an algorithmic
implementation — write `chapters/ch01_introduction/README.md` containing
a *decision-card template* in markdown. Later chapters fill it in for
each algorithm they implement. The template should have headings for:

  - Problem signal (when this algorithm earns the lookup)
  - Algorithm (one-line description)
  - Complexity (best / average / worst, time and space)
  - When it wins
  - When it fails
  - Standard library equivalent (in your language)
  - See also (cross-references to other chapters)

Run `pytest -q` to confirm the empty scaffold passes (no tests yet).
Initialize git, make the initial commit with the message
`ch01: scaffold repo and decision-card template`.

If my environment suggests a different language (Go, Rust, TypeScript),
ask before defaulting to Python.
```

**What this produces:** A committed repo with 13 chapter directories, a working CI scaffold, and the decision-card template ready to fill in. ~25 files, no algorithmic code yet.

**How to adapt this prompt:**

- *For your own project:* Replace Python with your language. Go: `go mod init`, table-driven tests, `testing.B`. Rust: Cargo with criterion benches. TypeScript: pnpm + vitest + tinybench.
- *For ChatGPT / Gemini:* Both produce the same scaffold. Without Claude Code's file-write loop, ask for each file in a separate fenced block with the path commented at the top, then assemble locally.
- *For Claude Code:* Native fit. Let it run the full create-and-commit loop before reviewing.
- *For a Claude Project:* Skip — Claude Code is the right tool. (If you want continuity across sessions, create a Project that holds the repo's README and decision-card template as context for non-coding consultations.)

**Connection to previous chapters:** None — this is the seed.

**Preview of next chapter:** Chapter 2 builds the benchmark harness every later chapter uses. The skeleton you just created is where the harness will live.
