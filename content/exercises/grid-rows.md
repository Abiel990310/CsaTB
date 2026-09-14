---
id: grid-rows
title: "Row and column sums"
difficulty: core
chapter: two-d-arrays
topics: [arrays, loops]
check: unit
standard: java21
---

Two methods over a rectangular `int[][]`.

- `rowSums(g)` — an array holding the total of each row.
- `columnSums(g)` — an array holding the total of each column.

The starter has `rowSums` working and `columnSums` written as a stub, because
the second is where the indexing actually has to be thought about.

A grid with no rows gives an empty array from both.

## Starter
```java
static int[] rowSums(int[][] g) {
    int[] out = new int[g.length];
    for (int r = 0; r < g.length; r++) {
        for (int c = 0; c < g[r].length; c++) {
            out[r] += g[r][c];
        }
    }
    return out;
}

static int[] columnSums(int[][] g) {
    return new int[0];      // your turn
}
```

## Tests
```java
int[][] grid = {{1, 2, 3}, {4, 5, 6}};
checkEq(java.util.Arrays.toString(rowSums(grid)), "[6, 15]");
checkEq(java.util.Arrays.toString(columnSums(grid)), "[5, 7, 9]");

int[][] tall = {{1, 2}, {3, 4}, {5, 6}};
checkEq(java.util.Arrays.toString(rowSums(tall)), "[3, 7, 11]");
checkEq(java.util.Arrays.toString(columnSums(tall)), "[9, 12]");

int[][] single = {{42}};
checkEq(java.util.Arrays.toString(columnSums(single)), "[42]");

int[][] empty = {};
checkEq(columnSums(empty).length, 0);
```

## Hints
- The result of `columnSums` has one entry per column, so its length is `g[0].length` — not `g.length`.
- Loop columns on the outside and rows on the inside, or loop rows outside and accumulate into `out[c]`. Both work.
- Indexing is always `g[row][column]`, whichever loop is outer. Writing `g[c][r]` throws on any non-square grid.
- Guard the empty case: `g.length == 0` means there is no `g[0]` to ask for a width.

## Solution
```java
static int[] rowSums(int[][] g) {
    int[] out = new int[g.length];
    for (int r = 0; r < g.length; r++) {
        for (int c = 0; c < g[r].length; c++) {
            out[r] += g[r][c];
        }
    }
    return out;
}

static int[] columnSums(int[][] g) {
    if (g.length == 0) return new int[0];   // no rows means no columns to size from

    int[] out = new int[g[0].length];
    for (int r = 0; r < g.length; r++) {
        for (int c = 0; c < g[r].length; c++) {
            out[c] += g[r][c];              // same traversal, different destination
        }
    }
    return out;
}
```

## Notes
The solution's two methods traverse **identically** — row-major, the same nested
loop — and differ only in where they accumulate: `out[r]` against `out[c]`. That
is worth noticing, because the instinct is to swap the loops for columns and
that is not necessary.

Swapping the loops works too, and then the bounds come from different places:
the outer loop runs to `g[0].length` and the inner to `g.length`. Either is
correct; accumulating into `out[c]` is less error-prone because the traversal
stays the one you already know.

**The empty guard is not defensive padding.** `new int[g[0].length]` on a grid
with no rows throws `ArrayIndexOutOfBoundsException` before the loop starts, and
the test for it is in the suite because a real FRQ will specify the empty case
and mark it.

The width comes from `g[0]` because a rectangular grid's rows all match. On a
jagged array the question "how many columns" has no single answer, which is why
the specification says rectangular.
