---
title: "The free-response section"
navTitle: "Free response"
summary: >-
  Four questions, 45% of the score, marked per rubric point. Knowing where the
  points are is worth more than knowing more Java.
objectives:
  - Recognise the four free-response types
  - Write an answer that collects partial credit
  - Avoid the errors that lose points on correct code
  - Budget time across the section
status: complete
standard: java21
---

Section II is four questions in 90 minutes for 45% of the score. Unlike the
multiple-choice section, **partial credit is the norm** — and that fact should
change how you write.

## The four types

| # | Type | What it asks |
|---|---|---|
| 1 | Methods and control structures | Write a method using loops and conditionals |
| 2 | Class | Write a complete class from a specification |
| 3 | Array / `ArrayList` | Traverse or modify a 1-D collection |
| 4 | 2-D array | Traverse or modify a grid |

:::note
This four-type structure is long-standing, and the 2025–26 revision changed the
course substantially — it removed inheritance entirely. Treat the list as
**probably right and worth confirming** against the current Course and Exam
Description before relying on it for a study plan.

The chapter's advice on collecting partial credit does not depend on the
breakdown being exact.
:::

## Rubric points, not right answers

An answer is marked against a list of specific points — a correct loop bound,
a correct return, a correct update. You collect them independently.

The consequences are practical:

**Write something for every part.** A part left blank scores zero with
certainty. An attempt that gets the loop right and the condition wrong scores
the loop.

**Later parts often assume earlier ones.** If part (b) uses the method from part
(a), you may **call your part (a) method even if you think it is wrong** — you
are marked on part (b)'s logic, not on (a)'s correctness a second time.

**Do not leave a method half-deleted.** Crossed-out code is not marked, but
confused code might be. Commit to one version.

## Use the methods you are given

Specifications hand you helper methods. Using them is expected, and
reimplementing them wastes time and invites mistakes:

```java run title="Use what the specification provides"
public class Main {
    public static void main(String[] args) {
        Roster r = new Roster();
        System.out.println("count above 80: " + r.countAbove(80));
    }
}

class Roster {
    private int[] scores = {72, 95, 88, 61, 80};

    // Imagine this one is GIVEN by the specification.
    public int getScore(int i) { return scores[i]; }
    public int size()          { return scores.length; }

    // This is the part you are asked to write.
    public int countAbove(int limit) {
        int count = 0;
        for (int i = 0; i < size(); i++) {       // given method, not scores.length
            if (getScore(i) > limit) count++;    // given method, not scores[i]
        }
        return count;
    }
}
```

Reaching into `scores` directly would work here, and on the exam it can cost a
point when the specification says to use the accessor — the rubric sometimes
names it.

## The errors that lose points on correct logic

```java run title="Four things that cost marks"
public class Main {
    public static void main(String[] args) {
        int[] a = {1, 2, 3};

        // 1. length vs length() vs size()
        System.out.println("array:     " + a.length);
        System.out.println("String:    " + "abc".length());
        System.out.println("ArrayList: " + new java.util.ArrayList<>(java.util.List.of(1,2)).size());

        // 2. Returning from the right place — inside the loop returns too early.
        System.out.println("all positive (correct): " + allPositive(a));

        // 3. Comparing objects with == instead of .equals
        String s = new String("hi");
        System.out.println("== : " + (s == "hi") + "   .equals : " + s.equals("hi"));

        // 4. Off-by-one at the top of a loop
        int last = a[a.length - 1];
        System.out.println("last element: " + last);
    }

    static boolean allPositive(int[] a) {
        for (int v : a) {
            if (v <= 0) return false;    // a failure is decided immediately
        }
        return true;                     // success only after the whole loop
    }
}
```

That `allPositive` structure is worth memorising: **return false inside the loop,
return true after it.** Putting `return true` inside answers after the first
element, which is the commonest logic error on FRQ 1.

## Writing without a compiler

The exam is digital now, in Bluebook, but it does not compile your code. Four
habits that matter more without a compiler:

- **Declare every variable**, with its type. `for (i = 0; ...)` is a compile
  error and a lost point.
- **Match your braces as you write them** — open and close together, then fill
  in.
- **Check your method signature against the specification**, exactly: name,
  return type, parameter types and order.
- **Reread the return type.** A method declared `int` that ends without
  returning on some path does not compile.

## Time

Roughly 22 minutes each. A practical split:

- **2 minutes reading**, including the parts you are not on yet — later parts
  often clarify earlier ones.
- **15 minutes writing.**
- **5 minutes checking** — trace your loop on a two-element example and on an
  empty one.

If a part will not come, **move on and come back**. Four questions each worth
the same means a fourth question left blank costs far more than a weak part (c).

:::quiz
{
  "question": "On FRQ part (b) you must use a method written in part (a), but you think your part (a) is wrong. What should you do?",
  "options": [
    {
      "text": "Call your part (a) method as specified — part (b) is marked on its own logic",
      "correct": true,
      "why": "Rubric points are awarded per part. Part (b) is assessed on whether it uses the method correctly, not on whether that method works. Reimplementing it wastes time and forfeits nothing you would otherwise keep."
    },
    {
      "text": "Rewrite part (a)'s logic inside part (b) so it is correct",
      "correct": false,
      "why": "This costs time and can lose points where the rubric expects the method to be called. It also does not recover part (a)'s marks, which are already settled."
    },
    {
      "text": "Leave part (b) blank, since it depends on broken code",
      "correct": false,
      "why": "A blank scores zero with certainty. Part (b)'s points are available regardless of what part (a) does."
    },
    {
      "text": "Write a note explaining that part (a) may be wrong",
      "correct": false,
      "why": "Readers mark code against a rubric, not explanations. The minute spent writing the note is better spent on the code."
    }
  ]
}
:::

:::recap
- Four questions, 45%, marked per rubric point — partial credit is the norm.
- Never leave a part blank; a partly correct attempt collects the points it
  earns.
- Call your own earlier methods even if you doubt them. Later parts are marked
  on their own logic.
- Use the helper methods the specification provides rather than reaching into
  fields.
- `return false` inside the loop, `return true` after it.
- About 22 minutes per question, and move on rather than stalling.
:::
