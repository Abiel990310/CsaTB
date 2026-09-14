---
id: array-off-by-one
title: "Three array traversals, three boundaries"
difficulty: intro
chapter: arrays
topics: [arrays, loops]
check: unit
standard: java21
---

Three methods over an `int[]`. Each has a boundary bug — the kind that either
throws or silently misses an element.

- `sumAll(a)` — the total. Runs one index too far.
- `largest(a)` — the biggest value. Starts comparing from the wrong place, so a
  single-element array and an all-negative array both go wrong.
- `countMatches(a, target)` — how many elements equal `target`. Stops one short.

An empty array sums to 0 and matches 0 times. `largest` is only called on
non-empty arrays.

## Starter
```java
static int sumAll(int[] a) {
    int total = 0;
    for (int i = 0; i <= a.length; i++) {   // one too far
        total += a[i];
    }
    return total;
}

static int largest(int[] a) {
    int best = 0;                            // assumes a positive is present
    for (int i = 1; i < a.length; i++) {
        if (a[i] > best) best = a[i];
    }
    return best;
}

static int countMatches(int[] a, int target) {
    int count = 0;
    for (int i = 0; i < a.length - 1; i++) { // stops one short
        if (a[i] == target) count++;
    }
    return count;
}
```

## Tests
```java
checkEq(sumAll(new int[]{1, 2, 3}), 6);
checkEq(sumAll(new int[]{}), 0);
checkEq(sumAll(new int[]{-4}), -4);

checkEq(largest(new int[]{3, 9, 2}), 9);
checkEq(largest(new int[]{7}), 7);
checkEq(largest(new int[]{-5, -2, -9}), -2);
checkEq(largest(new int[]{0, -1}), 0);

checkEq(countMatches(new int[]{1, 2, 2, 3}, 2), 2);
checkEq(countMatches(new int[]{5, 5, 5}, 5), 3);
checkEq(countMatches(new int[]{}, 1), 0);
checkEq(countMatches(new int[]{4}, 4), 1);
```

## Hints
- Valid indices run from `0` to `a.length - 1`, so the loop condition is `i < a.length` — not `<=`, and not `< a.length - 1`.
- `largest` starting at 0 quietly assumes some element is positive. The all-negative test is what exposes it.
- Start `largest` from the array's own first element: `int best = a[0];` and loop from `i = 1`.
- `countMatches` never looks at the final element. Same fix as `sumAll`.

## Solution
```java
static int sumAll(int[] a) {
    int total = 0;
    for (int i = 0; i < a.length; i++) {
        total += a[i];
    }
    return total;
}

static int largest(int[] a) {
    int best = a[0];                 // the array supplies its own starting point
    for (int i = 1; i < a.length; i++) {
        if (a[i] > best) best = a[i];
    }
    return best;
}

static int countMatches(int[] a, int target) {
    int count = 0;
    for (int i = 0; i < a.length; i++) {
        if (a[i] == target) count++;
    }
    return count;
}
```

## Notes
All three bugs are the same mistake at different edges, and the fix is the same
sentence: **valid indices are `0` through `length - 1`**, so the condition is
`i < a.length`.

`largest` is the one worth dwelling on, because it does not throw. Seeding
`best = 0` is a guess that some element exceeds zero, and on `{-5, -2, -9}` the
method confidently returns `0` — a value that is not in the array at all. A bug
that returns a plausible number is worse than one that crashes, because nothing
draws attention to it.

Seeding from `a[0]` removes the guess entirely: the starting value is
guaranteed to be a real element, so the answer is always one of the inputs.
That is why the loop then starts at `i = 1` — element zero has already been
counted.
