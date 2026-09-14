---
title: "Recursion"
navTitle: "Recursion"
summary: >-
  A method that calls itself. The exam tests tracing it by hand far more often
  than writing it.
objectives:
  - Identify the base case and the recursive case
  - Trace a recursive call by hand and produce its output
  - Explain why a missing base case causes StackOverflowError
  - Recognise recursion over arrays and strings
status: complete
standard: java21
---

Recursion questions on the exam are almost always **tracing** questions: here is
a recursive method, what does `f(4)` return? Writing recursion is a smaller part
of the mark than reading it, so this chapter emphasises the reading.

Every recursive method has exactly two parts:

- a **base case** that returns without recursing, and
- a **recursive case** that calls itself on a *smaller* input.

Miss either and it never terminates.

## Tracing, with the calls made visible

```java run title="Factorial, printing every call and return"
public class Main {
    static int indent = 0;

    static int factorial(int n) {
        System.out.println("  ".repeat(indent) + "factorial(" + n + ") called");
        indent++;

        int result;
        if (n <= 1) {
            result = 1;                          // base case
        } else {
            result = n * factorial(n - 1);       // recursive case
        }

        indent--;
        System.out.println("  ".repeat(indent) + "factorial(" + n + ") returns " + result);
        return result;
    }

    public static void main(String[] args) {
        factorial(4);
    }
}
```

The shape of that output is the thing to internalise. The calls go **all the way
down** to the base case before **any** of them return, and then the returns
unwind back up. Nothing is multiplied until the bottom is reached.

That is why a trace question is answered from the bottom up: find the base case
value first, then work outwards.

## The base case is not optional

```java run expect-throw title="What a missing base case does"
public class Main {
    static int broken(int n) {
        return n + broken(n - 1);     // nothing stops it
    }

    public static void main(String[] args) {
        System.out.println("about to recurse without a base case…");
        System.out.println(broken(5));
    }
}
```

That is asserted to throw, and it does: `StackOverflowError`. Each call needs a
stack frame holding its parameters and its place in the code, and the stack is
finite. A few thousand frames deep, it runs out.

Note that `broken(5)` does eventually reach negative numbers and keeps going —
the condition that would stop it simply does not exist. A base case that exists
but is never *reached* fails the same way:

```java
static int alsoBroken(int n) {
    if (n == 0) return 0;
    return alsoBroken(n - 2);      // from an odd n, steps straight past 0
}
```

Starting from an odd number this skips zero forever. The base case must be
reachable from every legal input, not merely present.

## Recursion over an array

The pattern: do something with one element, recurse on the rest.

```java run title="Sum and reverse, recursively"
public class Main {
    static int sum(int[] a, int i) {
        if (i >= a.length) return 0;          // past the end: nothing left
        return a[i] + sum(a, i + 1);          // this one, plus the rest
    }

    static String reverse(String s) {
        if (s.length() <= 1) return s;                        // base case
        return reverse(s.substring(1)) + s.charAt(0);         // rest, then first
    }

    public static void main(String[] args) {
        System.out.println(sum(new int[]{3, 1, 4, 1, 5}, 0));
        System.out.println(reverse("recursion"));
        System.out.println(reverse(""));       // base case: empty stays empty
    }
}
```

An index parameter is how an array recursion shrinks — the array itself does not
get smaller, so something else must.

:::pitfall
The base case for an empty input is easy to omit and is exactly what the exam
tests. `sum` on an empty array returns `0`; `reverse` of `""` returns `""`.

Both are correct and both fall out of the base case being written as
`i >= a.length` rather than `i == a.length - 1`. Writing base cases in terms of
"nothing left" rather than "one left" handles the empty input for free.
:::

## Binary search, recursively

The same algorithm as chapter 4.4, expressed as recursion — and the exam shows
it both ways:

```java run title="Binary search as recursion"
public class Main {
    static int search(int[] a, int target, int lo, int hi) {
        if (lo > hi) return -1;                       // base: window is empty
        int mid = (lo + hi) / 2;
        if (a[mid] == target) return mid;             // base: found it
        if (a[mid] < target) return search(a, target, mid + 1, hi);
        return search(a, target, lo, mid - 1);
    }

    public static void main(String[] args) {
        int[] sorted = {1, 3, 5, 7, 9, 11, 13};
        System.out.println("11 -> " + search(sorted, 11, 0, sorted.length - 1));
        System.out.println("4  -> " + search(sorted, 4, 0, sorted.length - 1));
    }
}
```

Two base cases here, which is normal: one for success and one for exhausting the
search space. The "smaller input" is the shrinking `lo`–`hi` window.

## Tracing quickly

For an exam trace, work **bottom-up**:

1. Find the base case and its value.
2. Substitute upward, one level at a time.
3. Write each level down. Do not hold four levels in your head.

For `factorial(4)`:

```
factorial(1) = 1
factorial(2) = 2 × 1 = 2
factorial(3) = 3 × 2 = 6
factorial(4) = 4 × 6 = 24
```

Four lines, no mental stack, and it is very hard to get wrong.

:::quiz
{
  "question": "static int f(int n) { if (n <= 0) return 0; return n + f(n - 2); }\n\nWhat does f(5) return?",
  "options": [
    {
      "text": "9, because it adds 5 + 3 + 1 and then hits the base case at −1",
      "correct": true,
      "why": "From 5 it steps 5, 3, 1, then −1 which satisfies n <= 0 and returns 0. So 5 + 3 + 1 + 0 = 9. The base case is reachable from an odd start because it tests <= rather than ==."
    },
    {
      "text": "It throws StackOverflowError, since 5 is odd and skips zero",
      "correct": false,
      "why": "That would be true if the base case were n == 0. Because it is n <= 0, the value −1 satisfies it and the recursion terminates — which is exactly why <= is the safer way to write it."
    },
    {
      "text": "15, the sum of 5 + 4 + 3 + 2 + 1",
      "correct": false,
      "why": "That is the n − 1 recursion. This one steps down by 2, so it visits only 5, 3 and 1."
    },
    {
      "text": "8, the sum of 5 + 3",
      "correct": false,
      "why": "This stops one level early. After 3 comes f(1), and 1 is still greater than 0, so it contributes before f(−1) ends the chain."
    }
  ]
}
:::

:::recap
- Every recursive method needs a base case that returns without recursing and a
  recursive case on a **smaller** input.
- Calls descend to the base case before any of them return; the returns unwind
  upward.
- A missing base case gives `StackOverflowError` — and so does one that exists
  but is never reached.
- Write base cases as "nothing left" (`i >= a.length`, `n <= 0`) rather than
  "one left", which handles empty inputs and odd starts for free.
- Trace bottom-up on paper, one line per level.
:::
