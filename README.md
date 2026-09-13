# CsaTB

An interactive **AP Computer Science A** textbook. Every code sample compiles
and runs in the browser; every practice problem is graded by actually compiling
and running your submission.

Built on the same engine as [CppTB](https://github.com/Abiel990310/CppTB) and
[JavaTB](https://github.com/Abiel990310/JavaTB), with everything
language-specific isolated into `build/language.ts` and `server/compile.ts`.

> **Not an official College Board product.** AP® is a trademark of the College
> Board, which does not endorse and is not involved with this book. Every
> practice question here is original, written to the same shapes the exam uses —
> none is reproduced from a real exam.

## What it is for

JavaTB teaches Java. This teaches the exam, which is narrower and more specific:
the subset of Java that AP tests, the question types it asks, and where the
rubric actually awards points. Where you want the deeper version of a topic, it
links across to JavaTB.

## Status

Early. The engine is ported and builds; the chapters are not written yet. See
[`docs/ROADMAP.md`](docs/ROADMAP.md) — including the check against the current
Course and Exam Description that has to happen before the outline is trusted.

## Running it locally

```bash
npm install
npm run dev      # http://localhost:5173
```

Compiling and grading need a JDK 21 on your PATH. Without one the site still
works — the browser falls back to Compiler Explorer's public API.

## Verifying

```bash
npm run verify
```

Compiles every runnable sample, asserts that the ones marked `expect-error`
really do fail, grades every problem's worked solution (must pass) and starter
(must fail), and typechecks the site. This is what CI runs, and nothing is
committed without it.

## Writing

`docs/ROADMAP.md` is the queue and `docs/AUTHORING.md` is the method.
