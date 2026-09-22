---
id: math-return-types
title: "Getting an int back out of Math"
difficulty: intro
chapter: math-class
topics: [math, casting, types]
check: unit
standard: java21
---

`Math.pow` and `Math.sqrt` both hand back a `double`, and both of these methods
have to return an `int`. Put the cast where it belongs.

- `cube(n)` — return `n` cubed, computed with `Math.pow`.
- `rootFloor(n)` — return the largest whole number whose square is at most `n`.
  `n` is never negative.
- `absOverflows(n)` — return `true` when `Math.abs(n)` comes back **negative**,
  and `false` otherwise.

That last one has exactly one input for which it is `true`. Finding it is the
problem.

## Starter
```java
static int cube(int n) {
    return 0;
}

static int rootFloor(int n) {
    return 0;
}

static boolean absOverflows(int n) {
    return false;
}
```

## Tests
```java
checkEq(cube(0), 0);
checkEq(cube(1), 1);
checkEq(cube(2), 8);
checkEq(cube(5), 125);
checkEq(cube(-3), -27);
checkEq(cube(10), 1000);

checkEq(rootFloor(0), 0);
checkEq(rootFloor(1), 1);
checkEq(rootFloor(2), 1);
checkEq(rootFloor(15), 3);
checkEq(rootFloor(16), 4);
checkEq(rootFloor(17), 4);
checkEq(rootFloor(99), 9);
checkEq(rootFloor(100), 10);
checkEq(rootFloor(10000), 100);

check(absOverflows(Integer.MIN_VALUE));
check(!absOverflows(Integer.MAX_VALUE));
check(!absOverflows(0));
check(!absOverflows(-1));
check(!absOverflows(-2147483647));
```

## Hints
- Both `Math.pow` and `Math.sqrt` return a `double`, and a `double` will not go
  into an `int` without you saying so. `javac` calls it a *possible lossy
  conversion*.
- `(int)` truncates towards zero. For `rootFloor` that is exactly the wanted
  behaviour and no extra work is needed — `Math.sqrt(15)` is about 3.873.
- `absOverflows` does not need arithmetic of its own. Call `Math.abs` and look
  at the sign of what comes back.
- An `int` runs from `-2147483648` to `2147483647`. One of those two has no
  positive counterpart.

## Solution
```java
static int cube(int n) {
    return (int) Math.pow(n, 3);
}

static int rootFloor(int n) {
    return (int) Math.sqrt(n);
}

static boolean absOverflows(int n) {
    return Math.abs(n) < 0;
}
```

## Notes
`cube` and `rootFloor` are the same one-line shape, and the difference between
them is worth naming. In `cube` the cast throws nothing away — `Math.pow(5, 3)`
is exactly `125.0` — so it exists purely to satisfy the type system. In
`rootFloor` the cast is doing real work: truncation *is* the answer, which is
why no rounding or adjustment belongs anywhere near it.

`absOverflows` looks like a trick question and is not. `Math.abs(n) < 0` reads
as a contradiction — an absolute value is a magnitude, and magnitudes are not
negative — but it is `true` for `Integer.MIN_VALUE`, and the reason is the
asymmetry of two's complement. There is one more negative `int` than positive,
so `-2147483648` has no positive counterpart. `Math.abs` computes it anyway,
the result wraps, and the value returns unchanged.

The check on `-2147483647` is the one that matters. It is next door to the
overflowing value and behaves perfectly, which is what makes this worth a test
rather than a note: a guard written as `n < -1000000000` would pass four of
these five checks.
