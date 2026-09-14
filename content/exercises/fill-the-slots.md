---
id: fill-the-slots
title: "Arrays of objects start empty"
difficulty: intro
chapter: references
topics: [references, arrays, null]
check: unit
standard: java21
---

`new String[n]` creates the array and nothing else. Every slot holds `null`
until something is put in it, and reaching through a `null` throws.

- `repeated(word, n)` — a `String[]` of length `n` with `word` in every slot.
- `totalLength(words)` — the sum of the lengths of the strings in the array,
  **skipping any null slots** rather than throwing.
- `firstNonNull(words)` — the first element that is not null, or `null` if every
  slot is empty or the array has no elements.

`n` is zero or more. `words` is never itself null, but its elements may be.

## Starter
```java
static String[] repeated(String word, int n) {
    String[] out = new String[n];
    return out;                          // every slot is still null
}

static int totalLength(String[] words) {
    int total = 0;
    for (int i = 0; i < words.length; i++) {
        total += words[i].length();      // throws on the first null
    }
    return total;
}

static String firstNonNull(String[] words) {
    return words[0];                     // assumes there is a slot, and that it is filled
}
```

## Tests
```java
String[] three = repeated("ab", 3);
checkEq(three.length, 3);
checkEq(three[0], "ab");
checkEq(three[2], "ab");
checkEq(repeated("x", 0).length, 0);

checkEq(totalLength(new String[]{"a", "bc", "def"}), 6);
checkEq(totalLength(new String[]{"a", null, "def"}), 4);
checkEq(totalLength(new String[3]), 0);
checkEq(totalLength(new String[]{}), 0);

checkEq(firstNonNull(new String[]{"a", "b"}), "a");
checkEq(firstNonNull(new String[]{null, null, "c"}), "c");
checkEq(firstNonNull(new String[3]), null);
checkEq(firstNonNull(new String[]{}), null);
```

## Hints
- Allocating the array is the first of two steps. The second is a loop that puts something in each slot.
- `words[i].length()` reaches *through* the reference in slot `i`. If that slot is null there is nothing to reach.
- Guard with `if (words[i] != null)` before touching the element, not after.
- `firstNonNull` has two ways to find nothing: an empty array, and an array of all nulls. Returning `null` at the end covers both, because a loop over an empty array simply does not run.

## Solution
```java
static String[] repeated(String word, int n) {
    String[] out = new String[n];
    for (int i = 0; i < n; i++) {
        out[i] = word;
    }
    return out;
}

static int totalLength(String[] words) {
    int total = 0;
    for (int i = 0; i < words.length; i++) {
        if (words[i] != null) {
            total += words[i].length();
        }
    }
    return total;
}

static String firstNonNull(String[] words) {
    for (int i = 0; i < words.length; i++) {
        if (words[i] != null) {
            return words[i];
        }
    }
    return null;
}
```

## Notes
**Two allocations, not one.** `new String[n]` makes the array; the loop fills it.
The starter does the first and forgets the second, which is the most common way
a free-response answer produces a `NullPointerException`.

Compare with `int[]`: `new int[3]` is immediately usable because its slots hold
`0`, a perfectly good number. There is no equivalent default object, so object
slots hold `null` — a reference to nothing — and stay unusable until filled.

**`totalLength`** shows what `null` costs. `words[i].length()` follows the
reference in slot `i` to a `String` and asks its length; when the slot holds
`null` there is no string to ask, and that is the exception. Note where the
guard goes — before the dereference. Checking afterwards is checking after the
program has already died.

**`firstNonNull`** returns `null` after the loop, and that single line handles
both empty cases. A loop over a zero-length array runs zero times and falls
straight through, so "no elements" and "no non-null elements" need no separate
branch. Reaching for a special case there is a good instinct and, here, wasted
work.
