This is a re-introduction.  This time I wrote it for Python and in English, hence PE.

Once you have set up the programming environment, you can try to write a first small program.

# The Goal
Today's goal is a multiplication table as follows:

<pre>
   1  2  3  4  5  6  7  8  9 10
   2  4  6  8 10 12 14 16 18 20
   3  6  9 12 15 18 21 24 27 30
   4  8 12 16 20 24 28 32 36 40
   5 10 15 20 25 30 35 40 45 50
   6 12 18 24 30 36 42 48 54 60
   7 14 21 28 35 42 49 56 63 70
   8 16 24 32 40 48 56 64 72 80
   9 18 27 36 45 54 63 72 81 90
  10 20 30 40 50 60 70 80 90 100
</pre>

In words:  The 10 rows (行) contain the multiples of the corresponding number.
In every row the number, then its double (两次), then the triple (三次), ..., the 10-fold (十次).

# What Elements do we need?

1. A *variable* (变量/变数) can store a certain value, e.g.
```Python
  x = 3
```
means that we create a variable named *x* and give it the value 3.

Correspondingly
```Python
  name = "Mel"
```
is a variable named `name` (名) that contains the text *Mel* (my name).

Of course, we do not want to type in the multiples manually, instead a computer should be able to compute it.  We can reach that in the following way:
```Python
  x*y
```
Assuming the variables `x` and `y` contain numbers, then `x*y` means that their product is computed.  Correspondingly, we can compute a sum by `4+5`.

## 2. What to do with these numbers?

We will print them on the command line.  The simplest option for that is:

```Python
  print(x*y)
```
Everything between ( and ) will be printed by the print command.  Print derives from printing on paper, because the first computers did not have any montinor (or screen), but instead printed the results on paper (or displayed it with electronic glowing tubes).  

## 3. We need one more ingredient

We want for every row 10 expressions and repeat that for 10 rows.

For one row this looks as follows

```Python
  y = 5
  for x in range(1, 11):
      print(x*y, end=" ")
  print()
```

The first expression should be familiar `y = 5` means that the variable `y` gets the value 5.
`for x in range(1, 11):` means that the following indented expression will be reapeated for all numbers from 1 below 11. In the first iteration, we have `x=1`, in the second iteration `x=2` and so on until `x=10` (the last number below 11).

You should also remember the expression `print(x*y, end=" ")`.  It computes the product and displays it on the command line.


## 4. How do we form the whole table?

We should repeat the above block not only for `y=5`, but need to repeat it for `y` in `1..10` as follows:

```Python
  for y in range(1, 11):
      for x in range(1, 11):
          print(x*y, end=" ")
      print()
```

# How can I Start the Program

If you set up your programming environment properly and wrote the programm in a new project, all you have to do is click the green arrow on the top bar and immediately the program will be executed.

## Where can I see the result?

When you started the program as explained above, there will be a new sub window in which you can the output of the program (that should look as the picture in the beginning).  In this window you will maybe see even more output before, maybe you have to scroll to the end.


# How can I Improve the Program?

When you look at the output, you will see that all numbers in a row are printed one after the other with only one space.  This means that the numbers in different rows are not aligned, because the earlier rows contain more 1-digit numbers.  We can fix that by expanding every number to 2 digits as follows:

```Python
  print("{:2}".format(x*y), end=" ")
```
`.format(...)` means that the following arguments will be inserted into the text before where there are `{...}`.  `{:2}` means that the numbers will be printed with 2 positions (or more) by padding them with spaces from the left if necessary.

## What happens with 100? (It needs 3 digits.)

Have a look at the (new) output.  Even though we requested only 2 digits, the number 100 is displayed with 3 digits.  The reason is that `{:2}` means at least 2 positions, more when the number is bigger.

## I want to print a bigger Multiplication table, like 20x20

Have a look at the line `for y in range(1, 11):`.  11 means that the last iterations runs with `y=10`.  So what is the fist change?

### How do I make the Table square shaped?

We have 2 for loops `for y in ...:` and `for x in ...:`, i.e. you also have to adjust the 2nd for loop.

Another problem may be that from the middle of the table most numbers exceed the 2 digits.  You can expand all numbers to 3 characters by adjusting to `{:3}.format(...)`.

Have fun trying out different things...

Next time, we will talk about conditional commands and counting smurfs.
