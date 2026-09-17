---
id: read-the-signature
title: "Call it correctly, from the signature alone"
difficulty: intro
chapter: calling-methods
topics: [methods, signatures]
check: unit
standard: java21
---

You are given a `Toolbox` class. Do not change it — reading its three
signatures is the whole exercise, and it is what the exam asks you to do with
classes you have never seen.

```java
static int    doubled(int n)            // static: call it on the class
static String repeat(String s, int n)   // static: returns a NEW string
int           bump(int by)              // no static: call it on an object
```

Write three methods that use it.

- `quadruple(n)` — four times `n`, using **only** `Toolbox.doubled`.
- `banner(word)` — the word three times, then `"!"`, so `banner("ab")` is
  `"ababab!"`.
- `bumpTwice(box, a, b)` — bump that box by `a`, then by `b`, and return the
  total it reports the second time.

## Starter
```java
// Given to you. Leave it alone.
static class Toolbox {
    static int    doubled(int n)          { return 2 * n; }
    static String repeat(String s, int n) { return s.repeat(Math.max(0, n)); }
    int total = 0;
    int bump(int by) { total += by; return total; }
}

static int quadruple(int n) {
    Toolbox.doubled(n);              // the answer goes nowhere
    Toolbox.doubled(n);
    return n;
}

static String banner(String word) {
    Toolbox.repeat(word, 3);         // same mistake, with a String
    return word + "!";
}

static int bumpTwice(Toolbox box, int a, int b) {
    Toolbox.bump(a);                 // bump is not static
    return box.bump(b);
}
```

## Tests
```java
checkEq(quadruple(3), 12);
checkEq(quadruple(0), 0);
checkEq(quadruple(-2), -8);

checkEq(banner("ab"), "ababab!");
checkEq(banner("x"), "xxx!");
checkEq(banner(""), "!");

Toolbox t1 = new Toolbox();
checkEq(bumpTwice(t1, 2, 3), 5);
checkEq(t1.total, 5);

Toolbox t2 = new Toolbox();
checkEq(bumpTwice(t2, 10, -4), 6);
checkEq(bumpTwice(t2, 1, 1), 8);
```

## Hints
- `Toolbox.doubled(n)` on a line by itself computes an answer and throws it away. Doubling twice means feeding the first answer into the second call.
- `repeat` hands back the repeated string. It does not modify `word`, and could not.
- `bump` has no `static` in its signature, so it belongs to an object. Call it on the `box` you were handed.
- Read what `bumpTwice` must return: the total after the *second* bump, which is exactly what the second call gives back.

## Solution
```java
// Given to you. Leave it alone.
static class Toolbox {
    static int    doubled(int n)          { return 2 * n; }
    static String repeat(String s, int n) { return s.repeat(Math.max(0, n)); }
    int total = 0;
    int bump(int by) { total += by; return total; }
}

static int quadruple(int n) {
    return Toolbox.doubled(Toolbox.doubled(n));
}

static String banner(String word) {
    return Toolbox.repeat(word, 3) + "!";
}

static int bumpTwice(Toolbox box, int a, int b) {
    box.bump(a);
    return box.bump(b);
}
```

## Notes
**`quadruple`** is the return-value lesson twice over. Each `Toolbox.doubled(n)`
on its own line runs, computes, and discards. Nesting the calls connects them:
the inner one's answer becomes the outer one's argument.

**`banner`** is the same mistake with a `String`, and it is worth meeting here
because chapter 1.4 is full of it. `repeat` hands back a new string; the
original is untouched, and dropping the result loses the only copy of the work.

**`bumpTwice`** is static against instance. `bump` has no `static`, so
`Toolbox.bump(a)` does not compile — there is no object for it to run on. It
has to be called on `box`.

Notice what the tests check afterwards: `t1.total` is 5, because `bump` **does**
change the object even on the call whose return we ignored. Discarding a return
throws away the answer, not the effect — and the two `t2` lines confirm the
totals keep accumulating across calls.
