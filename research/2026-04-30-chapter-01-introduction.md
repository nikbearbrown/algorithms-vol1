# Research notes — Chapter 01: Introduction to Algorithms

## Source folders used
- `Introduction_to_Algorithms/README.md` — empty, no source content
- `book.md` — book identity, audience, architecture, scope
- `outline.md` — chapter map, capabilities, three reader walkthroughs

## Factual claims preserved from source
- Book identity (Kindle, $0.99, KU, 4–5 year refresh) — from book.md
- Architecture (4 clusters, 13 chapters, LP at Ch 13) — from book.md
- Reader profiles (A: backend dev, B: career-switcher, C: undergrad) — from book.md and outline.md
- Series structure (Vol. 1 / Vol. 2 / magazine) — from book.md
- Comparison to CLRS / Sedgewick / Skiena / *Algorithms in a Nutshell* — from book.md
- ML-scope statement — from book.md preface
- Recommendation to read Chapters 2 and 3 first — from book.md preface

## Claude additions marked [verify]
- *Algorithms in a Nutshell* "out of print since 2015" — claim sourced from book.md, but the date specifically should be checked against publisher records. Marked [verify].

## [verify] count
- 1 inline `[verify]` (out-of-print date)

## Voice-anchoring notes
- This chapter is the highest risk for textbook drift because it's the orientation chapter — natural temptation to open with "Welcome." Resisted. Opens with TL;DR followed immediately by "When this book applies."
- No hooks, no scene-setting, no "I want to tell you why this matters." Reference voice held throughout.
- The three reader walkthroughs are the most narrative section. Kept tight (each one paragraph, action-driven).
- Decision table fits reference shape — table of "you are asking" → "start at." Reader scans, picks a chapter, leaves.
- No exercises, no learning objectives, no chapter-opening hook, no chapter summary, no preview of next chapter. Structure check passes.

## What made this chapter hard to keep in reference shape
- The introduction-chapter convention in pedagogy is to motivate. The reference convention is to orient. The temptation to motivate ("algorithms are the heart of computer science") had to be cut on sight three times during drafting.
- The "How this book sits next to others" section drifted toward review-style prose on the first pass. Pulled back to one-paragraph-each functional comparisons.

## Word count
~1,520 words (target: ~1,500)

## Open issues for editor
- *Algorithms in a Nutshell* out-of-print date — verify against O'Reilly catalog
- The decision-rules table covers chapters not yet drafted. Numbers and names match outline.md but should be re-verified once chapters are revised.
- "Reader A — picking a sort for a streaming workload" cites Python's `list.sort()` and Java's `Arrays.sort()`. These claims about Timsort being the underlying algorithm are widely-canonical but worth confirming against current language-version docs in editorial pass.
