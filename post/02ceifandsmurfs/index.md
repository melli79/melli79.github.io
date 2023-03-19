Today we want to count smurfs (蓝精灵).  This is how it works:

0. We start with the ordinary numbers from 1: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, ...

1. Then we replace every 4th number by "smurf" according to its multiplicity (几次):
```log
  1, 2, 3, smurf, 5, 6, 7, 2-smurf, 9, 10, 11, 3-smurf, 13, 14, 15, ...
```

3. 16 is smurf-smurf.

#  0. The numbers
Probably you remember the for loop, that allows us to count, like this:

```C++
  for (int n=1; n<101; n++)
  {
    cout<<n<<", ";
  }
  cout<<endl;
```

# 1. Smurfs
It seems that we need to test for every number whether it is divisible by 4.  This can be done as follows:
```C++
  n%4 == 0
```

The term means that the computer computes the remainder (余数) of `n` under division (除法) by 4, and then checks whether it is 0.  For example `1%4=1` not equal to 0, `2%4=2`, `3%4=3`, `4%4=0` yes, `5%4=1` and so on.

Whenever `n%4==0`, we need to write "smurf", otherwise the number `n` as a String.  How to do that in C++?  C++ derives from English so we write `if` (如果, not when -- 每次, because it is not clear, whether the condition is ever fulfilled), otherwise is shorter written as `else` (否则), thus:

```C++
  if (n%4 == 0)
  {
    cout<<"smurf"<<", ";
  } else {
    cout<<n<<", ";
  }
```

But we do not wish to squeeze (夹) everything into the main loop. Instead we want to put the conversion to smurf into a function (函数/方法):

```C++
  string smurfify(int n)
  {
    ostringstream result;
    if (n%4 != 0)
    {
      result<<n;
    } else
    {
      result<<"smurf";
    }
    return result.str();
  }
```

Our function is called `smurfify` (改为蓝精灵, conversion to a smurf, a pun on the English language).  For that we need to know the number that needs to be smurfified, thus we expect a number `(int n)`.  Within the function, the number is called `n`, and `int` means that an integral number (整数, not $0.5$ or a string "Hello") is passed in.

But we also want to return a result (结果).  Therefore we write `string smurfify(...)`, i.e. in the end we return a string (字串) or text.

What are the 2 cases?  We check if `n` divided by 4 gives the remainder not equal `!=` 0, that is for 1, 2, 3 or 5, 6, 7 or ... .  In this case we add `result<<n` the formatted number.

When `n` is divisible by 4 (`n%4 == 0`), the else-branch is executed.  Here we add `result<<"smurf"`.

In the end we return (返回给) more-or-less the result.  But the result is an output `stringstream`, so instead we want its string.  This is achieved as follows: `return result.str()`.

Now, our program looks as follows:

```C++
  #include <iostream>
  #include <string>
  #include <sstream>
  using namespace std;

  string smurfify(int n)
  {
    ostringstream result;
    if (n%4 != 0)
    {
      result<<n;
    } else
    {
      result<<"smurf";
    }
    return result.str();
  }

  int main()
  {
    for (int n=1; n<31; n++)
    {
      string txt = smurfify(n);
      cout<<txt<<", ";
    }
    cout<<endl;
  }
```

The result is:

```log
  1, 2, 3, smurf, 5, 6, 7, smurf, 9, 10, 11, smurf, 13, 14, 15, ...
```

We have reached half of the goal: every 4th number is replaced by "smurf".  Yay!!


# 2. How to annotate the multiples?

Actually, we wanted to replace 8 by "2-smurf".  The 2 is $8÷4$.  We can reach this by:

```C++
  string smurfify(int n)
  {
    ostringstream result;
    if (n%4 != 0)
    {
      result<<n;
    } else
    {
      n /= 4;
      result<<n<<"-smurf";
    }
    return result.str();
  }
```

`n /= 4` means that we divide `n` by 4 and store the result again in `n`.

When you restart the program, the result is the following:

```log
  1, 2, 3, 1-smurf, 5, 6, 7, 2-smurf, 9, 10, 11, 3-smurf, 13, 14, 15, 4-smurf, 17, ...
```

## 2.1 Remove the "1-"

Ok, it seems that we have to distinguish another case:  If `n` is 1 after the division, then we only want to write "smurf".  We can reach there by:

```C++
  string smurfify(int n)
  {
    ostringstream result;
    if (n%4 != 0)
    {
      result<<n;
    } else
    {
      n /= 4;
      if (n==1)
      {
        result<<"smurf";
      } else
      {
        result<<n<<"-smurf";
      }
    }
    return result.str();
  }
```

And now:

```log
  1, 2, 3, smurf, 5, 6, 7, 2-smurf, 9, 10, 11, 3-smurf, 13, 14, 15, 4-smurf, 17, ...
```


# 3. "16" is smurf-smurf

There is still a problem left:  16 is not "4-smurf", but `n` should be reduced to 1 and the suffix is "smurf-smurf".  We can get there with the following code:

```C++
  string smurfify(int n)
  {
    ostringstream result;
    ...
    {
      n /= 4;
      string suffix = "smurf";
      while (n%4 == 0)
      {
        n /= 4;
        suffix += "-smurf";
      }
      if (n == 1)
      {
        result<<suffix;
      } else
      {  
        result<<n<<"-"<<suffix;
      }
    return result.str();
  }
```

`while (<condition>)` means that the `condition` (条件) is tested (试试).  When it is fulfilled (true, 真), the code block is executed.  Then the condition is tested again, and so on, until the condition is no longer true (while -- 当……时. means also as long as).  This is called a while loop (while循环).  The difference to a for loop is that we do not know how many times we need to repeat, but instead we can test whether we should repeat.  In a for loop we need to know from the beginning how many times we want to repeat.

`suffix += "-smurf"` means that we append (追加) "-smurf" once.  After the first pass, `n` is smaller by a factor of 4 and `suffix == "smurf-smurf"`.  Suffix (后缀) means an ending, the opposite would be a prefix (前缀).

Now, the result looks as follows:

```log
  1, 2, 3, smurf, 5, 6, 7, 2-smurf, 9, 10, 11, 3-smurf, 13, 14, 15, smurf-smurf, 17, 18, 19, 5-smurf, 21, 22, 23, 6-smurf, 25, 26, 27, 7-smurf, 29, 30,
```

So we are done.  Yippie!!!


# 9. Try for yourself

First you should try to run the program by yourself.  Does it start?  Can you see any output?  Are there any problems/errors?  If you are stuck, you can either ask your dad or you ask me (e.g. when we meet the next time).

## 9.1 One, two, three -- Congee

Can you write a function `string congee(int n)` that replaces every 3rd number by "congee" (粥)?  Congee in English sounds very similar to three, that is why we replace every 3rd number.

How to adjust the main loop such that it uses the new function?

Can you refine (完善/提高) the function a bit?
