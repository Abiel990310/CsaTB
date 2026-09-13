# What to write next

Take the top unticked chapter. One chapter per session, one chapter per commit.

## Read this before writing a single chapter

**The unit list below is not yet verified against the current CED.** It is the
ten-unit structure that AP Computer Science A carried for years, written down
from knowledge rather than from the College Board's published Course and Exam
Description. The College Board revised AP CSA for 2025–26, and the revision is
believed to have changed both the unit arrangement and the free-response
structure.

So, before Wave A is written:

- [ ] **Check the current CED.** Confirm the unit list, their order, their exam
      weightings, and the number and type of free-response questions. Correct
      this file, and record the date the check was made and what changed.
- [ ] Confirm the Java subset the exam allows. AP CSA tests a deliberately small
      slice of Java; teaching outside it is worse than useless, because it costs
      the reader time on something the exam will not reward and may actively
      penalise in a free-response answer.

Until that check happens, treat every heading below as provisional. A forty-
chapter book built on a stale outline is the most expensive mistake available
here, and it is cheap to avoid by looking once.

## What this book is, and is not

JavaTB teaches Java. **This book teaches the exam.** They overlap in syntax and
in nothing else:

- JavaTB can spend a chapter on why `HashMap` resizes. AP CSA does not test
  `HashMap` at all, so this book does not have that chapter.
- AP CSA tests things JavaTB treats as beneath mention — the exact output of a
  `for` loop written to be confusing, the difference between `==` and `.equals`
  on `Integer`, hand-tracing a recursive call. Those get chapters here.
- Every problem here is shaped like an exam question, because the reader's goal
  is a score, not craftsmanship. That is a legitimate goal and the book should
  serve it honestly rather than pretend to be about something loftier.

Where a reader wants the deeper version, link across to JavaTB. That is what the
hub is for.

## Provisional unit structure

Ten units, to be confirmed against the CED before anything is written.

### Wave A — the language the exam uses

- [ ] 1.1 Primitive types, and the arithmetic that surprises you
- [ ] 1.2 Using objects: references, constructors, and the String methods tested
- [ ] 1.3 Boolean expressions and `if`
- [ ] 1.4 Iteration, and tracing a loop by hand
- [ ] 1.5 Writing classes: fields, constructors, accessors, mutators

### Wave B — data

- [ ] 2.1 Arrays
- [ ] 2.2 `ArrayList`
- [ ] 2.3 2-D arrays
- [ ] 2.4 The standard algorithms the exam expects you to recognise

### Wave C — the rest of the course

- [ ] 3.1 Inheritance, overriding, and polymorphism
- [ ] 3.2 Recursion, and tracing it without a debugger

### Wave D — the exam itself

- [ ] 4.1 How the multiple-choice section is written, and how to read it
- [ ] 4.2 The free-response types, one chapter each
- [ ] 4.3 Scoring: what earns a point and what does not

## Rules specific to this book

- **Problems must be original.** Real College Board free-response and
  multiple-choice questions are copyrighted. Write to the same archetypes and
  the same difficulty; never transcribe. This is not negotiable and it is much
  cheaper to honour from chapter one than to retrofit across two hundred
  problems.
- **This is not an official College Board product** and must never imply it is.
  One line in the README says so; nothing in the prose should contradict it.
- **Teach the rubric, not just the answer.** AP free response is scored per
  rubric point, and "justify your answer" earns nothing without the reasoning.
  A book that shows where the points are is worth more than one that shows a
  correct program.
- **Stay inside the tested subset.** When something useful is outside it, say
  so in one line and link to JavaTB rather than teaching it here.

## Standing work

- Add problems to any chapter carrying fewer than two.
- Every chapter needs `objectives` in its front-matter, or it renders as a blank
  row on `/reference/` and `/progress/`.
