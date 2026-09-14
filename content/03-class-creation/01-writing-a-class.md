---
title: "Writing a class"
navTitle: "Writing a class"
summary: >-
  FRQ 2 asks you to write a class from a specification. It is the most
  formulaic question on the exam, and that is good news.
objectives:
  - Write fields, constructors, accessors and mutators from a specification
  - Use this to disambiguate a parameter from a field
  - Explain what private buys and why the exam insists on it
  - Write toString and know when it is called
status: complete
standard: java21
---

Unit 3 is 10–18% of the exam, but its importance is larger than that: one of the
four free-response questions asks you to write a class from a written
specification, and the structure of the answer is always the same.

Learn the shape and FRQ 2 becomes filling in a template.

## The shape

```java run title="Every part, in the order they are written"
public class Main {
    public static void main(String[] args) {
        Book b = new Book("Dune", 412);
        System.out.println(b.getTitle() + " has " + b.getPages() + " pages");

        b.setPages(500);
        System.out.println("after setPages: " + b);     // toString is called here
    }
}

class Book {
    // 1. Fields — always private.
    private String title;
    private int pages;

    // 2. Constructor — same name as the class, no return type.
    public Book(String title, int pages) {
        this.title = title;      // this.title is the field, title is the parameter
        this.pages = pages;
    }

    // 3. Accessors ("getters") — return a field.
    public String getTitle() { return title; }
    public int getPages()    { return pages; }

    // 4. Mutators ("setters") — change a field.
    public void setPages(int pages) {
        this.pages = pages;
    }

    // 5. toString — what printing this object shows.
    public String toString() {
        return title + " (" + pages + " pp)";
    }
}
```

Five parts, in that order. An FRQ specification names the fields and the methods;
your job is to produce exactly this structure with the right names.

:::note
A constructor has **no return type** — not even `void`. Writing
`public void Book(...)` makes it an ordinary method that happens to share the
class's name, the real constructor goes missing, and `new Book("Dune", 412)`
stops compiling. It is a small typo with a confusing error message.
:::

## What `this` is for

```java run title="With and without this"
public class Main {
    public static void main(String[] args) {
        System.out.println(new Counter(5));
        System.out.println(new Broken(5));
    }
}

class Counter {
    private int count;
    public Counter(int count) {
        this.count = count;        // field = parameter
    }
    public String toString() { return "Counter holding " + count; }
}

class Broken {
    private int count;
    public Broken(int count) {
        count = count;             // assigns the parameter to itself
    }
    public String toString() { return "Broken holding " + count; }
}
```

`Broken` compiles cleanly and leaves the field at its default `0`. When a
parameter has the same name as a field, the parameter **shadows** it, so the
bare name refers to the parameter everywhere in that method. `this.count` is how
you reach past the shadow to the field.

You can avoid the whole issue by naming the parameter differently — but the exam
uses matching names constantly, so `this` has to be automatic.

## Why `private`

```java run expect-error title="private really does block access"
public class Main {
    public static void main(String[] args) {
        Account a = new Account(100);
        a.balance = -5000;             // will not compile
    }
}

class Account {
    private int balance;
    public Account(int balance) { this.balance = balance; }
    public int getBalance() { return balance; }
}
```

That sample is asserted not to compile, and the verifier confirms it. The point
is not secrecy — it is that **the class controls its own invariants**. With a
public field, any code anywhere can put the object into a state its methods
never intended.

```java run title="A mutator that enforces a rule"
public class Main {
    public static void main(String[] args) {
        Account a = new Account(100);
        System.out.println("start:            " + a.getBalance());
        System.out.println("withdraw 30:      " + a.withdraw(30));
        System.out.println("balance now:      " + a.getBalance());
        System.out.println("withdraw 1000:    " + a.withdraw(1000));
        System.out.println("balance still:    " + a.getBalance());
    }
}

class Account {
    private int balance;
    public Account(int balance) { this.balance = balance; }
    public int getBalance() { return balance; }

    public boolean withdraw(int amount) {
        if (amount > balance) return false;    // the rule lives here
        balance -= amount;
        return true;
    }
}
```

The balance can never go negative, because the only route to it checks first.
That guarantee is what `private` buys, and it is the answer to "why is
encapsulation good" if a question asks.

## `toString`

```java run title="When toString is called for you"
public class Main {
    public static void main(String[] args) {
        Point p = new Point(3, 4);

        System.out.println(p);                    // called implicitly
        System.out.println("point: " + p);        // and in concatenation
        System.out.println(p.toString());         // and explicitly

        Point noToString = null;
        System.out.println("null prints as: " + noToString);   // no crash
    }
}

class Point {
    private int x, y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    public String toString() { return "(" + x + ", " + y + ")"; }
}
```

`println` and string concatenation both call `toString` automatically. Without
one, you get the default — the class name, an `@`, and a hash — which is never
what a question wants.

Concatenating a `null` reference prints `"null"` rather than throwing, which is
worth knowing because it is the one place a null does not blow up.

:::quiz
{
  "question": "A constructor is written as: public Item(String name) { name = name; }\n\nWhat happens?",
  "options": [
    {
      "text": "It compiles, and the field is left at null — the parameter is assigned to itself",
      "correct": true,
      "why": "The parameter shadows the field, so both sides of the assignment are the parameter. The field keeps its default, null for a String. this.name = name is the fix."
    },
    {
      "text": "It does not compile, because a variable cannot be assigned to itself",
      "correct": false,
      "why": "Java permits it. A compiler may warn about a self-assignment, but it is legal code and the exam's compiler-free setting will not mention it at all."
    },
    {
      "text": "It works correctly, since Java matches the parameter to the field by name",
      "correct": false,
      "why": "Java does the opposite — matching names means the parameter wins inside the method. Reaching the field requires this."
    },
    {
      "text": "It throws NullPointerException when the constructor runs",
      "correct": false,
      "why": "Nothing is dereferenced, so nothing throws. The object is constructed with a null field, and the exception comes later, if some method calls a method on it."
    }
  ]
}
:::

:::recap
- Five parts in order: private fields, constructor, accessors, mutators,
  `toString`.
- A constructor has no return type. Adding `void` silently turns it into an
  ordinary method.
- A parameter with a field's name shadows it. `this.x = x` is how you assign
  past the shadow.
- `private` exists so the class controls its own invariants — a mutator can
  enforce rules a public field cannot.
- `toString` is called by `println` and by string concatenation. A null
  reference concatenates as `"null"` without throwing.
:::
