---
title: "Arrays"
navTitle: "Arrays"
summary: >-
  The largest unit on the exam starts here. Fixed length, zero-indexed, and the
  two off-by-one errors that account for most lost marks.
objectives:
  - Declare, create and initialise an array
  - Traverse an array with both loop forms and know when each is usable
  - Avoid ArrayIndexOutOfBoundsException at both ends
  - Explain why an array parameter can be modified by a method
status: complete
standard: java21
---

Unit 4 is **30–40% of the exam score** — the largest single unit, and larger
than Units 1 and 3 combined. It starts with arrays, and almost everything later
in the unit is a loop over one.

## Three ways to make one

```java run title="Declaring and creating"
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        // 1. Size only: every element gets a default value.
        int[] a = new int[4];

        // 2. Listed values, length inferred.
        int[] b = {10, 20, 30};

        // 3. Same, written out in full — needed when not on a declaration line.
        int[] c = new int[]{7, 8};

        System.out.println(Arrays.toString(a));   // defaults
        System.out.println(Arrays.toString(b));
        System.out.println(Arrays.toString(c));
        System.out.println("lengths: " + a.length + " " + b.length + " " + c.length);
    }
}
```

The defaults are worth knowing because the exam tests them: `0` for numeric
types, `false` for `boolean`, and **`null` for any object type** — including
`String`, which is the one that surprises people.

```java run title="Defaults, including the dangerous one"
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] ints = new int[3];
        boolean[] flags = new boolean[3];
        String[] words = new String[3];

        System.out.println(Arrays.toString(ints));
        System.out.println(Arrays.toString(flags));
        System.out.println(Arrays.toString(words));

        // A String[] is full of nulls, not empty strings:
        System.out.println("is it \"\"? " + "".equals(words[0]));
        System.out.println("is it null? " + (words[0] == null));
    }
}
```

Calling a method on one of those nulls throws `NullPointerException`, and it is
a common way for an FRQ to fail at run time after looking complete.

## `length` is a field, not a method

```java
array.length      // arrays — no parentheses
string.length()   // Strings — parentheses
list.size()       // ArrayList — different name entirely
```

Three collections, three spellings, and mixing them up is a compile error rather
than a silent bug — which makes it annoying rather than dangerous. It still
costs time on a written FRQ, where the compiler is not there to help.

## Traversing

```java run title="Both loop forms"
public class Main {
    public static void main(String[] args) {
        int[] scores = {88, 95, 72, 100};

        // Indexed: you know where you are, and you can write.
        int total = 0;
        for (int i = 0; i < scores.length; i++) {
            total += scores[i];
        }

        // Enhanced: shorter, read-only over the array itself.
        int best = scores[0];
        for (int s : scores) {
            if (s > best) best = s;
        }

        System.out.println("total " + total + ", best " + best);
    }
}
```

**Choose the indexed form when you need the position** — to compare neighbours,
to write into the array, or to start somewhere other than the beginning.
**Choose the enhanced form when you only need the values.**

:::pitfall
The enhanced for loop **cannot modify the array.** The loop variable is a copy,
so assigning to it changes nothing.

```java
for (int s : scores) { s = 0; }        // scores is unchanged
for (int i = 0; i < scores.length; i++) { scores[i] = 0; }   // this works
```

The first compiles cleanly and does nothing, which makes it a favourite
multiple-choice question.
:::

## The two off-by-one errors

Valid indices run from `0` to `length - 1`. Both ends have a classic mistake.

```java run expect-throw title="The last element is length - 1"
public class Main {
    public static void main(String[] args) {
        int[] a = {1, 2, 3};
        System.out.println("last is " + a[a.length - 1]);
        System.out.println("about to go one too far…");
        System.out.println(a[a.length]);     // throws
    }
}
```

That sample is asserted to throw, and the verifier checks that it does — a
demonstration of a crash that stopped crashing would be worse than useless.

The loop condition is the place this is decided:

- `i < a.length` — correct.
- `i <= a.length` — one too far, and throws on the last iteration.

## Arrays are objects

An array variable holds a **reference**. Two consequences that the exam tests
directly:

```java run title="Passing an array lets a method change it"
import java.util.Arrays;

public class Main {
    static void zeroFirst(int[] data) {
        data[0] = 0;               // reaches through the reference
    }

    static void reassign(int[] data) {
        data = new int[]{99, 99};  // rebinds the local copy only
    }

    public static void main(String[] args) {
        int[] nums = {5, 6, 7};

        zeroFirst(nums);
        System.out.println("after zeroFirst: " + Arrays.toString(nums));

        reassign(nums);
        System.out.println("after reassign:  " + Arrays.toString(nums));
    }
}
```

The first method changes the caller's array; the second does not. Java passes
the **reference by value** — the method gets its own copy of the arrow, so it
can follow the arrow and alter what is there, but pointing its copy somewhere
else has no effect outside.

That single distinction explains every array-parameter question on the exam.

:::quiz
{
  "question": "What does this print?\n\nint[] a = {1, 2, 3};\nfor (int x : a) { x = x * 2; }\nSystem.out.println(a[0]);",
  "options": [
    {
      "text": "1 — the enhanced for loop cannot write back into the array",
      "correct": true,
      "why": "x is a copy of each element. Doubling the copy leaves the array untouched. To modify elements you need the indexed form and a[i] = a[i] * 2."
    },
    {
      "text": "2 — each element is doubled in place",
      "correct": false,
      "why": "That is what the code looks like it does, which is why this question appears so often. The loop variable is a separate int; assigning to it changes nothing outside the loop body."
    },
    {
      "text": "It does not compile, since x is effectively final",
      "correct": false,
      "why": "The enhanced-for variable is an ordinary local and may be reassigned. It compiles and runs cleanly — and accomplishes nothing, which is the trap."
    },
    {
      "text": "0 — x is reset each iteration",
      "correct": false,
      "why": "x is reassigned from the array each iteration, so it takes the values 1, 2, 3. Nothing ever writes 0 anywhere, and a[0] was never touched."
    }
  ]
}
:::

## Practice

:::exercise array-off-by-one

:::recap
- Arrays are fixed length, zero-indexed, and `length` is a field with no
  parentheses.
- Defaults are `0`, `false`, and **`null`** for object types — a `String[]` is
  full of nulls, not empty strings.
- Valid indices are `0` to `length - 1`; the loop condition is `i < a.length`.
- The enhanced for loop gives copies and cannot write into the array. Use the
  indexed form to modify.
- An array parameter lets a method change the caller's contents, because the
  reference is copied but the array is not.
:::
