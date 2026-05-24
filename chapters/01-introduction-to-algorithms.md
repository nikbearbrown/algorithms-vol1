# Chapter 1 — Introduction to Algorithms

*The map before the territory.*

---

There is a moment every engineer knows. You are staring at a problem that feels algorithmic — something about sorting, or searching, or finding the shortest path through a graph — and you cannot quite remember what the right tool is called, or you can name it but cannot remember what it actually does, or you know both things but want to confirm one complexity bound before you commit. The problem is real. The deadline is real. What you need is a fast answer, not a course.

This book is for that moment.

It is not a textbook. It is not a theoretical reference. It is a handbook — the kind of book you open to a specific chapter because you have a specific problem, confirm what you need, and close. The chapters are designed for lookup. You are expected to skip around.

But before you skip, read this chapter. Not because it teaches an algorithm — it teaches none — but because it tells you how the book is shaped, which chapter answers which kind of question, and when this book is the wrong book and something else is right. That orientation is fast, and skipping it costs more time than reading it.

---

## What kind of question this book answers

The question has to have a certain shape for this book to help.

Here is the shape: you have a problem in front of you, you suspect there is a named algorithm or a known analysis technique that applies, and you want to identify it, recall what it does, confirm its cost, or recognize where it fails. Questions of the form: "Which sort do I use here?" "What does it cost to do this?" "When does this approach break?" "Is this provably optimal or just usually good enough?"

That shape is what this book addresses.

Here is a shape this book does not address: "How do I implement a red-black tree in C++ from scratch?" Your standard library does that for you in production, and a textbook with full code listings does it for learning. This book tells you when a balanced tree is the right data structure and what it costs — not how to build one from memory.

Here is another shape this book does not address: "What is the gradient of attention with respect to the query weights?" That is a different book. The *Algorithms by Bear* magazine covers ML-era methods. Vol. 2 covers decision-making algorithms — Bayesian inference, optimal stopping, game theory. This volume covers the classical canon: analysis, data structures, sorting, graphs, greedy algorithms, divide-and-conquer, dynamic programming, network flow, NP-completeness, approximation, randomized algorithms, and linear programming. Thirteen chapters. Four clusters. One canon.

The boundary matters. Every reference book has one. This one is drawn around the classical algorithms that appear in most software engineering work and almost every algorithms interview, stopping before the territory that belongs to other books.

![Two-region boundary map ](images/01-introduction-to-algorithms-fig-01.png)
*Figure 1.1 — Two-region boundary map *

---

## What you need before you start

Two chapters carry the vocabulary the rest of the book uses: Chapter 2 on algorithm analysis and Chapter 3 on data structures.

Chapter 2 gives you Big O notation, recurrence relations, and the master theorem. These are not decoration. Every later chapter states its costs in terms of Big O, and several derive those costs through recurrences. If you open Chapter 8 on dynamic programming without having read Chapter 2, the recurrences will not parse. The fix is short — Chapter 2 is not long — but it is real.

Chapter 3 gives you heaps, hash tables, and balanced trees. These three structures appear constantly in later chapters, sometimes as the central subject, more often as tools the algorithm uses. Chapter 5 on graphs uses priority queues — which are heaps. Chapter 6 on greedy algorithms uses heaps to extract minimum-cost edges. Chapter 8 on dynamic programming stores intermediate results in hash tables. If you do not know what a heap costs to insert into and extract the minimum from, the analyses in those chapters will feel arbitrary.

So: read Chapter 2. Read Chapter 3. Then skip wherever you want.

If you need a refresher on recursion or probability — two topics that come up in Chapters 7 and 12 specifically — those are on the companion page at bearbrown.co rather than in the book itself. The book keeps the probability primer off the main track because most readers do not need it until Chapter 12, and forcing them past it to get to Chapter 4 would be wrong.

![Dependency graph showing Ch1→Ch2→Ch3 as the mandatory spine,](images/01-introduction-to-algorithms-fig-02.png)
*Figure 1.2 — Dependency graph showing Ch1→Ch2→Ch3 as the mandatory spine,*

---

## How the book is shaped

Thirteen chapters in four clusters. Here they are.

**Foundations.** Chapters 1 through 3. Introduction — this chapter. Algorithm Analysis. Data Structures. The vocabulary layer. Nothing in Chapters 4 through 13 re-explains Big O or re-introduces heaps. Read these first and you will not have to re-read them.

**Core methods.** Chapters 4 through 8. Sorting and Caching. Graphs and Graph Search. Greedy Algorithms. Divide and Conquer. Dynamic Programming. These five families solve most algorithmic problems a working engineer encounters. If your problem is not in here, it is probably in the optimization cluster or it is not a classical algorithms problem.

**Optimization.** Chapters 9 through 11, plus Chapter 13. Network Flow. NP-Completeness and Intractability. Approximation Algorithms. Linear Programming. You reach for these when the problem is combinatorially hard — when you suspect there is no efficient exact algorithm — or when it has the structure of a system of constraints with an objective you want to maximize or minimize.

**Probabilistic.** Chapter 12. Randomized Algorithms. One chapter, placed between the optimization chapters, because randomized methods are tools the optimization chapters reach for and the probabilistic perspective deserves its own treatment rather than being scattered across the book as footnotes.

Linear Programming is last — Chapter 13 — because it is the capstone. It closes threads that Chapters 6, 9, 10, and 11 leave open. By the time you arrive at it, you have enough context to see why it is powerful and what it subsumes.

![Four-cluster architecture diagram ](images/01-introduction-to-algorithms-fig-03.png)
*Figure 1.3 — Four-cluster architecture diagram *

---

## How each chapter is shaped

Every chapter follows the same anatomy. This is intentional. A reference book that is consistent is faster to use than one that is creative.

Each chapter opens with a TL;DR — three to five sentences that state what the chapter covers, when the method applies, and what it costs. If you are in a hurry, the TL;DR tells you whether you are in the right chapter.

Then comes a section called "When [method] applies" or "Recognition pattern" — what does the problem look like that calls for this chapter? This is the diagnostic section. It is what you read when you are not yet sure whether your problem fits here or somewhere else.

Then a pointer to prerequisites. This is honest: if the chapter requires you to know something from an earlier chapter, it says so explicitly and tells you which one.

Then three to five sections covering the method itself — what it is, how it works, why it costs what it costs, where the interesting engineering lives.

Then a "Decision rules" section. Usually a table or a short flowchart. If the chapter covers multiple variants of a method — and many chapters do, because most methods come in several flavors — the decision rules tell you which variant fits which situation. This is the section you return to after you have read the chapter once and need to apply the method later.

Then one worked example from a real practitioner domain. Not a pedagogical hypothetical. A real problem — the kind that appears in production systems or research code — worked through in enough detail to show the method actually running, not just described.

Then a "Failure modes" section. Where does this method go wrong? What is the named misconception the chapter wants to prevent? Most chapters have one central failure mode — one thing engineers get wrong repeatedly — and the chapter names it directly.

Then cross-references to other chapters, a handoff to the companion page for extended material, an anti-capability paragraph that names what the chapter does *not* enable, and a capability statement that closes the chapter by saying what you can now do.

There are no exercises. No learning objectives written for a course. No chapter-opening hooks designed to create engagement. This is a reference book. The anatomy serves the reference function, not the pedagogy function.

![Anatomy of a single chapter page ](images/01-introduction-to-algorithms-fig-04.png)
*Figure 1.4 — Anatomy of a single chapter page *

---

## Which chapter to consult

When the diagnosis is clear, go directly to the chapter. When it is not, start at Chapter 2 or 3 and follow the cross-references.

| You are asking | Start at |
|---|---|
| What does this cost? | Chapter 2 — Algorithm Analysis |
| What structure should I use? | Chapter 3 — Data Structures |
| Which sort? Which cache policy? | Chapter 4 — Sorting and Caching |
| Shortest path? Connectivity? Search? | Chapter 5 — Graphs and Graph Search |
| Is greedy provably optimal here? | Chapter 6 — Greedy Algorithms |
| Self-similar subproblems? | Chapter 7 — Divide and Conquer |
| Overlapping subproblems? | Chapter 8 — Dynamic Programming |
| Bipartite matching? Min-cut segmentation? | Chapter 9 — Network Flow |
| Suspect this is NP-hard? | Chapter 10 — NP-Completeness and Intractability |
| Need a guarantee but not an optimal solution? | Chapter 11 — Approximation Algorithms |
| Randomization seems to help? | Chapter 12 — Randomized Algorithms |
| Linear constraints, linear objective? | Chapter 13 — Linear Programming |

One column of the table deserves a comment. "Self-similar subproblems" and "overlapping subproblems" look like they might mean the same thing. They do not. Self-similar subproblems — where the problem breaks into smaller versions of itself without the sub-answers depending on each other — calls for divide-and-conquer. Overlapping subproblems — where the same sub-problem appears multiple times and recomputing it is wasteful — calls for dynamic programming. The distinction is real and it matters. Chapter 7 and Chapter 8 each explain their half of it, and Chapter 8 opens with a section called "Divide and Conquer versus Dynamic Programming" that makes the boundary precise.

---

## How this book sits next to others

Four other books occupy the same territory. Each fits a different reading position, and knowing which is which saves you from consulting the wrong one.

**CLRS** — Cormen, Leiserson, Rivest, and Stein, *Introduction to Algorithms* — is the academic canon. Definitive coverage, formal proofs, the textbook scaffolding of a graduate course. Hardcover price. Use CLRS when you need a proof, a theoretical bound, or a graduate-level treatment. CLRS does not optimize for fast lookup. It optimizes for rigor. Those are different goals.

**Sedgewick**, *Algorithms* (4th edition), is Java-anchored, with full source listings and visualizations integrated with the text. Use Sedgewick when you want implementation-ready code in Java with tightly integrated theory. It is excellent. It is also a different kind of book — it assumes you want to implement, not just identify and apply.

**Skiena**, *The Algorithm Design Manual*, is the interview-preparation classic and something more. The "war stories" sections — real problems from Skiena's consulting practice — are genuinely good. The catalog of problems in the second half is useful for pattern-matching. Use Skiena when you want practitioner anecdotes and a problem-shaped reference.

***Algorithms in a Nutshell*** (Heineman, Pollice, and Selkow, O'Reilly) was the closest antecedent to this book — Kindle-priced, practitioner-focused, designed for lookup. It has been out of print since approximately 2015. No current book fills the slot it left. This volume occupies that slot.

This book is not a substitute for any of those. It is the cheap, plain-language, fast-lookup reference that complements them.

| book | price tier | primary use case | format (lookup | textbook |
| --- | --- | --- | --- | --- |
| five-column comparison of this book vs. CLRS vs. Sedgewick vs. Skiena vs. Algorithms in a Nutshell — | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Three readers using this book

Abstract descriptions of a book's purpose are less useful than watching the book actually do something. Here are three readers.

**Reader A — picking a sort for a streaming workload.** A backend engineer is building a service that ingests timestamped events and emits a sorted feed downstream. Throughput is bounded by sort cost. Reader A opens Chapter 4 and goes to the "Decision rules" section first. The table asks about the data distribution: events arrive nearly sorted by timestamp, with occasional out-of-order bursts. The section on Timsort explains that standard library sorts in Python and Java detect natural runs — sequences that are already sorted — and exploit them instead of sorting blindly from scratch. Reader A closes the chapter knowing the right call is the language-standard sort, not a hand-rolled quicksort, and has a one-sentence answer ready for any code reviewer who asks why.

**Reader B — looking up Bayesian inference and being redirected.** A former biologist working as a quant opens the book searching for "Bayes." The introduction's scope statement names the boundary: classical algorithms here, decision-making algorithms in Vol. 2. Reader B closes the book and orders Vol. 2. The redirect took thirty seconds. The scope statement did its job. This is what a well-drawn boundary is for.

**Reader C — using this book to unlock a harder book.** An undergraduate has CLRS Chapter 16 on greedy algorithms assigned for a midterm in three days. She has read the chapter twice and the proofs are not landing. She opens Chapter 6 of this book. The "Recognition pattern" section names the matroid and exchange-argument templates in plain English, without the formal apparatus. The interval scheduling worked example walks through the exchange argument step by step — here is the greedy choice, here is why swapping it out cannot improve the solution, here is what that structure is called. She re-reads the CLRS chapter the next morning and the proofs land. This book did not replace CLRS. It made CLRS legible.

Three readers, three uses. The book did not try to be all things. It tried to do one thing well.

---

## When this book is the wrong book

A book that tells you what it cannot do is more useful than one that pretends to do everything.

This book is wrong when you need formal proofs. CLRS has them; this book does not. The complexity bounds here are stated and explained, not proven from axioms. If you are studying for a qualifying exam or writing a paper, you need the proofs.

This book is wrong when you need implementation-ready code in a specific language. Sedgewick, the companion page, or your standard library documentation does that. This book tells you which algorithm to reach for and what it costs. Code listings are elsewhere.

This book is wrong if you skip Chapter 2. This is worth saying directly: every later chapter assumes you can read a Big O bound and run a basic complexity argument. If you open Chapter 8 cold, the recurrences will not parse. Chapter 2 is short. Read it first.

And this book is wrong — importantly wrong — when you treat the decision tables as substitutes for the chapter sections. The tables let you choose fast. The sections explain why the choice is correct. When a decision rule disagrees with your judgment on a real problem, the section is where you find out who is right. The table gets you to the right chapter. The chapter gets you to the right answer.

---

## What this chapter does not enable

Nothing. This chapter is orientation only. No algorithm has been explained. No complexity bound has been derived. No method has been introduced.

If you came here looking for an answer to a specific algorithmic question, the answer is in one of the next twelve chapters. The decision table above tells you which one. Go there.

---

## Capability statement

You can now navigate this book efficiently. You know what kind of reference it is — a lookup handbook, not a course textbook or a theoretical reference. You know how its chapters are structured and what each section is for. You know when to consult CLRS versus Sedgewick versus Skiena versus this book. You know which chapter to open first for a given algorithmic question. And you know when to close this book and reach for something else.

The next time a problem lands on your desk that smells algorithmic, you know where to start.

---

## Exercises

**Warm-up**

1. Open the decision table in this chapter. For each of the following problems, identify the starting chapter the table directs you to, and write one sentence explaining why that chapter applies. (a) You need to find the minimum number of hops between two nodes in an unweighted network. (b) You have a list of job deadlines and want to maximize the number of jobs completed on time. (c) You suspect a scheduling problem you've been handed cannot be solved efficiently in the general case. *(Tests: navigating the decision table; translating problem descriptions into the table's diagnostic language.)*

2. The table distinguishes "self-similar subproblems" from "overlapping subproblems." In your own words — without looking ahead to Chapters 7 or 8 — describe what you think the difference is based only on this chapter's explanation. Then name a type of problem you would expect each framing to apply to. *(Tests: reading comprehension of a precise distinction; preparing the conceptual ground for Chapters 7 and 8.)*

3. Which two chapters does this chapter say must be read before any others, and what specific vocabulary does each one supply? State the answer in terms of what breaks if you skip each chapter. *(Tests: prerequisite awareness; understanding why the foundation chapters are non-optional.)*

**Application**

4. A colleague hands you a problem and says: "I think this is NP-hard, but I'm not sure, and even if it is, I need something I can ship." Map the two parts of that sentence to two different chapters in this book. Explain what each chapter would offer your colleague and what it would not. *(Tests: recognizing that NP-completeness diagnosis and approximation algorithms are separate questions answered by separate chapters.)*

5. You are choosing between this book and CLRS to answer a specific question. For each of the following, state which book you would reach for and why. (a) You want to confirm the worst-case time complexity of heapsort. (b) You want to understand the formal proof that heapsort is optimal in the comparison model. (c) You want to know whether heapsort or Timsort is the right choice for your nearly-sorted input. *(Tests: understanding the distinct purposes of a lookup handbook versus a theoretical reference.)*

6. The "Three readers" section describes Reader C using this book to unlock CLRS. Describe a fourth reader — a specific professional role, a specific algorithmic problem, a specific thing they need — and trace which chapter they would open, which section within that chapter they would go to first, and what they would have when they close the book. Use only chapters and sections described in this chapter. *(Tests: applying the book's navigation model to a novel scenario.)*

**Synthesis**

7. The chapter argues that a consistent chapter anatomy makes a reference book faster to use than a creative one. Evaluate that claim. What does consistency cost, if anything? What would a reader lose if each chapter were structured differently to fit its subject? *(Tests: reasoning about reference-book design as a deliberate trade-off.)*

8. This chapter draws a boundary between what this book covers and what belongs to other books. Identify two topics — one that falls clearly inside the boundary and one that falls clearly outside it — and for each, explain what the boundary criterion is. Then identify one topic where you are genuinely uncertain which side of the boundary it falls on, and explain why the uncertainty exists. *(Tests: understanding scope as a design decision; applying the boundary argument to cases not explicitly named in the chapter.)*

**Challenge**

9. The chapter says: "The table gets you to the right chapter. The chapter gets you to the right answer." This implies the decision table is necessary but not sufficient. Design a situation in which following the decision table correctly — going to the right chapter — still leads an engineer to the wrong algorithmic choice. What does the engineer need from the chapter sections that the table cannot provide? *(Tests: understanding the limits of heuristic lookup tools; reasoning about when rules fail.)*

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

**What this produces:** A committed repo with 13 chapter directories, a working CI scaffold, and the decision-card template ready to fill in. Approximately 25 files, no algorithmic code yet.

**How to adapt this prompt:**

- *For your own project:* Replace Python with your language. Go: `go mod init`, table-driven tests, `testing.B`. Rust: Cargo with criterion benches. TypeScript: pnpm + vitest + tinybench.
- *For ChatGPT / Gemini:* Both produce the same scaffold. Without Claude Code's file-write loop, ask for each file in a separate fenced block with the path commented at the top, then assemble locally.
- *For Claude Code:* Native fit. Let it run the full create-and-commit loop before reviewing.
- *For a Claude Project:* Skip — Claude Code is the right tool. If you want continuity across sessions, create a Project that holds the repo's README and decision-card template as context for non-coding consultations.

**Connection to previous chapters:** None — this is the seed.

**Preview of next chapter:** Chapter 2 builds the benchmark harness every later chapter uses. The skeleton you just created is where the harness will live.
