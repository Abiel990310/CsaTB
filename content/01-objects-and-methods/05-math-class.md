---
title: "The Math class, and choosing a number in a range"
navTitle: "The Math class"
summary: >-
  Five methods, four of which hand back a double. The one that does not is
  where most of the difficulty lives, and the random-in-a-range idiom is worth
  deriving rather than memorising.
objectives:
  - Name what each Math method returns, and cast when the type does not fit
  - Write the idiom that picks a random integer in an inclusive range
  - Explain why Math is called on the class and never on an object
  - Say what Math.abs does with Integer.MIN_VALUE
status: complete
standard: java21
requires: [primitives-and-arithmetic]
---

`Math` is the first library class the exam assumes you already know, and it is
small enough to learn in one sitting. The AP Java Quick Reference — the sheet
you are given during the exam — lists five methods on it, and that is the whole
tested surface.

The methods are not the hard part. The types are. Four of the five hand back a
`double`, which collides immediately with chapter 1.1's rule that Java will not
quietly narrow a `double` into an `int`. And the fifth, `Math.random`, gives
you a number in a range nobody wants, so turning it into a number somebody
wants is an idiom you have to build.

```java run title="All five, and what each gives back"
public class Main {
    public static void main(String[] args) {
        System.out.println(Math.abs(-7));       // 7      — an int
        System.out.println(Math.abs(-7.5));     // 7.5    — a double
        System.out.println(Math.pow(2, 3));     // 8.0    — note the .0
        System.out.println(Math.sqrt(16));      // 4.0    — note the .0
        System.out.println(Math.random() < 1);  // true   — always
    }
}
```

Two of those printed values have a `.0` on the end that a reader expecting
arithmetic would not have predicted. That is the chapter.

## Five methods, and what each gives back

| Call | Returns | Note |
|---|---|---|
| `Math.abs(int)` | `int` | magnitude, sign discarded |
| `Math.abs(double)` | `double` | the same method name, a different method |
| `Math.pow(a, b)` | `double` | **always**, even for `pow(2, 3)` |
| `Math.sqrt(x)` | `double` | **always**, even for `sqrt(16)` |
| `Math.random()` | `double` | in the range `0.0` up to but not including `1.0` |

`abs` appearing twice is not a typo. Those are two **overloads**, chosen by the
argument's type exactly as chapter 1.3 described: pass an `int` and you get an
`int` back, pass a `double` and you get a `double`. It is the only method here
whose return type depends on what you hand it.

:::note
`Math` has plenty of other methods — `Math.round`, `Math.min`, `Math.max`,
`Math.floor` — and they are real Java that compiles and runs. Chapter 1.1 uses
`Math.round` to contrast rounding with truncation.

They are simply **not on the quick reference**, so an exam question will not
require one. Knowing them is useful; relying on one in a free-response answer
when the reference sheet does not list it is a risk with no upside, since
anything they do can be written with what is listed.
:::

## `pow` and `sqrt` always return a double

This is the first place the types bite, and it does not compile.

```java run expect-error title="A double will not fit in an int by itself"
public class Main {
    public static void main(String[] args) {
        int n = Math.pow(2, 3);       // 8.0 is a double
        System.out.println(n);
    }
}
```

That sample is verified not to compile. `javac` says *incompatible types:
possible lossy conversion from double to int* — the same refusal chapter 1.1
met, arriving from a method's return type rather than from a literal. Java
narrows a `double` to an `int` only when you say so:

```java run title="Say so, with a cast"
public class Main {
    public static void main(String[] args) {
        int n = (int) Math.pow(2, 3);
        System.out.println(n);                       // 8
        System.out.println(Math.pow(2, 3));          // 8.0
        System.out.println((int) Math.sqrt(16));     // 4
        System.out.println(Math.sqrt(2));            // 1.4142135623730951
    }
}
```

The cast is not cosmetic. `(int)` **truncates**, as 1.1 established, so it
discards whatever is after the decimal point rather than rounding it. For
`Math.pow(2, 3)` there is nothing to discard. For `Math.sqrt(2)` there is
rather a lot.

:::pitfall
`(int) Math.sqrt(x)` is the integer **below** the square root, not the nearest
one. `(int) Math.sqrt(15)` is `3`, because the square root of 15 is about
3.873 and truncation goes down.

That is usually what a question about "how many whole ..." wants, and almost
never what a question about "the closest ..." wants. Decide which you are being
asked before you reach for the cast.
:::

## `Math.random` stops just short of 1.0

`Math.random()` returns a `double` that is **at least 0.0 and always less than
1.0**. One end is included and the other is not, and that asymmetry is the
whole specification — it is also what the next section is built on.

Asserting that is cheap. Demonstrating it costs one loop:

```java run title="A million draws, and where they land"
public class Main {
    public static void main(String[] args) {
        double lowest = 1.0;
        double highest = 0.0;

        for (int i = 0; i < 1_000_000; i++) {
            double r = Math.random();
            lowest = Math.min(lowest, r);
            highest = Math.max(highest, r);
        }

        System.out.println("lowest  " + lowest);
        System.out.println("highest " + highest);
        System.out.println("hit 1.0? " + (highest >= 1.0));   // false
    }
}
```

A million draws land somewhere around `0.0000002` at the bottom and
`0.9999993` at the top. The exact figures differ every run — that is rather the
point — but the last line does not: **`1.0` never appears.** It cannot, and a
million attempts failing to find it is the most convincing demonstration
available for something defined by exclusion.

## Choosing an integer in a range

Here is what the exam actually asks. You want a whole number from `min` to
`max`, with both ends included — a die roll, an array index, a random letter.
Build it in three steps rather than memorising the answer.

**Stretch it.** `Math.random()` covers `0.0` up to but not including `1.0`.
Multiplying by `n` stretches that to `0.0` up to but not including `n`. A die
has six outcomes, so multiply by 6 and you cover `0.0` up to but not
including `6.0`.

**Truncate it.** `(int)` throws away the fraction, so that block becomes the
integers `0, 1, 2, 3, 4, 5`. Six of them, which is right, but they start in the
wrong place.

**Shift it.** Add `min` to slide the block onto the range you wanted.

```java
(int)(Math.random() * (max - min + 1)) + min
```

The `+ 1` is the part everyone loses. The count of integers from `min` to `max`
**inclusive** is `max - min + 1`, not `max - min`: there are six numbers from 1
to 6, not five. Drop the `+ 1` and the top value becomes unreachable.

```java run title="The idiom, and the same idiom minus the + 1"
public class Main {
    static int roll(int min, int max) {
        return (int)(Math.random() * (max - min + 1)) + min;
    }

    static int broken(int min, int max) {
        return (int)(Math.random() * (max - min)) + min;      // no + 1
    }

    public static void main(String[] args) {
        int lo = 9, hi = -9;
        for (int i = 0; i < 600_000; i++) {
            int r = roll(1, 6);
            lo = Math.min(lo, r);
            hi = Math.max(hi, r);
        }
        System.out.println("roll(1, 6)   saw " + lo + " to " + hi);

        lo = 9; hi = -9;
        for (int i = 0; i < 600_000; i++) {
            int r = broken(1, 6);
            lo = Math.min(lo, r);
            hi = Math.max(hi, r);
        }
        System.out.println("broken(1, 6) saw " + lo + " to " + hi);
    }
}
```

Six hundred thousand rolls of the correct version see both `1` and `6`. Six
hundred thousand of the broken one never once see a `6` — not rarely, never,
because `(int)(Math.random() * 5)` cannot exceed `4`.

That is what makes this bug expensive in real code. It is not a crash and it is
not intermittent. It is a six-sided die that silently rolls five, and the only
way to notice is to look at the range, which is why the sample above prints the
range rather than a few sample rolls.

:::warning
The multiplication must happen **before** the cast.

```java
(int)(Math.random() * 6)     // 0 to 5
(int) Math.random() * 6      // always 0
```

The second casts `Math.random()` on its own. That value is below 1.0, so the
cast truncates it to `0`, and `0 * 6` is `0` every single time. Both lines
compile, and one of them is a constant.
:::

## `Math` is static, all the way down

Every one of these is called on the **class**, never on an object —
`Math.sqrt(16)`, not `someMath.sqrt(16)`. Chapter 1.3 drew the distinction; this
is the class that makes it concrete, because there is no object to be had.

```java run expect-error title="There is no such thing as a Math"
public class Main {
    public static void main(String[] args) {
        Math m = new Math();
        System.out.println(m.sqrt(16));
    }
}
```

Verified not to compile: *Math() has private access in Math*. The constructor
is deliberately hidden, so an instance cannot be made even if you want one.

The reason is the one from 1.3. An instance method usually needs its object's
data — `word.length()` has to know *which* word. `Math.sqrt(16)` needs nothing
except the 16 you passed it, so there is nothing for it to be called on, and a
class that holds no data has no reason to be instantiated.

## `abs` has one value it cannot negate

One genuine trap, and it is a favourite because it looks impossible.

```java run title="A magnitude that came back negative"
public class Main {
    public static void main(String[] args) {
        System.out.println(Math.abs(-7));                  // 7
        System.out.println(Math.abs(Integer.MIN_VALUE));   // -2147483648
    }
}
```

`Math.abs` of a negative number returned a **negative** number, and that output
is verified.

It is chapter 1.1's overflow, reappearing. An `int` spans `-2147483648` to
`2147483647` — one more value below zero than above, because zero occupies a
slot on the positive side. So `+2147483648` is not an `int` at all. `Math.abs`
computes it, it wraps, and it comes back where it started.

Nothing is broken; the documented behaviour is exactly this. It is the last
consequence of the fixed-width arithmetic from 1.1, and it is worth knowing
because "take the absolute value, then use it as a length" is a line that works
on every input but one.

## Check yourself

:::quiz
{
  "question": "Which expression gives a random integer from 5 to 10, with both ends possible?",
  "options": [
    {
      "text": "(int)(Math.random() * 6) + 5",
      "correct": true,
      "why": "There are 10 − 5 + 1 = 6 values in the range, so the multiplier is 6. Math.random() * 6 covers [0.0, 6.0), truncating gives 0 through 5, and adding 5 shifts that to 5 through 10."
    },
    {
      "text": "(int)(Math.random() * 5) + 5",
      "correct": false,
      "why": "This is the missing + 1. The multiplier 5 gives 0 through 4, so the results run 5 to 9 and the 10 never appears — a bug that produces plausible output forever and is only visible if you check the range."
    },
    {
      "text": "(int)(Math.random() * 10) + 5",
      "correct": false,
      "why": "The multiplier is the count of values wanted, not the upper bound. This gives 0 through 9 shifted to 5 through 14, overshooting the top of the range by four."
    },
    {
      "text": "(int) Math.random() * 6 + 5",
      "correct": false,
      "why": "Without the brackets the cast applies to Math.random() alone, which is below 1.0 and truncates to 0. The expression is 0 * 6 + 5, which is 5 every time — a constant that looks like an idiom."
    }
  ]
}
:::

## Practice

:::exercise random-in-range

:::exercise math-return-types

:::recap
- The quick reference lists five `Math` methods: `abs` on an `int`, `abs` on a
  `double`, `pow`, `sqrt` and `random`. Others exist and compile; the exam will
  not require them.
- `pow` and `sqrt` always return a `double`, so assigning one to an `int` needs
  a cast — and the cast truncates rather than rounds.
- `Math.random()` runs from `0.0` up to but not including `1.0`. That excluded
  `1.0` is why the range idiom needs its `+ 1`.
- `(int)(Math.random() * (max - min + 1)) + min` picks from `min` to `max`
  inclusive. Without the `+ 1` the top value is unreachable; with the cast
  outside the brackets the whole thing is a constant.
- `Math` is entirely static and cannot be instantiated — the constructor is
  private, so `new Math()` does not compile.
- `Math.abs(Integer.MIN_VALUE)` is negative, because the positive counterpart
  is not an `int`.
:::
