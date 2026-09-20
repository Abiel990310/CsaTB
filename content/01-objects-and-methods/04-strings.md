---
title: "String, and the methods the exam actually tests"
navTitle: "String"
summary: >-
  Strings cannot be changed. Every method hands you a new one, which makes the
  discarded-return mistake from 1.3 the defining error of this topic.
objectives:
  - Explain what immutability means for a String variable
  - Use length, substring, indexOf and equals correctly
  - Get substring's two index conventions right
  - Handle indexOf returning -1
status: complete
standard: java21
requires: [calling-methods]
---

The exam tests a short list of `String` methods, and it tests them hard. The
list is worth learning; the one idea behind all of them is worth learning first.

**A `String` cannot be changed.** Not "should not" — cannot. There is no method
anywhere that modifies one. Every method that appears to alter a string actually
builds a **new** string and returns it, which makes chapter 1.3's discarded
return the defining mistake of this whole topic.

```java run title="The mistake, and the fix"
public class Main {
    public static void main(String[] args) {
        String s = "hello";

        s.toUpperCase();                  // builds "HELLO" and throws it away
        System.out.println(s);            // hello — unchanged

        s = s.toUpperCase();              // keep the new string
        System.out.println(s);            // HELLO
    }
}
```

Nothing warns you about line 5. It compiles, it runs, it computes `"HELLO"`, and
then the value goes nowhere — exactly as `twice(x)` did in 1.3. The difference
is that with strings it *looks* like a command.

## The methods

| Method | Returns | Note |
|---|---|---|
| `s.length()` | `int` | a **method**, with parentheses |
| `s.substring(a)` | `String` | from index `a` to the end |
| `s.substring(a, b)` | `String` | from `a` **up to but not including** `b` |
| `s.indexOf(t)` | `int` | first position of `t`, or **−1** if absent |
| `s.equals(t)` | `boolean` | same characters? |
| `s.compareTo(t)` | `int` | negative, zero or positive |

```java run title="All six, on one string"
public class Main {
    public static void main(String[] args) {
        String s = "calculus";

        System.out.println(s.length());          // 8
        System.out.println(s.substring(3));      // culus
        System.out.println(s.substring(0, 4));   // calc
        System.out.println(s.indexOf("cul"));    // 3
        System.out.println(s.indexOf("z"));      // -1
        System.out.println(s.equals("calculus"));// true
        System.out.println(s.compareTo("call")); // negative: "calc" < "call"
    }
}
```

`compareTo` is worth one sentence: it compares character by character and
returns the difference at the first position where they differ. The exam only
ever asks about the **sign**, so read it as "negative means the receiver comes
first alphabetically".

:::warning
`length` is a **method** on a `String` and a **field** on an array.

```java
String s = "abc";
int[] a = {1, 2, 3};
s.length()     // parentheses
a.length       // none
```

They are different things that happen to share a name, and writing `s.length`
or `a.length()` does not compile. Chapter 4.1 meets the array form.
:::

## substring's two conventions

`substring(a, b)` takes characters from index `a` **up to but not including**
`b`. That is the single most tested detail in this chapter.

```java run title="Half-open, and what falls out of it"
public class Main {
    public static void main(String[] args) {
        String s = "abcdef";

        System.out.println(s.substring(1, 4));   // bcd — not bcde
        System.out.println(s.substring(2, 2));   // empty, not "c"
        System.out.println(s.substring(6));      // empty, not an error
        System.out.println(s.substring(1, 4).length()); // 3 = 4 - 1
    }
}
```

Three consequences worth holding on to:

- **The length of the result is `b - a`.** That subtraction is exact only
  because the end is excluded, and it is the quickest way to check yourself.
- **`substring(i, i)` is empty**, since it asks for zero characters.
- **`substring(s.length())` is empty and legal.** Starting at the very end is
  allowed; starting past it is not.

```java run expect-throw title="One past the end is too far"
public class Main {
    public static void main(String[] args) {
        String s = "abcdef";
        System.out.println(s.substring(7));    // length is 6
    }
}
```

That is verified to throw a `StringIndexOutOfBoundsException`. Index `6` is the
boundary and is fine; `7` is past it.

## indexOf returns −1

When the text is not there, `indexOf` gives **−1**, not zero. Zero is a
perfectly good answer meaning "found at the very start", so the two must be
distinguished.

```java run title="Guard before you use it"
public class Main {
    static String after(String s, String marker) {
        int i = s.indexOf(marker);
        if (i == -1) {
            return "";                       // not present
        }
        return s.substring(i + marker.length());
    }

    public static void main(String[] args) {
        System.out.println(after("key=value", "="));   // value
        System.out.println(after("keyvalue", "="));    // (empty)
        System.out.println(after("=start", "="));      // start
    }
}
```

Feeding an unchecked `-1` into `substring` throws, so the guard is not optional.
Note the third case: the marker at index `0` is found, and `0` is not `-1`, so
the guard correctly lets it through — which is precisely why the test is
`== -1` rather than a falsiness check or `<= 0`.

## Concatenation

`+` joins strings, and if **either** side is a `String` the result is a
`String`. That interacts with left-to-right evaluation in a way the exam likes.

```java run title="Same numbers, same operator, different answers"
public class Main {
    public static void main(String[] args) {
        System.out.println("sum: " + 1 + 2);    // sum: 12
        System.out.println(1 + 2 + " total");   // 3 total
        System.out.println("x" + 1 + 2 + "y");  // x12y
        System.out.println(1 + 2 + "" + 3 + 4); // 334
    }
}
```

`+` is left-associative, so it runs left to right. In the first line `"sum: " + 1`
is already a string, so the `2` is appended rather than added. In the second
the `1 + 2` happens first, while both are still numbers, and only then does the
text join on.

The last line is both: `1 + 2` is `3`, the `""` turns it into text, and from
there everything appends.

:::quiz
{
  "question": "What does `\"abcdef\".substring(2, 4)` return?",
  "options": [
    {
      "text": "\"cd\" — indices 2 and 3, with 4 excluded",
      "correct": true,
      "why": "The second index is the stop point, not the last character taken. The result has length 4 − 2 = 2, which is the check worth doing every time."
    },
    {
      "text": "\"cde\", since it runs from index 2 to index 4 inclusive",
      "correct": false,
      "why": "That reads the end index as inclusive, which is the standard error. The result would then have length 3 while 4 − 2 is 2, and the mismatch is the giveaway."
    },
    {
      "text": "\"bc\", counting from index 1",
      "correct": false,
      "why": "String indices start at 0, so index 2 is 'c'. Counting from 1 shifts every answer by one character."
    },
    {
      "text": "\"cdef\", because the second argument is a length",
      "correct": false,
      "why": "The second argument is an end index, not a count. A version taking a length exists in some other languages, which is exactly why this is worth checking rather than assuming."
    }
  ]
}
:::

## Practice

:::exercise string-surgery

:::exercise find-and-split

:::recap
- Strings are **immutable**. Every method returns a new one, so a result you do
  not keep is lost — the 1.3 mistake, made easy by methods that read like
  commands.
- `s.length()` is a method; an array's `length` is a field. Neither spelling
  works for the other.
- `substring(a, b)` excludes `b`, so the result has length `b - a`.
  `substring(i, i)` is empty and `substring(s.length())` is legal.
- `indexOf` returns **−1** when absent, and `0` is a real position — guard with
  `== -1`, never with `<= 0`.
- `+` is left-associative, so `"x" + 1 + 2` is `x12` while `1 + 2 + "x"` is `3x`.
:::
