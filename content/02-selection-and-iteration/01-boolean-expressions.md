---
title: "Boolean expressions and short-circuiting"
navTitle: "Boolean expressions"
summary: >-
  Unit 2 is the largest slice of the exam after data collections, and this is
  where its multiple-choice questions live.
objectives:
  - Evaluate a compound boolean expression by hand
  - Explain what short-circuit evaluation does and when it matters
  - Apply De Morgan's laws to simplify a negated condition
  - Avoid the == versus .equals trap on objects
status: complete
standard: java21
---

Unit 2 is worth 25–35% of the exam score, and a large share of that is
multiple-choice questions asking what a condition evaluates to. Those questions
are not about cleverness — they are about tracing carefully, and about three
specific traps.

## The operators

```java run title="The three you need"
public class Main {
    public static void main(String[] args) {
        int x = 5;
        System.out.println(x > 3 && x < 10);   // both must hold
        System.out.println(x < 3 || x > 4);    // either will do
        System.out.println(!(x == 5));         // flips it
    }
}
```

Nothing surprising yet. The surprises start when the operands have side effects
or can fail.

## Short-circuiting

`&&` and `||` **stop as soon as the answer is decided.** If the left side of
`&&` is false, the right side is never evaluated at all.

```java run title="The right side is not always evaluated"
public class Main {
    static boolean shout(String label) {
        System.out.println("evaluated " + label);
        return true;
    }

    public static void main(String[] args) {
        System.out.println("--- false && ... ---");
        boolean a = false && shout("right of &&");

        System.out.println("--- true || ... ---");
        boolean b = true || shout("right of ||");

        System.out.println("--- true && ... ---");
        boolean c = true && shout("right of &&");

        System.out.println(a + " " + b + " " + c);
    }
}
```

The first two never print their "evaluated" line. The third does.

This is not an optimisation detail — it is a guarantee you are expected to rely
on, and it is what makes this safe:

```java run title="Order is what stops the crash"
public class Main {
    public static void main(String[] args) {
        String s = null;

        // Safe: the left side is false, so length() is never called.
        if (s != null && s.length() > 0) {
            System.out.println("not empty");
        } else {
            System.out.println("null or empty, and no exception");
        }
    }
}
```

Swap those two conditions and the program throws a `NullPointerException`.
**The order of the operands is part of the correctness**, which is unusual — for
most operators, order does not matter.

:::pitfall
The exam asks this as "how many times is the method called". Count carefully:
an operand to the right of a decided `&&` or `||` is **not** evaluated, so any
counter it increments does not increment.
:::

## De Morgan's laws

Negating a compound condition flips the operator as well as the parts:

```
!(a && b)  is  !a || !b
!(a || b)  is  !a && !b
```

```java run title="Checking both laws exhaustively"
public class Main {
    public static void main(String[] args) {
        boolean[] values = { true, false };
        boolean ok = true;

        for (boolean a : values) {
            for (boolean b : values) {
                if (!(a && b) != (!a || !b)) ok = false;
                if (!(a || b) != (!a && !b)) ok = false;
            }
        }

        System.out.println("both laws hold in all four cases: " + ok);
    }
}
```

Four cases is the entire truth table, so that loop is a proof rather than a
sample. Worth doing once: the laws then stop being something to recall under
pressure.

The common error is flipping the parts and leaving the operator alone —
writing `!a && !b` for `!(a && b)`. That is a *different condition*, true in
fewer cases, and it produces code that looks right and behaves wrongly.

## `==` against `.equals`

For objects, `==` asks whether two references point at the **same object**;
`.equals` asks whether they represent the **same value**.

```java run title="The trap, and why it hides"
public class Main {
    public static void main(String[] args) {
        String a = "hello";
        String b = "hello";
        String c = new String("hello");

        System.out.println("a == b     : " + (a == b));        // true — pooled
        System.out.println("a == c     : " + (a == c));        // false
        System.out.println("a.equals(c): " + a.equals(c));     // true

        Integer m = 127, n = 127;
        Integer p = 128, q = 128;
        System.out.println("127 == 127 : " + (m == n));        // true — cached
        System.out.println("128 == 128 : " + (p == q));        // false
    }
}
```

This is the trap the exam loves, and the reason it is a trap is that **`==`
often appears to work.** Identical string literals are pooled into one object,
and small `Integer` values are cached, so `==` gives the right answer for
`"hello"` and for `127` — then silently gives the wrong one for a string built
at run time or an integer above 127.

The rule is simple and admits no exceptions worth remembering:

- **Primitives** (`int`, `double`, `char`, `boolean`): use `==`.
- **Objects** (`String`, `Integer`, anything you wrote): use `.equals`.

:::quiz
{
  "question": "What does this print?\n\nint[] count = {0};\nboolean r = (count[0] > 0) && (++count[0] > 0);\nSystem.out.println(count[0]);",
  "options": [
    {
      "text": "0, because the left operand is false and the right is never evaluated",
      "correct": true,
      "why": "count[0] > 0 is 0 > 0, which is false, and && decides there. The increment on the right never runs, so the counter stays at 0. This is exactly the 'how many times' question in disguise."
    },
    {
      "text": "1, because ++count[0] runs before the comparison",
      "correct": false,
      "why": "It would run first within its own operand, but that operand is never reached. Short-circuiting happens before the right side is evaluated at all."
    },
    {
      "text": "1, because both sides of && are always evaluated",
      "correct": false,
      "why": "That describes & and |, the non-short-circuiting versions. && and || are guaranteed to stop once the result is determined."
    },
    {
      "text": "It throws an exception, since count[0] is compared before assignment",
      "correct": false,
      "why": "count[0] is initialised to 0 by the array initialiser, so the comparison is perfectly valid — it is simply false."
    }
  ]
}
:::

:::recap
- `&&` and `||` short-circuit: once the answer is decided, the other operand is
  never evaluated.
- That guarantee is what makes `s != null && s.length() > 0` safe, and it makes
  operand order part of correctness.
- De Morgan: negating flips the operator too. `!(a && b)` is `!a || !b`, not
  `!a && !b`.
- `==` compares references, `.equals` compares values. Use `==` for primitives
  only.
- `==` on objects often appears to work — pooled literals, cached small
  `Integer`s — which is precisely what makes it dangerous.
:::
