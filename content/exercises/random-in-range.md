---
id: random-in-range
title: "Pick a number, both ends included"
difficulty: intro
chapter: math-class
topics: [math, random, casting]
check: unit
standard: java21
---

Write `pick(min, max)`, returning a random `int` from `min` to `max` with
**both ends possible**. You may assume `min <= max`.

The checks do not test individual results — they cannot, since the answer is
different every call. Instead they call `pick` two hundred thousand times and
test the **range**: every result must land inside `[min, max]`, and both `min`
and `max` must actually turn up. That second half is the one that catches the
missing `+ 1`.

`pick(n, n)` must return `n`.

## Starter
```java
static int pick(int min, int max) {
    return min;                       // always in range, and never anything else
}
```

## Tests
```java
boolean inRange = true;
int lo = Integer.MAX_VALUE;
int hi = Integer.MIN_VALUE;
for (int i = 0; i < 200_000; i++) {
    int v = pick(3, 8);
    if (v < 3 || v > 8) inRange = false;
    lo = Math.min(lo, v);
    hi = Math.max(hi, v);
}
check(inRange);
checkEq(lo, 3);
checkEq(hi, 8);

boolean singleOk = true;
for (int i = 0; i < 1_000; i++) {
    if (pick(4, 4) != 4) singleOk = false;
}
check(singleOk);

boolean negOk = true;
int negLo = Integer.MAX_VALUE;
int negHi = Integer.MIN_VALUE;
for (int i = 0; i < 200_000; i++) {
    int v = pick(-5, -1);
    if (v < -5 || v > -1) negOk = false;
    negLo = Math.min(negLo, v);
    negHi = Math.max(negHi, v);
}
check(negOk);
checkEq(negLo, -5);
checkEq(negHi, -1);
```

## Hints
- `Math.random()` gives you `0.0` up to but not including `1.0`. You need whole
  numbers, so something has to stretch that range and something has to truncate
  it.
- How many integers are there from `min` to `max` inclusive? It is not
  `max - min`. Count them from 1 to 6 if you are unsure.
- Multiply first, then cast, then add. Casting `Math.random()` on its own gives
  `0` every time, because the value is always below 1.
- The starter passes the "inside the range" test and fails the "reaches both
  ends" one. That is exactly the shape of the bug this problem is about.

## Solution
```java
static int pick(int min, int max) {
    return (int)(Math.random() * (max - min + 1)) + min;
}
```

## Notes
The starter is worth looking at before the solution. `return min;` is a
perfectly valid random number generator by the loosest reading of the prompt —
every value it returns is in range — and the only test it fails is the one
asking whether the top of the range is reachable.

That is not a contrived check. **A range bug does not throw and does not look
wrong**; it produces plausible values forever, with one of them silently
missing. Testing a random function by its range rather than its values is the
only way to see it, which is why the checks are written the way they are.

The negative case is there because `max - min + 1` is the count of values and
counts are positive regardless of sign: from `-5` to `-1` is `-1 - (-5) + 1`,
which is 5. A solution that special-cases negatives has misread the formula.

Two hundred thousand trials is far more than needed. Missing one face of a
six-sided die that many times has probability `(5/6)^200000`, which is not a
small number so much as a number with thirty thousand zeros after the decimal
point. The check is deterministic in every way that matters.
