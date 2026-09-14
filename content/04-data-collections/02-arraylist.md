---
title: "ArrayList"
navTitle: "ArrayList"
summary: >-
  A list that grows, at the cost of boxing — and one removal bug that appears
  on the exam almost every year.
objectives:
  - Use the ArrayList methods the exam tests
  - Explain why an ArrayList holds objects rather than primitives
  - Remove elements while iterating without skipping any
  - Choose between an array and an ArrayList
status: complete
standard: java21
---

An array's length is fixed at creation. `ArrayList` fixes that, and the price is
that it stores **objects**, not primitives — which is where its two surprises
come from.

## The methods that matter

```java run title="The exam's working set"
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<String> names = new ArrayList<>();

        names.add("Ana");                 // append
        names.add("Ben");
        names.add("Cy");
        names.add(1, "Bo");               // insert at index, shifting the rest

        System.out.println(names);
        System.out.println("size    " + names.size());
        System.out.println("get(0)  " + names.get(0));

        names.set(0, "Anna");             // replace
        String gone = names.remove(2);    // remove by index, returns what left

        System.out.println("removed " + gone);
        System.out.println(names);
        System.out.println("contains Cy: " + names.contains("Cy"));
        System.out.println("indexOf Cy : " + names.indexOf("Cy"));
    }
}
```

Note `size()` rather than `length`, and `get(i)` rather than `[i]`. Different
spellings for the same ideas, and the exam expects both fluently.

## Why `ArrayList<int>` does not compile

```java run expect-error title="Primitives are not allowed as type arguments"
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<int> numbers = new ArrayList<>();   // will not compile
        numbers.add(5);
    }
}
```

That sample is asserted **not** to compile, and the verifier checks that it
really fails — an "expect-error" example that quietly started working would be
teaching the opposite of what it claims.

The fix is the wrapper type:

```java run title="Integer, and the boxing that comes with it"
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<Integer> numbers = new ArrayList<>();

        numbers.add(5);                 // autoboxed: int 5 becomes Integer 5
        numbers.add(10);

        int first = numbers.get(0);     // auto-unboxed back to int
        System.out.println("sum " + (first + numbers.get(1)));

        // Each element is a separate object with a header, which is why an
        // ArrayList<Integer> uses several times the memory of an int[].
        System.out.println(numbers);
    }
}
```

Autoboxing makes this mostly invisible, which is convenient and occasionally
harmful — see the removal trap below.

## Removing while iterating

This is the single most-tested bug in the unit.

```java run title="The skip, demonstrated"
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<Integer> a = new ArrayList<>();
        for (int v : new int[]{1, 2, 2, 3}) a.add(v);

        // Wrong: removing shifts everything left, and i still advances.
        for (int i = 0; i < a.size(); i++) {
            if (a.get(i) == 2) a.remove(i);
        }
        System.out.println("forwards, advancing anyway: " + a);

        ArrayList<Integer> b = new ArrayList<>();
        for (int v : new int[]{1, 2, 2, 3}) b.add(v);

        // Right: walk backwards, so a shift cannot move anything unvisited.
        for (int i = b.size() - 1; i >= 0; i--) {
            if (b.get(i) == 2) b.remove(i);
        }
        System.out.println("backwards:                 " + b);
    }
}
```

The first loop leaves a `2` behind. Removing index 1 shifts the second `2` down
into index 1, then `i++` moves to index 2 — stepping straight over it.

Two fixes, both acceptable on the exam:

- **Walk backwards.** Shifts only affect indices you have already passed.
- **Do not increment when you remove.** `if (...) a.remove(i); else i++;`

:::pitfall
`remove` is **overloaded**, and on an `ArrayList<Integer>` the two versions do
completely different things:

- `remove(int index)` — removes by **position**
- `remove(Object o)` — removes by **value**

`list.remove(2)` removes the element at index 2. To remove the *value* 2 you
must write `list.remove(Integer.valueOf(2))`.

Autoboxing does not save you here: an `int` literal matches the index version
exactly, so Java takes it without a word.
:::

```java run title="The overload, demonstrated"
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<Integer> a = new ArrayList<>();
        for (int v : new int[]{10, 20, 30}) a.add(v);

        ArrayList<Integer> b = new ArrayList<>(a);

        a.remove(2);                        // by index: drops 30
        b.remove(Integer.valueOf(20));      // by value: drops 20

        System.out.println("remove(2)                 -> " + a);
        System.out.println("remove(Integer.valueOf(20)) -> " + b);
    }
}
```

## Which to use

| Use an array when | Use an `ArrayList` when |
|---|---|
| the size is known and fixed | the size changes |
| the elements are primitives and speed matters | you need `add`, `remove`, `contains` |
| you want 2-D structure | you want to grow from empty |

For the exam, the honest answer is usually "whichever the question hands you".
Both appear, and FRQ 3 is specifically about one or the other.

:::quiz
{
  "question": "list holds [5, 5, 7]. What does list.remove(1) leave behind, where list is an ArrayList<Integer>?",
  "options": [
    {
      "text": "[5, 7] — it removes the element at index 1",
      "correct": true,
      "why": "An int literal selects remove(int index), so index 1 goes. The second 5 is removed, not the value 1, and no 1 is present anyway."
    },
    {
      "text": "[5, 5, 7] — there is no element equal to 1, so nothing changes",
      "correct": false,
      "why": "That would be remove(Object), which needs Integer.valueOf(1). A bare int always picks the index overload, and index 1 exists."
    },
    {
      "text": "[5, 5] — it removes the last element",
      "correct": false,
      "why": "Index 1 is the middle element, not the last. The last is index 2, since indices start at 0."
    },
    {
      "text": "It throws IndexOutOfBoundsException",
      "correct": false,
      "why": "Index 1 is valid in a three-element list, where legal indices are 0, 1 and 2."
    }
  ]
}
:::

:::recap
- `size()`, `get(i)`, `add`, `set`, `remove`, `contains`, `indexOf` — and note
  `size()` rather than `length`.
- `ArrayList` holds objects, so primitives need wrappers: `ArrayList<Integer>`,
  never `ArrayList<int>`.
- Removing forward while incrementing skips the next element. Walk backwards,
  or do not increment on a removal.
- `remove(int)` is by index and `remove(Object)` is by value. On an
  `ArrayList<Integer>` a bare int always means the index.
- Arrays for fixed size and primitives; `ArrayList` for growth and its methods.
:::
