---
title: "Calling methods, and reading a signature"
navTitle: "Calling methods"
summary: >-
  The exam hands you signatures for classes you have never seen and expects you
  to call them correctly. Reading one is a skill worth practising on its own.
objectives:
  - Read a method signature and say what it needs and what it gives back
  - Distinguish a static call from an instance call
  - Explain what happens to a return value you ignore
  - Predict which overload a call resolves to
status: complete
standard: java21
requires: [references]
---

Half of the multiple-choice section shows you a class you have never seen,
gives you its method signatures, and asks what some call returns. You are not
expected to know the class. You are expected to read the signatures.

So this chapter is about the line above the body — what each part of it tells
you, and what goes wrong when you ignore one.

## Anatomy

```java run title="Four signatures, four contracts"
public class Main {
    //     returns  name      takes
    static int      twice(int n)        { return 2 * n; }
    static void     shout(String s)     { System.out.println(s + "!"); }
    static boolean  isEven(int n)       { return n % 2 == 0; }
    static double   half(int n)         { return n / 2.0; }

    public static void main(String[] args) {
        System.out.println(twice(21));      // 42
        shout("hello");                     // hello!
        System.out.println(isEven(7));      // false
        System.out.println(half(7));        // 3.5
    }
}
```

Three parts matter every time:

- **The return type** says what you get back, and therefore what you may do
  with the call. `twice(21)` is an `int`, so it can go anywhere an `int` can.
- **The name** is what you write.
- **The parameter list** says what you must supply — how many, in what order,
  and of what types.

A **parameter** is the variable in the declaration; an **argument** is the value
you pass. The exam uses both words precisely, so it is worth keeping them apart.

## `void` means there is nothing to use

A method declared `void` returns nothing, and the call is a statement rather
than a value. Trying to use it as a value does not compile.

```java run expect-error title="void is not a value"
public class Main {
    static void shout(String s) { System.out.println(s + "!"); }

    public static void main(String[] args) {
        String result = shout("hello");   // shout gives back nothing
        System.out.println(result);
    }
}
```

That sample is verified not to compile. `void` is the signature telling you the
method's whole purpose is its **effect** — printing, changing a field, adding to
a list — and that there is no answer to collect.

## A return value you ignore is thrown away

The opposite mistake compiles cleanly, which makes it far more dangerous.

```java run title="The call happened. The answer went nowhere."
public class Main {
    static int twice(int n) { return 2 * n; }

    public static void main(String[] args) {
        int x = 5;

        twice(x);                       // runs, computes 10, discards it
        System.out.println(x);          // 5 — x never changed

        x = twice(x);                   // the answer is kept this time
        System.out.println(x);          // 10
    }
}
```

`twice(x)` does not change `x`. It cannot — `x` is an `int`, so the method got a
copy of the value, exactly as chapter 1.2 described. The only way the answer
reaches you is through the return, and the only way the return survives is if
you assign it or use it.

:::pitfall
**A method that returns something and changes nothing is useless if you drop the
result.** Nothing warns you: it compiles, it runs, and the value evaporates.

This is the single most common error with `String` methods, which is where
chapter 1.4 takes it up. `s.toUpperCase();` on its own leaves `s` exactly as it
was.
:::

## Static against instance

Two ways to call, and the signature tells you which.

- A method declared `static` belongs to the **class**. Call it on the class
  name: `Math.abs(-4)`.
- A method without `static` belongs to an **object**. Call it on a reference to
  one: `word.length()`.

```java run title="Both kinds, side by side"
public class Main {
    public static void main(String[] args) {
        System.out.println(Math.max(3, 9));     // static: on the class
        System.out.println(Math.abs(-4));       // static: on the class

        String word = "calculus";
        System.out.println(word.length());      // instance: on the object
        System.out.println(word.indexOf("cul"));// instance: on the object
    }
}
```

The reason is the one from chapter 1.2. An instance method usually needs the
object's data to do its job — `length()` has to know *which* string — so it must
be called on one. A static method does not, so there is nothing for it to be
called on.

:::warning
Calling an instance method on a `null` reference throws a
`NullPointerException`, because there is no object for the method to run on.

Calling a static method never can: it needs no object, so there is nothing to
be missing.
:::

## Overloading

Two methods may share a name if their parameter lists differ. Java picks by the
arguments you pass.

```java run title="Same name, three methods"
public class Main {
    static String describe(int n)          { return "one int: " + n; }
    static String describe(int a, int b)   { return "two ints: " + a + ", " + b; }
    static String describe(double d)       { return "a double: " + d; }

    public static void main(String[] args) {
        System.out.println(describe(5));         // one int
        System.out.println(describe(5, 6));      // two ints
        System.out.println(describe(5.0));       // a double
        System.out.println(describe('A'));       // one int: 65
    }
}
```

The last line is the interesting one. There is no `describe(char)`, so Java
widens `'A'` to the `int` `65` and takes the `int` version. Widening is allowed
when choosing an overload; narrowing is not, which is why `describe(5)` takes
the `int` version rather than widening to `double` — **the most specific match
that works wins**.

:::note
The **return type is not part of the signature** for this purpose. Two methods
that differ only by what they return will not compile, because a call site
cannot always say which was meant — `describe(5);` on its own line gives Java
nothing to choose with.
:::

## Check yourself

:::quiz
{
  "question": "A class offers `public int add(int n)` which adds n to a running total and returns the new total. You write `counter.add(5);` on a line by itself. What happens?",
  "options": [
    {
      "text": "The total does change — the method's effect happens regardless of whether you keep the return value",
      "correct": true,
      "why": "The method runs completely. It updates the object's field, and separately hands back a value that you discarded. Discarding a return never cancels what the method did to the object."
    },
    {
      "text": "Nothing, because the return value was not assigned to anything",
      "correct": false,
      "why": "Ignoring a return discards the answer, not the call. The method body still executed and still changed the object — which is exactly why this is worth distinguishing from the twice(x) case above, where there was no object to change."
    },
    {
      "text": "It fails to compile, since a non-void call must be assigned",
      "correct": false,
      "why": "Java allows a non-void call as a statement, and that permissiveness is the trap: nothing warns you when the value you wanted is thrown away."
    },
    {
      "text": "It throws an exception at runtime",
      "correct": false,
      "why": "There is nothing exceptional about discarding a value. The only exception risk here would be counter being null, which is a different problem."
    }
  ]
}
:::

## Practice

:::exercise read-the-signature

:::exercise return-or-effect

:::recap
- A signature says what a method **needs** (parameters) and what it **gives
  back** (return type). The exam supplies signatures and expects you to read
  them.
- `void` means there is no value to collect, and using the call as one does not
  compile.
- A non-void call used as a statement compiles, runs, and discards the answer.
  Nothing warns you.
- `static` methods are called on the class, instance methods on an object — and
  only the second kind can throw a `NullPointerException` for a missing object.
- Overloads are chosen by the argument list; the most specific match that works
  wins, and widening is allowed where narrowing is not.
- The return type is not part of what distinguishes overloads.
:::
