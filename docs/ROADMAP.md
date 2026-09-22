# What to write next

Take the top unticked chapter. One chapter per session, one chapter per commit.

## The CED check — done 2026-09-14, with a caveat

The first version of this file carried a ten-unit outline written from memory.
**It was substantially wrong.** The 2025–26 revision cut AP Computer Science A
from ten units to four, and removed inheritance from the course entirely — a
chapter the old outline had. Forty chapters built on that would have been the
most expensive mistake available here.

**Caveat on sourcing.** `apcentral.collegeboard.org` is blocked by this
environment's network egress, so the structure below comes from secondary
sources — prep publishers and teaching sites reporting on the revision — not
from the official CED PDF. It is consistent across several of them, which is
worth something, but it is not the primary document.

- [ ] **Confirm against the official CED PDF** when someone has a browser that
      can reach College Board. Check especially the four FRQ types: the source
      describing them ("Methods & Control Structures, Class Writing,
      Array/ArrayList, 2D Array") also says they have run unchanged since 2004,
      which is a claim about the *old* course. A revision that deleted a unit
      may well have changed them, and that source would not show it.

## The course, as revised for 2025–26

Four units. The weightings are the share of the exam score, and they are the
single most useful fact in this file: they say where a reader's time is worth
spending, and therefore how many chapters each unit earns.

| Unit | Weight | What it covers |
|---|---|---|
| 1 Using Objects and Methods | 15–25% | Java fundamentals, reference data, calling methods |
| 2 Selection and Iteration | 25–35% | Conditionals, loops, algorithms built from repetition |
| 3 Class Creation | 10–18% | Expressing behaviour and attributes as classes |
| 4 Data Collections | 30–40% | Arrays, `ArrayList`, 2-D arrays, searching, sorting, recursion |

**Gone from the course:** inheritance and polymorphism, which the old Unit 9
carried. The stated reason is closer alignment with introductory college
courses. Do not write those chapters; link to JavaTB for readers who want them.

**Added:** text files and data sets.

## The exam

- **Section I** — 42 multiple-choice questions, four options each, 55% of the
  score, roughly 2.1 minutes per question.
- **Section II** — 4 free-response questions, 45% of the score.
- **Digital.** From 2025–26 the exam is taken in College Board's Bluebook
  application, not on paper. That changes what practice should feel like, and
  it is a reason for this book to exist in a browser.

## Chapter plan

Chapter counts follow the weightings rather than the unit numbering, which is
why Unit 4 gets nearly half the book and Unit 3 gets three chapters.

### Wave A — Unit 1: Using Objects and Methods

- [x] 1.1 Primitive types, and the arithmetic that surprises you
- [x] 1.2 References: what a variable holds when it is not a number
- [x] 1.3 Calling methods, and reading a signature
- [x] 1.4 `String`, and the methods the exam actually tests
- [x] 1.5 The `Math` class, and choosing a number in a range — retitled: 1.1
      already owns integer division (it is that chapter's first objective, with
      its own section, the negative case and the `(int)`/`Math.round` contrast),
      so a second pass would have been the 6.1 mistake. What was genuinely
      missing is `Math.random`, `pow` and `sqrt`, none of which appeared
      anywhere in the book.

### Wave B — Unit 2: Selection and Iteration

- [x] 2.1 Boolean expressions, short-circuiting, and De Morgan
- [ ] 2.2 `if`, `else if`, and the dangling case
- [x] 2.3 Loops, and tracing one by hand (also covers nesting, while, break/continue)
- [x] 2.4 Nested loops, and counting how many times the body runs — folded into 2.3
- [ ] 2.5 `==` against `.equals`, which the exam asks every year
- [ ] 2.6 Building an algorithm out of a loop

### Wave C — Unit 3: Class Creation

- [x] 3.1 Fields, constructors, and what `this` is for (also covers accessors, mutators and toString)
- [ ] 3.2 Accessors, mutators, and why the exam cares about encapsulation
- [ ] 3.3 `static` against instance, and when each is right

### Wave D — Unit 4: Data Collections

- [x] 4.1 Arrays: declaring, filling, and going off the end
- [ ] 4.2 Traversing an array, and the off-by-one
- [x] 4.3 `ArrayList`: what it adds, and the boxing that comes with it
- [ ] 4.4 Removing while iterating, which is the classic wrong answer
- [x] 4.5 2-D arrays: row-major order and nested traversal
- [x] 4.6 Searching: linear and binary
- [x] 4.7 Sorting: selection and insertion (merge is not in the tested subset)
- [x] 4.8 Recursion, and tracing it without a debugger
- [ ] 4.9 Text files and data sets

### Wave E — the exam itself

- [ ] 5.1 How a multiple-choice question is built, and how to read one
- [x] 5.2 The free-response section — all four types, rubric strategy, timing
- [ ] 5.3 FRQ 2 in depth: writing a class
- [ ] 5.4 FRQ 3 in depth: array and `ArrayList`
- [ ] 5.5 FRQ 4 in depth: 2-D array
- [ ] 5.6 Scoring: what earns a point and what does not

## Rules specific to this book

- **Problems must be original.** Real College Board free-response and
  multiple-choice questions are copyrighted. Write to the same archetypes and
  the same difficulty; never transcribe. Cheaper to honour from chapter one
  than to retrofit across two hundred problems.
- **This is not an official College Board product** and must never imply it is.
- **Teach the rubric, not just the answer.** AP free response is scored per
  rubric point, and "justify your answer" earns nothing without the reasoning.
- **Stay inside the tested subset.** When something useful is outside it — the
  whole of inheritance, now — say so in one line and link to JavaTB.

## Standing work

- [ ] **Readers see raw `$` maths on four pages.** Found 2026-09-22 while
      writing 1.5. This book has no KaTeX — it is not in `package.json` and
      `build/markdown.ts` has no math plugin — but four chapters write
      `$7/2$`, `$\log_2 n$`, `$\tfrac13$` and about forty more spans, which
      reach the built page with the dollar signs and backslashes intact.
      Confirmed by grepping `dist/`, not by reading the source.

      Affected here: 1.1, 2.3 and 4.6. 1.5 was written around it.
      **The same defect is in CppTB and JavaTB** — 14 more files, neither
      carrying KaTeX either — so this is a family decision, not a CsaTB one.

      Two ways: add KaTeX to the three compiled books, as CalTB and PhysTB
      have; or rewrite the spans in prose and backticks. The complexity
      discussion in 4.6 (`n^2`, `log_2 n`) is the strongest argument for the
      first and the reason not to just delete the markup.

- Add problems to any chapter carrying fewer than two.
- Every chapter needs `objectives` in its front-matter, or it renders as a blank
  row on `/reference/` and `/progress/`.
