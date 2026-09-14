---
title: "References: what a variable holds when it is not a number"
navTitle: "References"
summary: >-
  Two variables can name the same object, and changing one changes the other.
  Everything surprising about objects on this exam comes from that.
objectives:
  - Say what an object variable actually stores
  - Predict when assigning one variable to another creates an alias
  - Explain why a method can change an object but cannot change which object you have
  - Recognise null, and say where a NullPointerException comes from
status: complete
standard: java21
requires: [primitives-and-arithmetic]
---

Chapter 1.1 was about variables that hold numbers. A variable of primitive type
holds its value, and copying it copies the value — `int b = a;` gives you a
second, independent number.

Objects do not work that way, and almost every question on this exam that looks
like a trick is the same fact wearing a different hat: **an object variable does
not hold the object. It holds a reference to it**, and copying the variable
copies the reference, not the thing.

```java run title="The same fact, twice"
public class Main {
    public static void main(String[] args) {
        int[] a = {1, 2, 3};
        int[] b = a;          // copies the reference, not the array
        b[0] = 99;
        System.out.println("a[0] = " + a[0]);   // 99 — a and b are one array

        int x = 1;
        int y = x;            // copies the value
        y = 99;
        System.out.println("x = " + x);         // 1 — x and y are two numbers
    }
}
```

Nothing about `=` changed between those two halves. What changed is what was
sitting in the variable.

## Two names, one object

When two variables refer to the same object they are **aliases**. There is only
one object; there are two ways to reach it, and a change made through either is
visible through both.

:::memviz
{
  "title": "int[] b = a — one array, two names",
  "steps": [
    {
      "caption": "int[] a = {1, 2, 3}; — the array lives on the heap, a holds a reference to it.",
      "stack": [{ "id": "a", "name": "a", "type": "int[]", "value": "ref", "anchor": "a" }],
      "heap": [{ "id": "arr", "name": "the array", "state": "new", "fields": [
        { "k": "[0]", "v": "1" }, { "k": "[1]", "v": "2" }, { "k": "[2]", "v": "3" }
      ]}],
      "arrows": [{ "from": "a", "to": "arr" }]
    },
    {
      "caption": "int[] b = a; — b gets a copy of the reference. No array is created.",
      "stack": [
        { "id": "a", "name": "a", "type": "int[]", "value": "ref" },
        { "id": "b", "name": "b", "type": "int[]", "value": "ref", "state": "new" }
      ],
      "heap": [{ "id": "arr", "name": "the array", "fields": [
        { "k": "[0]", "v": "1" }, { "k": "[1]", "v": "2" }, { "k": "[2]", "v": "3" }
      ]}],
      "arrows": [{ "from": "a", "to": "arr" }, { "from": "b", "to": "arr" }]
    },
    {
      "caption": "b[0] = 99; — follows b's reference to the one array and writes there.",
      "stack": [
        { "id": "a", "name": "a", "type": "int[]", "value": "ref" },
        { "id": "b", "name": "b", "type": "int[]", "value": "ref" }
      ],
      "heap": [{ "id": "arr", "name": "the array", "state": "danger", "fields": [
        { "k": "[0]", "v": "99" }, { "k": "[1]", "v": "2" }, { "k": "[2]", "v": "3" }
      ]}],
      "arrows": [{ "from": "a", "to": "arr" }, { "from": "b", "to": "arr" }],
      "note": "a[0] is 99 as well, because there is no second array to be unchanged."
    }
  ]
}
:::

The question "does `a` change?" is the wrong question. There is one array. It
changed, and both names see it.

## Copying for real

If you want two independent arrays you have to say so. Assignment will never do
it for you.

```java run title="A copy is a loop, or a library call"
import java.util.Arrays;                       // [hidden]

public class Main {
    public static void main(String[] args) {
        int[] a = {1, 2, 3};

        int[] alias = a;                        // same array
        int[] copy  = new int[a.length];        // a genuinely different array
        for (int i = 0; i < a.length; i++) {
            copy[i] = a[i];
        }

        a[0] = 99;
        System.out.println("alias[0] = " + alias[0]);   // 99
        System.out.println("copy[0]  = " + copy[0]);    // 1
    }
}
```

The loop is what the exam expects you to be able to write. `Arrays.copyOf` does
the same job in one line, but the free-response section asks you to produce the
loop often enough that it is worth having in your fingers.

:::pitfall
**`new` is the only thing that makes an object.** If a line does not contain
`new` (or call something that does, like `Arrays.copyOf` or `substring`), no
new object came into existence on that line, whatever it looks like.

Scan a question for `new`. Count them. That is how many objects there are, and
every other variable is another name for one of them.
:::

## Passing an object to a method

Java passes arguments **by value** — always, with no exceptions. The subtlety is
that when the argument is an object variable, the value being copied is the
reference.

That gives a method exactly one power and denies it another.

```java run title="What a method can and cannot do"
public class Main {
    static void modify(int[] data) {
        data[0] = 99;              // works — follows the reference to the array
    }

    static void replace(int[] data) {
        data = new int[]{7, 7, 7}; // useless — rebinds this method's own variable
    }

    public static void main(String[] args) {
        int[] nums = {1, 2, 3};

        modify(nums);
        System.out.println("after modify:  " + nums[0]);   // 99

        replace(nums);
        System.out.println("after replace: " + nums[0]);   // still 99
    }
}
```

**A method can change the object you handed it.** It cannot change which object
your variable refers to.

`modify` follows the reference and writes to the array that both names point at,
so the caller sees it. `replace` assigns to `data`, which is a local variable
holding a copy of the reference; the copy now points somewhere else and the
caller's variable has not moved. When `replace` returns, its local variable is
gone and the new array is garbage.

:::note
This is why "Java passes objects by reference" is a claim worth refusing, even
though it predicts the first half correctly. It predicts `replace` wrongly. The
accurate statement — the reference is passed by value — predicts both.
:::

## null

A reference variable can hold `null`, meaning *refers to nothing*. It is a
legal value, it is what object fields start as, and touching anything through it
fails.

```java run expect-throw title="The exception you will see most"
public class Main {
    public static void main(String[] args) {
        String[] names = new String[3];        // three slots, all null
        System.out.println(names[0]);          // prints "null" — fine
        System.out.println(names[0].length()); // dies here
    }
}
```

That sample is verified to throw, which is the point: `new String[3]` creates
the array but not any strings. The slots hold `null` until you put something in
them, and `null.length()` is a `NullPointerException`.

The distinction worth holding on to:

| | `int[] a = new int[3]` | `String[] s = new String[3]` |
|---|---|---|
| What the slots hold | the number `0` | `null` |
| Usable immediately | yes | no — fill them first |

Primitive arrays arrive usable because `0` is a real number. Object arrays
arrive empty, because a reference to nothing is the only sensible default.

:::warning
An array of objects needs **two** allocations: one for the array, then one per
element. Forgetting the second is the standard way to produce a
`NullPointerException` in a free-response answer.

```java
String[] words = new String[3];   // the array exists
words[0] = "first";               // now slot 0 does too
```
:::

## Check yourself

:::quiz
{
  "question": "A method receives an int[] and does `arr = new int[]{9, 9}; arr[0] = 5;`. What does the caller's array look like afterwards?",
  "options": [
    {
      "text": "Unchanged — the method rebound its own local variable before touching anything",
      "correct": true,
      "why": "The assignment made the local variable refer to a new array, so every later use of arr in that method is about the new one. The caller's variable still refers to the original, which nothing wrote to."
    },
    {
      "text": "Its first element is 5, since arrays are passed by reference",
      "correct": false,
      "why": "The reference is passed by value. Had the method written arr[0] = 5 without the reassignment, the caller would indeed see 5 — but the reassignment redirected the local variable first, so the write landed in the new array."
    },
    {
      "text": "It becomes {9, 9} with the first element changed to 5",
      "correct": false,
      "why": "That is what the new array holds, and the caller never receives it. A method cannot change which object the caller's variable refers to; it can only be returned."
    },
    {
      "text": "A NullPointerException, because the original array was discarded",
      "correct": false,
      "why": "Nothing is discarded on the caller's side. The caller's reference is still perfectly valid — it is the method's *new* array that becomes garbage when the method returns."
    }
  ]
}
:::

## Practice

:::exercise alias-or-copy

:::exercise fill-the-slots

:::recap
- A primitive variable holds a value; an object variable holds a **reference**.
  Copying the variable copies whichever of those it holds.
- Two variables referring to one object are aliases, and a change through either
  is visible through both. Assignment never copies an object.
- Only `new` (or a method that calls it) creates an object. Count the `new`s to
  count the objects.
- Arguments are passed by value, always. A method **can** change the object you
  gave it and **cannot** change which object your variable refers to.
- `new String[3]` gives three `null`s; `new int[3]` gives three zeros. Object
  arrays need a second allocation per element.
:::
