This is a re-introduction.  This time I wrote it for C++ and in English, hence CE.

Once you have set up a programming environment, you can try to write a first small program.

# The Goal
Today's goal is a multiplication table (乘法表) as follows:

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
```C++
  int x = 3;
```
means that we create a variable named *x* and give it the value 3, which is an integer (整数) hence the `int` in the beginning.

Correspondingly
```C++
  string name = "Mel";
```
is a variable named `name` (名字) that contains the text *Mel* (my name), text or `string` means a sequence of characters (字串).

Of course, we do not want to type in the multiples manually, instead a computer should be able to compute it.  We can reach that in the following way:
```Python
  x*y
```
Assuming the variables `x` and `y` contain numbers, then `x*y` means that their product (求积) is computed.  Correspondingly, we can compute a sum (求和) by `4+5`.

## 2. What to do with these numbers?

We will output them on the command line (命令行).  The simplest option for that is:

```Python
  cout<<x*y;
```
Everything between `<<` and `;` will be printed/output on the command line.  `cout` derives from C/C++ output.


## 3. We need one more ingredient

We want for every row 10 expressions and repeat that for 10 rows.

For one row this looks as follows[^1]

```C++
  int y = 5;
  for (int x=1; x<11; x++)
  {
    cout<<x*y<<" ";
  }
  cout<<endl;
```

[^1]: Even though I am personally a fan of Kernighan & Ritchie's egyptian style, many programming schools today prefer the Allman style (i.e. braces of matching identation each on a separate line).  Also my friend learned this style at school.

The first expression should be familiar `int y = 5` means that the variable `y` gets the value 5.
`for (int x=1; x<11; i++)` means that the following block / indented command(s) will be reapeated for all numbers from 1 below 11. In the first iteration (迭代), we have `x=1`, in the second iteration `x=2` and so on until `x=10` (the last number below 11).  This is called a for loop (for循环).

The additional `cout<<endl;` means that we finish the output row after the loop.


## 4. How do we form the whole table?

We should repeat the above block not only for `y=5`, but need to repeat it for `y` in `1..10` as follows:

```C++
  for (int y=1; y<11; y++)
  {
    for (int x=1; x<11; x++)
    {
      cout<<x*y<<" ";
    }
    cout<<endl;
  }
```

## 5. How can I make this a whole program?
For that you need 2 additional ingredients.

First, the whole set of instructions has to go into a function (函数/方法) called `main`.  The reason is that the computer needs to know where to start with your program.  By convention this is in the function `main` as follows:

```C++
  int main()
  {
    for (int y=1; y<11; y++)
    {
      ...
    }
    return 0;
  }
```

One part of the convention is that this function is called `main` (主), another convention is that it should return an `int`.  If everything went well, we should return 0, thus the program ends with `return 0;` (返回给).

## 6. Problems compiling the program

When I try to compile the program, the compiler complains that it does not know the object `cout`, the operator `<<` is not applicable and it does not know `endl` either.  Somehow something important is missing.

Yes, indeed.  All 3 elements are contained in the standard library and we need to tell the compiler where to find them and that it should use them.  Therefore, the program should start as follows:

```C++
  #include <iostream>
  using namespace std;

  int main()
  {
    ...
  }
```

Now, the program should compile and you should even be able to run it.


# 7. How can I Start the Program?

If you set up your programming environment properly and wrote the programm in a new project, all you have to do is click the run botton on the top bar and immediately the program will be executed.

## Where can I see the result?

When you started the program as explained above, there will be a new window (窗) in which you can see the output (输出) of the program (that should look as the picture in the beginning).


# 8. How can I Improve the Program?

When you look at the output, you will see that all numbers in a row are printed one after the other with only one space.  This means that the numbers in different rows are not aligned in columns (在列中对齐), because the earlier rows contain more 1-digit numbers.  We can fix that by expanding every number to 2 digits (数字) as follows:

```C++
  cout<<setw(2)<<x*y<<" ";
```
`<<setw(2)` means that the following number will be printed with 2 positions (or more) by padding them with spaces from the left if necessary.

## additional library
In order for the compiler to find the method `setw` we need an additional `#include` directive, namely for the library "iomanip", e.g. as follows:

```C++
  #include <iostream>
  #include <iomanip>
  using namespace std;
```


## What happens with 100? (It needs 3 digits.)

Have a look at the (new) output.  Even though we requested only 2 digits, the number 100 is displayed with 3 digits.  The reason is that `<<setw(2)` means at least 2 positions, more when the number is bigger.

## 9.1 I want to print a bigger Multiplication table, like $20\times20$

Have a look at the line `for (int y=1; y<11; y++)`.  `11` means that the last iteration runs with `y=10`.  So what is the first change?

### How do I make the Table square shaped (方形)?

We have 2 for loops `for (int y=...)` and `for (int x=...)`, i.e. you also have to adjust the 2nd for loop.

Another problem may be that from the middle of the table on, most numbers exceed the 2 digits.  You can expand all numbers to 3 characters by adjusting to `<<setw(3)<<...`.


# 9. Your time to try out things

Please use the opportunity to try out different things...

Next time, we will talk about conditional commands and counting smurfs.
