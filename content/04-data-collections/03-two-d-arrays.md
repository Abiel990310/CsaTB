---
title: "2-D arrays"
navTitle: "2-D arrays"
summary: >-
  An array of arrays, not a rectangle — and the FRQ that appears on every exam
  depends on knowing the difference.
objectives:
  - Declare and traverse a 2-D array in row-major order
  - Use the correct length for rows and columns
  - Traverse by column when a problem requires it
  - Explain why Java has no true 2-D array
status: complete
standard: java21
---

One of the four free-response questions is about 2-D arrays, every year. It is
the most predictable mark on the paper, and it turns on one structural fact.

## Java has no 2-D array

`int[][] grid` is an **array whose elements are arrays**. That is not a pedantic
distinction — it is why `grid.length` and `grid[0].length` mean different things,
and why rows can have different lengths.

```java run title="What a 2-D array actually is"
public class Main {
    public static void main(String[] args) {
        int[][] grid = {
            {1, 2, 3},
            {4, 5, 6}
        };

        System.out.println("rows:            " + grid.length);
        System.out.println("cols in row 0:   " + grid[0].length);
        System.out.println("grid[1][2]:      " + grid[1][2]);

        // Each row is a separate object.
        System.out.println("row 0 is an int[]: " + (grid[0] instanceof int[]));

        // And rows need not match in length — a "jagged" array.
        int[][] jagged = new int[3][];
        jagged[0] = new int[]{1};
        jagged[1] = new int[]{1, 2};
        jagged[2] = new int[]{1, 2, 3};

        for (int[] row : jagged) System.out.print(row.length + " ");
        System.out.println();
    }
}
```

The exam uses rectangular arrays almost exclusively, but the underlying model is
what makes `grid[0].length` the right way to ask for a column count — there is
no single "width" stored anywhere.

:::pitfall
`grid.length` is the number of **rows**. `grid[0].length` is the number of
**columns**.

Getting these backwards produces `ArrayIndexOutOfBoundsException` on any
non-square grid, and — worse — works fine on a square one. Test code on a
rectangle, not a square, or the bug hides.
:::

## Row-major traversal

The standard nested loop, and the one an FRQ expects unless it says otherwise:

```java run title="Row by row, left to right"
public class Main {
    public static void main(String[] args) {
        int[][] grid = {
            {1, 2, 3},
            {4, 5, 6}
        };

        int sum = 0;
        StringBuilder order = new StringBuilder();

        for (int r = 0; r < grid.length; r++) {
            for (int c = 0; c < grid[r].length; c++) {
                order.append(grid[r][c]).append(" ");
                sum += grid[r][c];
            }
        }

        System.out.println("visit order: " + order);
        System.out.println("sum: " + sum);
    }
}
```

Note `grid[r].length` rather than `grid[0].length` in the inner loop. On a
rectangular array they are the same; on a jagged one only the first is correct,
and writing it that way costs nothing.

The enhanced form works too, when positions are not needed:

```java run title="Enhanced for, nested"
public class Main {
    public static void main(String[] args) {
        int[][] grid = {{1, 2, 3}, {4, 5, 6}};

        int max = grid[0][0];
        for (int[] row : grid) {          // each element is a row
            for (int value : row) {       // each element is an int
                if (value > max) max = value;
            }
        }
        System.out.println("max: " + max);
    }
}
```

The outer variable is `int[]`, not `int`. Declaring it `int` is a compile error,
and it is a common one on a written FRQ where no compiler is watching.

## Column-major traversal

When a question asks about columns, the loops swap and the **bounds come from
different places**:

```java run title="Column by column, top to bottom"
public class Main {
    public static void main(String[] args) {
        int[][] grid = {
            {1, 2, 3},
            {4, 5, 6}
        };

        for (int c = 0; c < grid[0].length; c++) {   // columns: width
            int colSum = 0;
            for (int r = 0; r < grid.length; r++) {  // rows: height
                colSum += grid[r][c];
            }
            System.out.println("column " + c + " sums to " + colSum);
        }
    }
}
```

The indexing is still `grid[row][column]` — **always row first**. Only the loop
order changed. Writing `grid[c][r]` is the classic error, and on a
non-square grid it throws immediately.

## A worked FRQ-style task

```java run title="Does any row contain only even numbers?"
public class Main {
    static boolean hasAllEvenRow(int[][] grid) {
        for (int r = 0; r < grid.length; r++) {
            boolean allEven = true;
            for (int c = 0; c < grid[r].length; c++) {
                if (grid[r][c] % 2 != 0) {
                    allEven = false;
                    break;            // this row is settled
                }
            }
            if (allEven) return true; // found one, done
        }
        return false;
    }

    public static void main(String[] args) {
        System.out.println(hasAllEvenRow(new int[][]{{1,2},{4,6}}));  // true
        System.out.println(hasAllEvenRow(new int[][]{{1,2},{4,7}}));  // false
        System.out.println(hasAllEvenRow(new int[][]{{}}));           // true — vacuous
    }
}
```

Two structural points that earn rubric marks:

- **The flag is reset inside the outer loop.** Declaring `allEven` outside would
  let one bad row poison every later one.
- **The empty row returns true**, because "all of nothing is even" is vacuously
  satisfied. That is what the code does, and it is worth noticing rather than
  discovering in testing.

:::quiz
{
  "question": "For int[][] g = {{1,2,3},{4,5,6}}, what is g[0].length?",
  "options": [
    {
      "text": "3 — the number of columns, since g[0] is the first row",
      "correct": true,
      "why": "g[0] is the int[] {1,2,3}, so its length is 3. Row count is g.length, which is 2."
    },
    {
      "text": "2 — the number of rows",
      "correct": false,
      "why": "That is g.length. g[0].length asks a specific row how long it is, which gives the width."
    },
    {
      "text": "6 — the total number of elements",
      "correct": false,
      "why": "No single field holds that. You would compute it as g.length * g[0].length, and on a jagged array by summing each row's length."
    },
    {
      "text": "It does not compile — length applies to arrays, not to g[0]",
      "correct": false,
      "why": "g[0] is an array, which is exactly the point: a 2-D array is an array of arrays, so each element has its own length field."
    }
  ]
}
:::

:::recap
- `int[][]` is an array of arrays, so rows are separate objects and may differ
  in length.
- `grid.length` is rows; `grid[r].length` is that row's columns. Backwards works
  on squares and throws on rectangles.
- Indexing is always `grid[row][column]`, whichever loop is outer.
- Row-major nests columns inside rows; column-major swaps the loops but not the
  indexing.
- In the nested enhanced for, the outer variable is `int[]`.
:::
