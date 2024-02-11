# Solving-Sudoku-using-SQL

## What is Sudoku?
You can think of a Sudoku as a square divided into 9 equal squares (blocks) and the 9 squares each further divided into 9 much smaller squares (cells). That makes 9 blocks and 81 cells.

The blocks make a 3 x 3 grid (i.e. 3 rows and 3 columns). The cells make a 9 x 9 grid (i.e. 9 rows and 9 columns).

A Sudoku puzzle comes with numbers already present in some of the cells. The objective of the puzzle is to fill in the remaining cells with the right numbers. The cells must be filled according to the following simple rules:

    All the 9 cells in each row must have the numbers 1 to 9 with no repetition.
    All the 9 cells in each column must have the numbers 1 to 9 with no repetition.
    All the 9 cells within each block must have the numbers 1 to 9 with no repetition.

(‘No repetition’ is purely for emphasis, because if you use all of 1 to 9, and you have only 9 places to fill, there can be no repetition anyway.)

Sudoku comes in varying levels of difficulty and degrees of challenge, depending on which, and how many, cells have been filled in already. Whatever the case, you do not need any kind of serious mathematics or clever mental gymnastics; all you need is your own personal logical reasoning and a determination not to be beaten. That is why they can be so much fun. 

## How to Solve Sudoku Puzzle?
## Explanation:

Here's one way to solve a Sudoku puzzles:

Let x be a set (or a pile, if you prefer) of puzzles, either partially completed (with blank spaces left) or completely filled in.

    Start with x being the one puzzle as given.
    Take a puzzle from x that is not yet completely filled
    Make 9 copies of that puzzle, by filling in the first blank space with all 9 possible values, '1' through '9'.
    For each of those 9 copies, check to see if it is a valid entry (e.g., it does not duplicate a digit already in the same row)
    Put the new puzzles that did not have any errors back into x
    Repeat steps 2-5, until you have looked at all the puzzles partially filled-in puzzles
    At this point, any puzzle in x that is completely filled in will be a valid solution.

(That's not a very fast way to solve Sudokus, and not how most people actually do it, but it is one possible way.  Computers can use this approach, because computers are very fast, and they don't get bored.)

In the query x, s is a partially (or completely) filled-in puzzle, and ind is the position of the first blank (that is, unfilled) cell in that puzzle.  If ind=0, that means the puzzle is completely filled-in.

In a recursive WITH clause, the first branch of the UNION is a starting point.  This will be a normal, un-recursive query. In the query you posted, lines 2-3 are the first branch.  This corresponds to step 1 in the algorithm.

The second branch of the UNION (lines 5-24 in this example) is the complicated part: it's a query that references rows previously generated, and it's the part that gets done over and over (corresponding to steps 2-5 in the algorithm).

Lines 7 and 12 of the query correspond to step 2 of the algorithm.

Lines 8-11 of the query correspond to step 3 of the algorithm.  It cross-joins a row from x to the 9 rows in the sub-query (lines 8-10) to make 9 copies of the existing partially solved puzzle.

Lines 13-23 of the query correspond to steps 4 and 5 of the algorithm.  It is valid to insert the new character z into the puzzle if z is not already in

    the 9 cells of the same row
    the 9 cells of the same column, or
    the 9 cells of the same 3x3 cage.

Notice that all 3 of these tests involve checking 9 cells.  The sub- query on lines 14-16 generates the numbers 1-9, and the WHERE clause conditions (lines 18-23) check for the duplicates.

The super-query (lines 26-28) corresponds to step 7 of the algorithm: it displays only the completely filled-in version(s) of the puzzle.

## Usage

SQL shell:
Code that gets the Sudoku resolved

```

> with x( s, ind ) as
( select sud, instr( sud, ' ' )
  from ( select '&' sud from dual )
  union all
  select substr( s, 1, ind - 1 ) || z || substr( s, ind + 1 )
       , instr( s, ' ', ind + 1 )
  from x
     , ( select to_char( rownum ) z
         from dual
         connect by rownum <= 9
       ) z
  where ind > 0
  and not exists ( select null
                   from ( select rownum lp
                          from dual
                          connect by rownum <= 9
                        )
                   where z = substr( s, trunc( ( ind - 1 ) / 9 ) * 9 + lp, 1 )
                   or    z = substr( s, mod( ind - 1, 9 ) - 8 + lp * 9, 1 )
                   or    z = substr( s, mod( trunc( ( ind - 1 ) / 3 ), 3 ) * 3
                                      + trunc( ( ind - 1 ) / 27 ) * 27 + lp
                                      + trunc( ( lp - 1 ) / 3 ) * 6
                                   , 1 )
                 )
)
select line_wrap(s,9)
from x
where ind = 0
/

```

## Output
```
-------------------
|  LINE_WRAP(s,9)  |
-------------------
|5|3|4|6|7|8|9|1|2|
|6|7|2|1|9|5|3|4|8|
|1|9|8|3|4|2|5|6|7|
|8|5|9|7|6|1|4|2|3|
|4|2|6|8|5|3|7|9|1|
|7|1|3|9|2|4|8|5|6|
|9|6|1|5|3|7|2|8|4|
|2|8|7|4|1|9|6|3|5|
|3|4|5|2|8|6|1|7|9|
-------------------

```
