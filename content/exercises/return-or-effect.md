---
id: return-or-effect
title: "Does it return, or does it change something?"
difficulty: intro
chapter: calling-methods
topics: [methods, void, return]
check: unit
standard: java21
---

Every method does one of two things for its caller: hands back a value, or
changes something the caller can see. Mixing the two up is where the marks go.

- `sumOf(a)` — return the total of an `int[]`, changing nothing.
- `scaleInPlace(a, k)` — multiply every element by `k`, returning nothing. The
  caller's array must change.
- `withoutNegatives(a)` — return a **new** array holding only the
  non-negative elements, in order, leaving `a` untouched.

`a` is never null and may be empty.

## Starter
```java
static int sumOf(int[] a) {
    int total = 0;
    for (int v : a) total += v;
    a[0] = total;                    // changes the caller's array — it must not
    return total;
}

static void scaleInPlace(int[] a, int k) {
    int[] scaled = new int[a.length];
    for (int i = 0; i < a.length; i++) scaled[i] = a[i] * k;
    // scaled is dropped, so the caller sees nothing
}

static int[] withoutNegatives(int[] a) {
    for (int i = 0; i < a.length; i++) {
        if (a[i] < 0) a[i] = 0;      // edits in place, and keeps the wrong count
    }
    return a;
}
```

## Tests
```java
int[] s = {1, 2, 3};
checkEq(sumOf(s), 6);
checkEq(s[0], 1);                    // sumOf must not touch the array
checkEq(sumOf(new int[]{}), 0);
checkEq(sumOf(new int[]{-4}), -4);

int[] m = {1, 2, 3};
scaleInPlace(m, 10);
checkEq(m[0], 10);
checkEq(m[2], 30);
int[] empty = {};
scaleInPlace(empty, 5);
checkEq(empty.length, 0);

int[] mixed = {3, -1, 0, -7, 5};
int[] kept = withoutNegatives(mixed);
checkEq(kept.length, 3);
checkEq(kept[0], 3);
checkEq(kept[1], 0);
checkEq(kept[2], 5);
checkEq(mixed.length, 5);            // the original is untouched
checkEq(mixed[1], -1);
checkEq(withoutNegatives(new int[]{}).length, 0);
checkEq(withoutNegatives(new int[]{-1, -2}).length, 0);
```

## Hints
- `sumOf` returns a value and should leave the array exactly as it found it. The starter's `a[0] = total` writes through the reference and the caller sees it.
- `scaleInPlace` returns `void`, so the *only* way it can matter is by writing through the reference it was given. Building a new array and dropping it achieves nothing.
- `withoutNegatives` returns a new array, so nothing it does may touch `a`. Zeroing negatives in place fails twice: it edits the original, and zero is not the same as removed.
- Two passes is the simplest way: count the non-negatives first, allocate an array of exactly that size, then fill it.

## Solution
```java
static int sumOf(int[] a) {
    int total = 0;
    for (int v : a) total += v;
    return total;
}

static void scaleInPlace(int[] a, int k) {
    for (int i = 0; i < a.length; i++) {
        a[i] = a[i] * k;
    }
}

static int[] withoutNegatives(int[] a) {
    int count = 0;
    for (int v : a) {
        if (v >= 0) count++;
    }
    int[] out = new int[count];
    int j = 0;
    for (int v : a) {
        if (v >= 0) {
            out[j] = v;
            j++;
        }
    }
    return out;
}
```

## Notes
The three methods are the three honest shapes, and each starter breaks its own
contract in a different direction.

**`sumOf` returns and must not change.** Writing `a[0] = total` is a *side
effect*: the caller asked a question and had their data edited. The test that
catches it reads `s[0]` afterwards, and that is the kind of check worth writing
whenever a method is supposed to be read-only.

**`scaleInPlace` changes and returns nothing.** With `void` there is no channel
back except the reference, so building a local array is work the caller can
never see — the same "rebinding does nothing" lesson as chapter 1.2's `replace`.

**`withoutNegatives` returns new and must not change.** The starter zeroes
negatives where they sit, which fails both halves: `mixed` is modified, and the
result is still length 5 because a zero is present rather than removed. You
cannot shrink an array in place; a different length means a different array.

Two passes are needed because the size has to be known before `new int[count]`.
Counting first is not wasted effort — it is what makes one allocation possible
instead of copying repeatedly.
