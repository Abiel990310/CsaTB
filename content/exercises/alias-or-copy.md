---
id: alias-or-copy
title: "Alias or copy — say which, then build one"
difficulty: intro
chapter: references
topics: [references, arrays]
check: unit
standard: java21
---

Three methods about the difference between two names for one array and two
arrays.

- `duplicate(a)` — return a **new** array with the same contents. Changing the
  result must not touch `a`.
- `sharesStorageWith(a, b)` — `true` when the two variables refer to the same
  array object, `false` otherwise. Equal contents in two separate arrays is
  `false`.
- `bumpFirst(a)` — add 1 to element 0 of the caller's array, in place, and
  return nothing.

`duplicate` and `bumpFirst` are only called on non-empty arrays.
`sharesStorageWith` may receive `null` for either argument; two `null`s share
nothing.

## Starter
```java
static int[] duplicate(int[] a) {
    return a;                    // hands back the same array
}

static boolean sharesStorageWith(int[] a, int[] b) {
    return a.length == b.length; // compares the wrong thing entirely
}

static void bumpFirst(int[] a) {
    int[] local = new int[a.length];
    for (int i = 0; i < a.length; i++) local[i] = a[i];
    local[0] = local[0] + 1;     // writes to a copy nobody else can see
}
```

## Tests
```java
int[] original = {1, 2, 3};
int[] copy = duplicate(original);
checkEq(copy[0], 1);
checkEq(copy.length, 3);
copy[0] = 99;
checkEq(original[0], 1);                       // the copy must be independent
original[1] = 50;
checkEq(copy[1], 2);                           // in both directions

checkEq(sharesStorageWith(original, original), true);
int[] alias = original;
checkEq(sharesStorageWith(original, alias), true);
checkEq(sharesStorageWith(original, duplicate(original)), false);
checkEq(sharesStorageWith(new int[]{1, 2}, new int[]{1, 2}), false);
checkEq(sharesStorageWith(new int[]{1, 2}, new int[]{9, 9}), false);
checkEq(sharesStorageWith(null, null), false);
checkEq(sharesStorageWith(original, null), false);

int[] target = {10, 20};
bumpFirst(target);
checkEq(target[0], 11);
bumpFirst(target);
checkEq(target[0], 12);
checkEq(target[1], 20);
```

## Hints
- `return a;` returns the reference it was given, so the caller gets a second name for the array it already had. A real copy starts with `new int[a.length]`.
- Equal contents and equal identity are different questions. `==` on two array variables asks the identity one: are these the same object?
- Comparing lengths says nothing about identity — two unrelated arrays of the same size would pass.
- `bumpFirst` builds a copy, edits the copy, and drops it. To change the caller's array, write through the reference you were handed: `a[0]`.

## Solution
```java
static int[] duplicate(int[] a) {
    int[] out = new int[a.length];
    for (int i = 0; i < a.length; i++) {
        out[i] = a[i];
    }
    return out;
}

static boolean sharesStorageWith(int[] a, int[] b) {
    if (a == null || b == null) return false;
    return a == b;
}

static void bumpFirst(int[] a) {
    a[0] = a[0] + 1;
}
```

## Notes
**`duplicate`** is the whole chapter in three lines. `return a;` compiles, runs,
and looks like it returns a copy — and the test that catches it is the one that
writes to the result and then reads the original. Only `new` makes an array.

**`sharesStorageWith`** is the identity question, and `==` on two reference
variables is exactly how you ask it: it compares the references, not the
contents. That is why `sharesStorageWith(new int[]{1,2}, new int[]{1,2})` is
`false` — same numbers, two objects.

Hold on to that. In chapter 2.5 the same `==` on two `String` variables asks the
same identity question, surprises everyone, and is on the exam every year. It is
not a special rule about strings; it is this rule, applied to strings.

The `null` guard comes first because `a == b` would be `true` for two `null`s,
and "both refer to nothing" is not "both refer to the same array".

**`bumpFirst`** returns nothing, so writing through the reference is the *only*
way it can have any effect. The starter's copy-edit-discard is what a method
does when it reassigns instead of writing through — the work happens, and the
caller never sees it.
