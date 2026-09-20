---
id: string-surgery
title: "Four string methods, used correctly"
difficulty: intro
chapter: strings
topics: [strings, immutability]
check: unit
standard: java21
---

Four small methods. Each starter makes one of the two mistakes this chapter is
about: discarding a returned string, or getting `substring`'s end index wrong.

- `shout(s)` — `s` in upper case with `"!"` on the end.
- `firstTwo(s)` — the first two characters. If `s` is shorter than two
  characters, return all of it.
- `middle(s)` — `s` with its first and last characters removed. For a string of
  length 2 or less, return `""`.
- `initials(first, last)` — the first letter of each, upper case, no separator.
  So `initials("ada", "lovelace")` is `"AL"`.

Neither `first` nor `last` is empty.

## Starter
```java
static String shout(String s) {
    s.toUpperCase();                 // computed, then thrown away
    return s + "!";
}

static String firstTwo(String s) {
    return s.substring(0, 2);        // throws when s is shorter than 2
}

static String middle(String s) {
    return s.substring(1, s.length() - 1 + 1);   // the end index is one too far
}

static String initials(String first, String last) {
    return first.substring(0, 1) + last.substring(0, 1);   // never upper-cased
}
```

## Tests
```java
checkEq(shout("hey"), "HEY!");
checkEq(shout(""), "!");
checkEq(shout("Mixed Case"), "MIXED CASE!");

checkEq(firstTwo("abcdef"), "ab");
checkEq(firstTwo("ab"), "ab");
checkEq(firstTwo("a"), "a");
checkEq(firstTwo(""), "");

checkEq(middle("abcde"), "bcd");
checkEq(middle("abc"), "b");
checkEq(middle("ab"), "");
checkEq(middle("a"), "");
checkEq(middle(""), "");

checkEq(initials("ada", "lovelace"), "AL");
checkEq(initials("Grace", "Hopper"), "GH");
checkEq(initials("x", "y"), "XY");
```

## Hints
- `toUpperCase` hands back a new string and leaves `s` alone. Use what it returns.
- `firstTwo` needs a length check before it cuts. `Math.min` turns two cases into one.
- For `middle`, the last character is at index `length - 1`, and `substring`'s end index is **excluded** — so that index is exactly the end you want.
- `initials` can upper-case the single-character substring, or upper-case the whole name first. Either works; neither happens by itself.

## Solution
```java
static String shout(String s) {
    return s.toUpperCase() + "!";
}

static String firstTwo(String s) {
    return s.substring(0, Math.min(2, s.length()));
}

static String middle(String s) {
    if (s.length() <= 2) return "";
    return s.substring(1, s.length() - 1);
}

static String initials(String first, String last) {
    return (first.substring(0, 1) + last.substring(0, 1)).toUpperCase();
}
```

## Notes
**`shout`** is the chapter in one line. The starter calls `toUpperCase`, which
does all the work and returns a new string, and then ignores it and appends to
the original. Nothing warns you.

**`firstTwo`** shows why a length check comes before a cut. `substring(0, 2)` on
`"a"` throws, because index 2 is past the end of a one-character string.
`Math.min(2, s.length())` collapses the short and long cases into one
expression — and note it handles `""` too, where the result is `substring(0, 0)`
and correctly empty.

**`middle`** is the end-index convention. The last character sits at
`length - 1`, and since `substring` **excludes** its end index, passing
`length - 1` stops just before it — which is exactly what "drop the last
character" means. The starter's `- 1 + 1` is someone reasoning as though the
end were inclusive, and it returns the last character instead of dropping it.

The guard matters at length 2: `substring(1, 1)` would be empty anyway, but at
length 1 the call would be `substring(1, 0)`, and an end index below the start
throws.

**`initials`** is only a trap if you assume a method changed something. Both
substrings are fresh strings; upper-casing has to be asked for.
