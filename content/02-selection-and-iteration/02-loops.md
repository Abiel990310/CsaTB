---
title: "Loops, and tracing them by hand"
navTitle: "Loops"
summary: >-
  The exam asks what a loop prints and how many times its body runs. Both are
  answered by tracing, and tracing is a skill you practise.
objectives:
  - Trace a loop by hand and produce its output
  - Count how many times a loop body executes
  - Convert between for and while
  - Recognise the off-by-one and the infinite loop
status: complete
standard: java21
---

Unit 2 is 25–35% of the exam, and loop questions dominate it. Almost all of them
are one of two questions: **what does this print**, and **how many times does the
body run**.

Neither rewards cleverness. Both reward writing the values down.

## The three parts of a `for`

```java run title="Which part runs when"
public class Main {
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            System.out.println("body with i = " + i);
        }
        System.out.println("--- loop finished ---");
        // i is out of scope here: it belongs to the loop.
    }
}
```

The order is: **initialise once**, then repeatedly **test, body, update**.

The test happens *before* each body, including the first — so a `for` whose
condition starts false runs zero times, which is a favourite question.

```java run title="Zero iterations, and the boundary"
public class Main {
    public static void main(String[] args) {
        int count = 0;
        for (int i = 5; i < 5; i++) count++;
        System.out.println("i < 5 starting at 5:  " + count + " iterations");

        count = 0;
        for (int i = 5; i <= 5; i++) count++;
        System.out.println("i <= 5 starting at 5: " + count + " iterations");

        count = 0;
        for (int i = 0; i < 5; i++) count++;
        System.out.println("0 to < 5:             " + count + " iterations");

        count = 0;
        for (int i = 1; i <= 5; i++) count++;
        System.out.println("1 to <= 5:            " + count + " iterations");
    }
}
```

**Counting iterations.** For `for (int i = a; i < b; i++)` the body runs
$b - a$ times. For `i <= b` it runs $b - a + 1$. Getting that one wrong is the
off-by-one, and it is worth deriving rather than memorising: list the values
$i$ takes and count them.

## Tracing

Write a column per variable and a row per iteration. Nothing else is reliable
under exam conditions.

```java run title="Trace this before you run it"
public class Main {
    public static void main(String[] args) {
        int total = 0;
        for (int i = 1; i <= 4; i++) {
            total += i * i;
            System.out.println("i=" + i + "  i*i=" + (i*i) + "  total=" + total);
        }
        System.out.println("answer: " + total);
    }
}
```

| $i$ | $i^2$ | total after |
|---|---|---|
| 1 | 1 | 1 |
| 2 | 4 | 5 |
| 3 | 9 | 14 |
| 4 | 16 | 30 |

Four rows, no ambiguity. The temptation is to do it in your head and be wrong by
one row.

## Nested loops multiply

```java run title="Counting the inner body"
public class Main {
    public static void main(String[] args) {
        int inner = 0;
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 4; j++) {
                inner++;
            }
        }
        System.out.println("3 × 4 = " + inner);

        // A triangular nest: the inner bound depends on the outer variable.
        int tri = 0;
        StringBuilder shape = new StringBuilder();
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j <= i; j++) {
                tri++;
                shape.append("*");
            }
            shape.append("\n");
        }
        System.out.println("1+2+3+4 = " + tri);
        System.out.print(shape);
    }
}
```

Independent nests multiply: $3 \times 4 = 12$.

When the inner bound depends on the outer variable, they do not. `j <= i` gives
$1 + 2 + 3 + 4 = 10$, a triangular number — and the exam asks this shape
specifically because multiplying gives the wrong answer.

## `while`, and the infinite loop

A `for` is a `while` with the three parts gathered up:

```java run title="The same loop, both ways"
public class Main {
    public static void main(String[] args) {
        StringBuilder a = new StringBuilder();
        for (int i = 0; i < 4; i++) a.append(i).append(" ");

        StringBuilder b = new StringBuilder();
        int i = 0;                 // initialise
        while (i < 4) {            // test
            b.append(i).append(" ");
            i++;                   // update — easy to forget
        }

        System.out.println("for:   " + a);
        System.out.println("while: " + b);
        System.out.println("same:  " + a.toString().equals(b.toString()));
    }
}
```

Use `while` when the number of iterations is not known in advance — reading
until a sentinel, repeating until converged. Use `for` when it is.

:::pitfall
**The commonest infinite loop is a missing update.** In a `for`, the update sits
in the header where it is hard to omit. In a `while` it is an ordinary statement
in the body, and forgetting it is silent.

The second commonest is updating the wrong variable — incrementing `i` while the
condition tests `j`. Both compile perfectly.
:::

## `break` and `continue`

```java run title="Stopping early and skipping ahead"
public class Main {
    public static void main(String[] args) {
        int[] data = {4, 8, 15, 16, 23, 42};

        // break: stop the loop entirely
        for (int v : data) {
            if (v > 15) { System.out.println("first over 15: " + v); break; }
        }

        // continue: skip to the next iteration
        int oddSum = 0;
        for (int v : data) {
            if (v % 2 == 0) continue;
            oddSum += v;
        }
        System.out.println("sum of odds: " + oddSum);
    }
}
```

`break` exits the loop; `continue` skips the rest of this iteration and moves on.
In nested loops, both affect only the **innermost** loop containing them — which
is exactly what a tracing question will check.

:::quiz
{
  "question": "How many times does the innermost body run?\n\nfor (int i = 0; i < 4; i++)\n  for (int j = 0; j < i; j++)\n    count++;",
  "options": [
    {
      "text": "6 — the inner bound is i, so the counts are 0 + 1 + 2 + 3",
      "correct": true,
      "why": "When i is 0 the inner condition j < 0 is immediately false, so it runs zero times. Then 1, then 2, then 3 — totalling 6."
    },
    {
      "text": "16, since both loops run 4 times",
      "correct": false,
      "why": "That is the answer for independent nests, where the inner bound is a constant. Here the inner bound is i, so it grows as the outer loop proceeds."
    },
    {
      "text": "10 — the counts are 1 + 2 + 3 + 4",
      "correct": false,
      "why": "That would be j <= i, which runs one extra time per outer iteration. With j < i the first pass runs zero times, not one."
    },
    {
      "text": "12, since the inner loop averages 3 iterations",
      "correct": false,
      "why": "The average is 6/4 = 1.5, not 3. Adding the actual counts is both easier and correct."
    }
  ]
}
:::

:::recap
- A `for` initialises once, then repeats test–body–update. The test comes first,
  so zero iterations is possible.
- `i < b` from `a` runs $b-a$ times; `i <= b` runs $b-a+1$. Derive it by listing
  the values.
- Trace with a column per variable and a row per iteration. Do not do it in your
  head.
- Independent nests multiply; a nest whose inner bound depends on the outer
  variable gives a triangular total.
- The commonest infinite loop is a missing or misdirected update in a `while`.
- `break` and `continue` affect only the innermost enclosing loop.
:::
