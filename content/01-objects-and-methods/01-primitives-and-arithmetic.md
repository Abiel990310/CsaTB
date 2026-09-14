---
title: "Primitives and the arithmetic that surprises you"
navTitle: "Primitives"
summary: >-
  Integer division, overflow, and floating-point error — three behaviours that
  are not bugs and are tested every year.
objectives:
  - Predict the result of integer division and modulus
  - Explain when a cast is needed and where to put it
  - Recognise integer overflow
  - Say why floating-point comparisons fail
status: complete
standard: java21
---

Unit 1 is 15–25% of the exam, and a disproportionate share of that is arithmetic
that behaves exactly as specified and not at all as expected.

## Integer division truncates

```java run title="The single most-tested behaviour in Unit 1"
public class Main {
    public static void main(String[] args) {
        System.out.println("7 / 2   = " + (7 / 2));       // 3, not 3.5
        System.out.println("7 % 2   = " + (7 % 2));       // 1, the remainder
        System.out.println("-7 / 2  = " + (-7 / 2));      // -3, toward zero
        System.out.println("-7 % 2  = " + (-7 % 2));      // -1, sign of the left
        System.out.println("7.0 / 2 = " + (7.0 / 2));     // 3.5 — one double is enough
    }
}
```

Two integers divided give an integer, and the fraction is **discarded, not
rounded**. $7/2$ is $3$ and $9/2$ is $4$ — both truncate toward zero rather than
rounding to nearest.

Negative division truncates toward zero too, so $-7/2$ is $-3$ rather than
$-4$. And `%` takes the sign of the **left** operand, which is why `-7 % 2` is
$-1$ and not $1$. That is a genuine difference from mathematical modulus and it
appears in questions about even/odd tests on possibly-negative values.

:::pitfall
`if (n % 2 == 1)` **fails for negative odd numbers**, because `-3 % 2` is `-1`,
not `1`.

Use `n % 2 != 0` instead. It is the same length and it is correct on the whole
range.
:::

```java run title="The even/odd trap"
public class Main {
    public static void main(String[] args) {
        for (int n : new int[]{3, -3}) {
            System.out.println(n + ": n%2==1 says " + (n % 2 == 1)
                               + ", n%2!=0 says " + (n % 2 != 0));
        }
    }
}
```

## Casting, and where to put it

```java run title="The cast has to happen before the division"
public class Main {
    public static void main(String[] args) {
        int a = 7, b = 2;

        System.out.println("(double)(a / b) = " + (double)(a / b));   // 3.0 — too late
        System.out.println("(double) a / b  = " + ((double) a / b));  // 3.5 — correct
        System.out.println("a / (double) b  = " + (a / (double) b));  // 3.5 — also fine
    }
}
```

The first casts the *result* of an integer division, by which point the
information is gone. Casting either operand first promotes the whole expression
to `double`.

Casting the other way truncates:

```java run title="double to int discards, it does not round"
public class Main {
    public static void main(String[] args) {
        System.out.println("(int) 3.9  = " + (int) 3.9);    // 3
        System.out.println("(int) -3.9 = " + (int) -3.9);   // -3, toward zero
        System.out.println("Math.round(3.9) = " + Math.round(3.9));
        System.out.println("Math.round(3.4) = " + Math.round(3.4));
    }
}
```

`(int)` truncates; `Math.round` rounds. A question asking for "the whole number
of…" usually wants truncation, and one asking to "round to the nearest" wants
`Math.round`.

## Overflow wraps silently

```java run title="int has a largest value, and adding to it wraps"
public class Main {
    public static void main(String[] args) {
        int max = Integer.MAX_VALUE;
        System.out.println("Integer.MAX_VALUE = " + max);
        System.out.println("max + 1           = " + (max + 1));

        // Even an intermediate result can overflow before being widened.
        int big = 100000;
        System.out.println("big * big as int  = " + (big * big));
        System.out.println("as long           = " + ((long) big * big));
    }
}
```

`max + 1` is the most negative `int`. No exception, no warning — the arithmetic
wraps around, and the program carries on with a wrong number.

The third line is the version that catches people: `big * big` is computed as
`int` arithmetic *and then* assigned, so widening afterwards is too late. Casting
one operand first, as in the last line, does it properly.

## Floating point is approximate

```java run title="Why == on doubles is a trap"
public class Main {
    public static void main(String[] args) {
        double sum = 0.1 + 0.2;
        System.out.println("0.1 + 0.2      = " + sum);
        System.out.println("== 0.3 ?         " + (sum == 0.3));
        System.out.println("difference       " + (sum - 0.3));
        System.out.println("close enough ?   " + (Math.abs(sum - 0.3) < 1e-9));
    }
}
```

`0.1` and `0.2` have no exact binary representation, any more than $\tfrac13$
has an exact decimal one. The sum is very slightly off, and `==` notices.

The fix is to compare with a tolerance: `Math.abs(a - b) < 0.000001`. This is
not a Java defect — every language using binary floating point behaves this way,
and the exam expects you to know it.

:::quiz
{
  "question": "int a = 7, b = 2; What does System.out.println((double)(a / b)) print?",
  "options": [
    {
      "text": "3.0 — the integer division happens first, then the cast",
      "correct": true,
      "why": "The parentheses force a / b to be evaluated as int arithmetic, giving 3. Casting that to double gives 3.0. The fractional part was discarded before the cast could matter."
    },
    {
      "text": "3.5 — the cast makes the division floating-point",
      "correct": false,
      "why": "It would, if it were applied to an operand: (double) a / b gives 3.5. Here it applies to the already-computed result."
    },
    {
      "text": "3 — casting to double does not change how it prints",
      "correct": false,
      "why": "A double prints with a decimal point, so it shows 3.0 rather than 3. The value is right; the formatting differs."
    },
    {
      "text": "4.0 — integer division rounds to nearest",
      "correct": false,
      "why": "Integer division truncates rather than rounding. 7 / 2 is 3, and 9 / 2 would be 4 — by discarding the remainder, not by rounding it."
    }
  ]
}
:::

:::recap
- Two ints divided give an int; the fraction is discarded, truncating toward
  zero.
- `%` takes the sign of its left operand, so `n % 2 == 1` fails for negative
  odds. Use `n % 2 != 0`.
- Cast an **operand**, not the result: `(double) a / b`, never
  `(double)(a / b)`.
- `(int)` truncates and `Math.round` rounds.
- `int` overflow wraps silently, including in intermediate results — cast to
  `long` before multiplying, not after.
- Never compare doubles with `==`; compare the difference against a tolerance.
:::
