# CsaTB — working notes for Claude

An interactive AP Computer Science A textbook, on the same engine as CppTB and
JavaTB. Every code sample is compiled by a real compiler and every practice
problem is auto-graded.

**Ten chapters written** — 2.1 boolean expressions, plus most of Unit 4:
arrays, ArrayList, 2-D arrays, searching and sorting, recursion. Chosen by exam
weight rather than unit order, since Unit 4 alone is 30–40% of the score. 48
samples, all compiling. `docs/ROADMAP.md` holds the rest.

## What the CED check found

Done 2026-09-14, and it mattered: the 2025–26 revision cut AP CSA from **ten
units to four** and **removed inheritance from the course**. The outline this
repo started with had both wrong. `docs/ROADMAP.md` now carries the corrected
structure, the exam weightings, and a chapter plan proportioned to them.

One thing is still open, and it is flagged at the top of that file: College
Board's site is blocked by this environment's network egress, so the structure
came from secondary sources rather than the official CED PDF. The four FRQ
types in particular deserve confirmation — the source describing them also
claims they are unchanged since 2004, which is a statement about the course
that was just revised.

## The one rule

Nothing ships unverified:

```bash
npm run verify      # snippets compile, problems grade, types check
```

If a claim cannot be demonstrated by a program that runs, write the program or
delete the claim.

## This book is not JavaTB

JavaTB teaches Java; this teaches the exam. Where the exam does not test
something, this book does not cover it — it links across to JavaTB instead.
Where the exam tests something JavaTB considers trivia, it gets a chapter here.
`docs/ROADMAP.md` spells this out; do not blur it.

Two rules that follow, and are not negotiable:

- **Problems are original.** Real College Board questions are copyrighted. Same
  archetypes, same difficulty, never transcribed.
- **This is not an official College Board product.** Nothing may imply it is.

## Layout

Identical to JavaTB — `content/NN-part/NN-chapter.md` for the book,
`content/exercises/*.md` for problems, `build/` for the pipeline, `scripts/` for
the verifiers, `docs/AUTHORING.md` for the method.

The engine was ported wholesale from JavaTB, so `docs/AUTHORING.md` is JavaTB's
guide. It is accurate about the widgets and the harness. It does not yet carry
the AP-specific rules above — fold them in when the CED check happens.

## Commands

```bash
npm run dev        # localhost:5173
npm run verify     # everything, in one go
npm run build      # static build
```

Needs a JDK on PATH for the verifiers.

## Scheduling

Covered by **`trig_017zYznE5wwj44ZGRirzmBRL`**, the shared 07:00 UTC+8 routine
that maintains all the books. One item per run. Do not create a second Routine.

**Pushing `main` deploys the site**, to https://abiel990310.github.io/CsaTB/.
The workflow derives the Pages sub-path from the repo name, so nothing is
hard-coded. `gh-pages` is generated output — never edit it by hand.

## Inherited gotcha worth knowing

The search index must build its URLs through `url()` from `build/base.ts`, never
as a raw `/part/chapter/`. All these sites are served from `/<repo>/`, and a raw
path resolves against the domain root and 404s — invisibly, because `npm run
dev` serves from `/` where both forms are identical. That bug shipped in both
CppTB and JavaTB before it was found. The port already carries the fix.
