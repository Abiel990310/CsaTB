---
id: find-and-split
title: "indexOf returns −1, and 0 is a real answer"
difficulty: core
chapter: strings
topics: [strings, indexOf]
check: unit
standard: java21
---

`indexOf` reports a position, or `-1` when there is none. Zero is a position —
the first one — so the two have to be told apart carefully.

- `before(s, marker)` — everything in `s` before the first `marker`. If the
  marker is absent, return `s` unchanged. If it is at the very start, return
  `""`.
- `after(s, marker)` — everything after the first `marker`. If absent, return
  `""`.
- `countOf(s, ch)` — how many times the one-character string `ch` appears.
- `stripPrefix(s, p)` — `s` with `p` removed from the front, but **only** if `s`
  actually starts with `p`. Otherwise `s` unchanged.

`marker`, `ch` and `p` are never empty.

## Starter
```java
static String before(String s, String marker) {
    int i = s.indexOf(marker);
    if (i <= 0) return s;            // wrongly treats "found at 0" as "absent"
    return s.substring(0, i);
}

static String after(String s, String marker) {
    int i = s.indexOf(marker);
    return s.substring(i + 1);       // no guard, and the wrong offset
}

static int countOf(String s, String ch) {
    int n = 0;
    for (int i = 0; i < s.length(); i++) {
        if (s.substring(i, i + 1) == ch) n++;   // compares references
    }
    return n;
}

static String stripPrefix(String s, String p) {
    return s.substring(p.length());  // cuts even when the prefix is not there
}
```

## Tests
```java
checkEq(before("key=value", "="), "key");
checkEq(before("=value", "="), "");          // found at 0, not absent
checkEq(before("novalue", "="), "novalue");
checkEq(before("a=b=c", "="), "a");

checkEq(after("key=value", "="), "value");
checkEq(after("novalue", "="), "");
checkEq(after("key=", "="), "");
checkEq(after("a->b", "->"), "b");           // marker longer than one character

checkEq(countOf("banana", "a"), 3);
checkEq(countOf("banana", "z"), 0);
checkEq(countOf("", "a"), 0);
checkEq(countOf("aaa", "a"), 3);

checkEq(stripPrefix("unhappy", "un"), "happy");
checkEq(stripPrefix("happy", "un"), "happy");
checkEq(stripPrefix("un", "un"), "");
checkEq(stripPrefix("u", "un"), "u");        // shorter than the prefix
```

## Hints
- `i <= 0` lumps together two different answers: `-1` means absent, `0` means found at the very start. Test for `-1` exactly.
- In `after`, skipping past the marker means advancing by its **length**, not by 1 — otherwise a two-character marker leaves half of itself behind.
- `==` on two strings compares references, not characters. Use `.equals`.
- `stripPrefix` has to check before it cuts. `startsWith` answers exactly that question, and it is safe even when `s` is shorter than `p`.

## Solution
```java
static String before(String s, String marker) {
    int i = s.indexOf(marker);
    if (i == -1) return s;
    return s.substring(0, i);
}

static String after(String s, String marker) {
    int i = s.indexOf(marker);
    if (i == -1) return "";
    return s.substring(i + marker.length());
}

static int countOf(String s, String ch) {
    int n = 0;
    for (int i = 0; i < s.length(); i++) {
        if (s.substring(i, i + 1).equals(ch)) n++;
    }
    return n;
}

static String stripPrefix(String s, String p) {
    if (!s.startsWith(p)) return s;
    return s.substring(p.length());
}
```

## Notes
**`before`** is why the guard must be `== -1`. Writing `i <= 0` folds "absent"
together with "found at position 0", and the test `before("=value", "=")`
separates them: the marker *is* there, at the front, so the answer is the empty
string — not the whole input. Treating a valid index as a failure is the
mirror image of forgetting to check at all.

**`after`** has two bugs and they are independent. Without the `-1` guard,
`substring(-1 + 1)` happens to return the whole string rather than throwing,
which is worse than a crash — the wrong answer looks plausible. And advancing by
`1` instead of `marker.length()` only shows up once the marker is longer than
one character, which is what `after("a->b", "->")` is for.

**`countOf`** uses `==` on strings, which compares whether two references point
at the same object rather than whether the characters match. Here the
substrings are freshly built, so they are never the same object and the count is
always 0. `.equals` asks the question that was meant. Chapter 2.5 takes this up
properly; it is listed here because it is impossible to avoid once you start
comparing strings.

**`stripPrefix`** must check before cutting. `startsWith` is the right tool and
is safe even when `s` is shorter than `p` — it simply answers `false`, where a
bare `substring(p.length())` would throw, which is what the `"u"` test pins
down.
