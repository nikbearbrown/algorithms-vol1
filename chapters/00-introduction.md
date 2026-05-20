<!--
00-introduction.md — Book-level introduction.

The Introduction does different work than the Preface:
  - Preface  = why the book exists, why you wrote it (author's voice)
  - Introduction = what the book argues and how it is organized (reader's roadmap)

This file is a stub. Sections 1–10 and 12–13 are placeholders for a later pass.
Section 11 (A note about AI) is substantive and written.

A good model for the full version: Pearl's "The Mind Over Data" introduction,
Molnar's Interpretable ML introduction. Both are argument-first and tell the
reader exactly what to expect from each chapter.
-->

# Introduction

<!-- [1] COLD OPEN
     A specific named scene with real stakes.
     No "this book will...", no throat-clearing.
     Open on a sentence that contains the whole problem.
     Like the Swedish triage case in computational-skepticism-for-ai. -->

[COLD OPEN PLACEHOLDER]

<!-- [2] THE CENTRAL CLAIM — one sentence.
     "This book is about the gap between [X] and [Y]." -->

[CENTRAL CLAIM PLACEHOLDER]

<!-- [3] THE CENTRAL ARGUMENT — a testable, contestable claim
     about what the book is doing. -->

[CENTRAL ARGUMENT PLACEHOLDER]

<!-- [4] AUDIENCE LOCATION — one sentence locating who this is for. -->

[AUDIENCE PLACEHOLDER]

---

## What This Book Is

<!-- [5] Scope. The work the book names. Vocabulary it teaches. -->

[SCOPE PLACEHOLDER]

## What This Book Is Not

<!-- [6] Explicit exclusions. Prerequisites. -->

[EXCLUSIONS PLACEHOLDER]

---

## A Central Concept That Runs Throughout

<!-- [7] A recurring idea readers should watch for across chapters.
     Like "the fluency trap" in computational-skepticism-for-ai. -->

[CENTRAL CONCEPT PLACEHOLDER]

<!-- [8] (OPTIONAL) A RUNNING NARRATIVE THREAD
     A case that recurs across chapters as a worked example.
     Like "Ash" in computational-skepticism-for-ai.
     Delete this section if not using a running thread. -->

## A Running Narrative Thread

[NARRATIVE THREAD PLACEHOLDER — delete this section if not using one]

---

## How This Book Is Organized

<!-- [9] Chapter-by-chapter map. Group into movements (clusters of 3–5)
     if applicable. One sentence per chapter is enough. -->

[CHAPTER MAP PLACEHOLDER]

## How to Read This Book

<!-- [10] Order. Prerequisites for skipping around.
     Self-contained chapters. Chapter-closing features
     (e.g., "What would change my mind", "Still puzzling", exercises). -->

[READING GUIDE PLACEHOLDER]

---

## A Note about AI

The model is fluent in algorithm vocabulary. It will explain a quicksort, narrate a Dijkstra trace, and produce code for either on request. That fluency is a teaching asset and a verification trap, and the difference is what this note is about.

Algorithms is a field where correctness is mathematically defined and empirically testable. Every algorithm has an invariant. Every implementation has a runtime. Every claim of complexity has a proof. The model can recite all three for any algorithm in this book. What it cannot do, reliably, is verify any of them for the specific implementation you write or read.

Where the model genuinely helps: producing the canonical explanation of why an algorithm works (the loop invariant, the recurrence, the exchange argument), walking through a worked trace at small input size, generating skeleton code in your target language, and stress-testing your understanding with adversarial inputs you may not have thought to construct. The model is also excellent at translating between formal and informal expressions of the same idea — a recurrence relation in one paragraph, an English-language argument for the same bound in the next.

Where the model does damage: declaring that your implementation is correct, predicting actual runtime on your machine, producing code that compiles and is subtly wrong on edge cases the model did not consider. The model has read many implementations of every algorithm in this book. It will produce code that looks like a textbook implementation and contains an off-by-one, a wrong sentinel value, or an unstable comparator that the textbook implementation does not. Worse, when the code is wrong, the model will defend it with the same confidence it defended the correct version. Trust the test, not the prose around the code.

A specific failure mode worth naming: the model is biased toward producing the *expected* algorithm for a problem class even when the problem statement subtly requires a different one. Ask it for sorting code and you will get one of the canonical sorts, not the variant your problem requires. Read every problem statement against the code the model produces and ask whether the algorithm actually matches.

The rule that covers all three: the model knows the algorithm. Your tests know your code. Where those two knowledges should converge, the convergence is empirical — you have to run the tests. The model's word that the code is right is worth nothing. The benchmark and the unit test are worth everything.

---

## Closing

<!-- [12] Callback to the opening scene. End with a directive. -->

[CLOSING PLACEHOLDER]

---

**Tags:** <!-- [13] 5–8 discoverability tags --> [TAGS PLACEHOLDER]
