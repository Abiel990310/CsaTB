---
title: "Searching and sorting"
navTitle: "Search and sort"
summary: >-
  Four algorithms the exam expects you to trace by hand, and the one
  precondition that makes binary search work.
objectives:
  - Implement linear and binary search
  - State the precondition binary search requires
  - Trace selection and insertion sort after a given number of passes
  - Compare the algorithms by number of comparisons
status: complete
standard: java21
---

These four algorithms appear on the exam as **tracing** questions: given an
array and an algorithm, what does it look like after three passes? That is a
different skill from writing them, and it is the one worth practising.

## Linear search

Check each element until you find the target or run out.

```java run title="Linear search, with a count"
public class Main {
    static int linearSearch(int[] a, int target) {
        for (int i = 0; i < a.length; i++) {
            if (a[i] == target) return i;
        }
        return -1;                    // the convention for "not found"
    }

    public static void main(String[] args) {
        int[] data = {9, 3, 7, 1, 8};
        System.out.println("find 7  -> index " + linearSearch(data, 7));
        System.out.println("find 42 -> index " + linearSearch(data, 42));
    }
}
```

It works on **any** array, sorted or not, and that generality is its whole
advantage. On an array of $n$ elements it needs up to $n$ comparisons.

## Binary search

Repeatedly halve the search space by comparing against the middle element.

```java run title="Binary search, showing the shrinking window"
public class Main {
    static int binarySearch(int[] a, int target) {
        int lo = 0, hi = a.length - 1;

        while (lo <= hi) {
            int mid = (lo + hi) / 2;
            System.out.println("  looking in [" + lo + ", " + hi + "], mid=" + mid
                               + " value=" + a[mid]);
            if (a[mid] == target) return mid;
            if (a[mid] < target) lo = mid + 1;   // discard the left half
            else hi = mid - 1;                   // discard the right half
        }
        return -1;
    }

    public static void main(String[] args) {
        int[] sorted = {1, 3, 5, 7, 9, 11, 13};
        System.out.println("find 11:");
        System.out.println("-> index " + binarySearch(sorted, 11));
        System.out.println("find 4:");
        System.out.println("-> index " + binarySearch(sorted, 4));
    }
}
```

Seven elements, at most three comparisons. Doubling the array adds one
comparison rather than doubling the work — that is what logarithmic means, and
it is why binary search matters.

:::warning
**Binary search requires a sorted array.** On unsorted data it does not search
slowly or search wrongly — it discards half the array based on a comparison that
means nothing, and returns −1 for elements that are present.

It will not throw. It will not warn. It will simply be wrong, which makes this
the precondition to state in any FRQ that uses it.
:::

```java run title="What it does to unsorted data"
public class Main {
    static int binarySearch(int[] a, int target) {
        int lo = 0, hi = a.length - 1;
        while (lo <= hi) {
            int mid = (lo + hi) / 2;
            if (a[mid] == target) return mid;
            if (a[mid] < target) lo = mid + 1;
            else hi = mid - 1;
        }
        return -1;
    }

    public static void main(String[] args) {
        int[] unsorted = {9, 3, 7, 1, 8};
        System.out.println("7 is at index 2, but binary search says: "
                           + binarySearch(unsorted, 7));
    }
}
```

## Selection sort

Repeatedly find the smallest remaining element and swap it into place.

```java run title="Selection sort, one pass at a time"
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] a = {29, 10, 14, 37, 13};
        System.out.println("start:  " + Arrays.toString(a));

        for (int i = 0; i < a.length - 1; i++) {
            int minIndex = i;
            for (int j = i + 1; j < a.length; j++) {
                if (a[j] < a[minIndex]) minIndex = j;
            }
            int temp = a[i];
            a[i] = a[minIndex];
            a[minIndex] = temp;

            System.out.println("pass " + (i + 1) + ": " + Arrays.toString(a));
        }
    }
}
```

The pattern the exam tests: **after pass $k$, the first $k$ elements are in
final sorted position** and nothing else is guaranteed. A question asking for
the array "after two passes" wants exactly that state.

## Insertion sort

Take each element and slide it back into the sorted region to its left.

```java run title="Insertion sort, one pass at a time"
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] a = {29, 10, 14, 37, 13};
        System.out.println("start:  " + Arrays.toString(a));

        for (int i = 1; i < a.length; i++) {
            int value = a[i];
            int j = i - 1;
            while (j >= 0 && a[j] > value) {
                a[j + 1] = a[j];       // shift right
                j--;
            }
            a[j + 1] = value;

            System.out.println("pass " + i + ": " + Arrays.toString(a));
        }
    }
}
```

Its pattern is different, and mixing the two up is the standard tracing error:
**after pass $k$, the first $k+1$ elements are sorted among themselves** — but
they are not necessarily in final position, because a smaller element may still
arrive from the right.

| | after pass $k$ |
|---|---|
| **selection** | first $k$ elements are in **final** position |
| **insertion** | first $k+1$ are sorted **relative to each other** |

## How many comparisons

| Algorithm | Typical | Best case |
|---|---|---|
| linear search | $n$ | 1 |
| binary search | $\log_2 n$ | 1 |
| selection sort | $n^2$ | $n^2$ — always |
| insertion sort | $n^2$ | $n$ — already sorted |

Selection sort does the same work regardless of input: it scans the whole
remaining array every pass, even on sorted data. Insertion sort's inner `while`
exits immediately when the element is already in place, so a sorted array costs
one comparison per element.

That asymmetry is the most commonly asked comparison question about these two.

:::quiz
{
  "question": "Selection sort runs on {29, 10, 14, 37, 13}. What is the array after two passes?",
  "options": [
    {
      "text": "{10, 13, 14, 37, 29}",
      "correct": true,
      "why": "Pass 1 swaps the smallest (10) into index 0, giving {10, 29, 14, 37, 13}. Pass 2 finds the smallest of the rest (13) and swaps it with index 1, giving {10, 13, 14, 37, 29}."
    },
    {
      "text": "{10, 14, 29, 37, 13}",
      "correct": false,
      "why": "That is insertion sort's state after two passes — it sorts the leading elements among themselves by shifting. Selection sort swaps, which leaves displaced elements elsewhere."
    },
    {
      "text": "{10, 13, 14, 29, 37}",
      "correct": false,
      "why": "That is the fully sorted array, which takes four passes. After two, only the first two positions are final."
    },
    {
      "text": "{13, 10, 14, 37, 29}",
      "correct": false,
      "why": "Selection sort places the smallest first, so index 0 must hold 10 after pass 1. This has them the wrong way round."
    }
  ]
}
:::

:::recap
- Linear search works on any array in up to $n$ comparisons; binary search needs
  a **sorted** one and takes about $\log_2 n$.
- Binary search on unsorted data fails silently — no exception, just wrong
  answers.
- Selection sort: after pass $k$ the first $k$ are in **final** position.
- Insertion sort: after pass $k$ the first $k+1$ are sorted **among
  themselves**.
- Selection sort ignores the input's order; insertion sort is linear on an
  already-sorted array. That is the usual comparison question.
:::
