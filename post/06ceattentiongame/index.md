After we have represented people in the computer last time, we want to test the user's attention this time.

# 0. Our goal

```log
  Attention game

  I give you a couple of numbers and you are to find which number occurs more than twice.

  I say: 3 4 0 9 9 5 2 0 9 4 6 3 5 7 8
  your choice: 9

  That's correct!  You get 1 point.



  I say: 2 3 9 8 5  7 2 3 9 8  4 7 5 6 4
  your choice: 4

  That's wrong!  You lose 1 point.



  I say: 3 9 4 5 8  3 9 0 4 8  7 5 2 1 6
  your choice: -

  You passed.  You have 0 points after 3 turns.
```

When you reach 10 points, you win the game.

When you fall below -10 points, you lose the game.


# 1. How to present 15 random numbers to the user?
## 1.1 Generating random numbers

You can do this with `random()` but it produces a random number between 0 and `RAND_MAX`.  If you want a random number between 0 and 10, you could, e.g. use `random()%10`.  Let us put that in a function.

```C++
  short nextDigit()
  {
  	return random() % 10;
  }
```

But in order to give the random number generator a good start, we need to seed (加种子) it, e.g. with the current time as follows:

```C++
  #include <cstdlib>
  #include <ctime>
  using namespace std;

  short nextDigit()
  {
    ...
  }

  int main()
  {
    srand(time(NULL));
    ...
    return 0;
  }
```

## 1.2 How do you make that 15 digits?
```C++
  #include <iostream>
  ...

  int main()
  {
  	...
    for (int n=1; n<=15; n++)
    {
      cout<<nextDigit<<" ";
    }
    cout<<endl;
    ...
    return 0;
  }
```

## 1.3 Reading an answer from the user
The core is `cin>>input`, but before you do that, you want the user to know that you expect input, e.g. as follows:

```C++
  int main()
  {
  	...
  	char input;
  	cout<<"Your choice: ";
  	cin>>input;
  	...
  }
```

## 1.4 Deciding whether the user is right
For that we need two things.  First, we need to store the digits, e.g. in a `vector<short>` and secondly, we need to decide whether the input is a digit or something else.

```C++
  int main()
  {
    ...
    int points = 0;
    vector<short> numbers;
    for (int n=1; n<=15; n++)
    {
      short digit = nextDigit;
      cout<<digit<<" ";
      numbers.push_back(digit);
    }
    char input;
    ...
    if ('0'<=input && input<='9')
    {
      short choice = input-'0';
      if (count(choice, numbers)>2)
      {
        cout<<"That's right! You get 1 point."<<endl;
        points++;
      }
      else
      {
        cout<<"That's wrong!  You lose 1 point"<<endl;
        points--;
      }
    }
    ...
  }
```

## 1.5 How do we count (计数) how often a number occurs?
In other words, we need to implement the function `count`. Obviously it takes 2 arguments:  The `choice` of the user and the `numbers` (that is why we stored them beforehand).  Then we run over all numbers and compare whether the current number equals the choice.

```C++
  int count(short choice, const vector<short>& numbers)
  {
    int amount = 0;
    for (short num : numbers)
    {
      if (num==choice)
      {
        amount++;
      }
    }
    return amount;
  }
```


## 1.6 What if the user passes (通过)?
Then we should show them how many points they already reached.  For that we have counted the number of points.

```C++
  int main()
  {
    ...
    int points = 0;
    int round = 0;
    ...
    round++;
    if ('0'<=input && input<='9')
    {
      ...
    }
    else
    {
      cout<<"You passed.  You have "<<points<<" points in "<<round<<" rounds."<<endl;
    }
    ...
  }
```

## 1.7 The user wants to play more than 1 round
So how many rounds?  Since we don't know, we will put that in an infinite loop.

```C++
  int main()
  {
    ...
    int points = 0;
    int round = 0;
    while (true)
    {
      round++;
      vector<short> numbers;
      for (int n=1; n<=15; n++)
      {
        ...
      }
      ...
      cin>>input;
      ...
      cout<<endl<<endl;
    }
    return 0;
  }
```

## 1.8 But somewhen the user should stop
So how do we want to decide that?  We could say that the user wins if they reach 10 points.  And we could say that the user loses if they drop to -10 points.

```C++
  #include <iostream>
  ...

  int main()
  {
    ...
    while (true)
    {
      ...
      if (points>=10)
      {
      	cout<<"You have reached the goal.  You win!"<<endl;
      }
      if (points<=-10)
      {
      	cout<<"You lose!"<<endl;
      }
    }
    cout<<"You got "<<points<<" points in "<<round<<" rounds."<<endl;
    return 0;
  }
```


# 2. How many numbers do we have to show?

This is a more tricky question.  But we can describe the limits of that.

Obviously, we need at least 3 numbers, otherwise there cannot be 3 equal numbers.

If we show 11 numbers, then we know by the pigeon hole principle (鸽子洞原则) that there will be at least 1 number twice -- you have 11 pigeons and 10 holes, so 2 pigeons will go in 1 hole.

If we show 21 numbers, then we know by the same pigeon hole principle that there will be at least 1 number three times.

So the answer is somewhere between 3 and 21.

You can experiment with the number of numbers (the upper limit for `n`) in order to see what produces 3 equal numbers in many times, but not too many 3-fold numbers.  I thought that 15 is a reasonable choice, but maybe 12 is already sufficient.


# 3. What if we want to punish reluctant (勉强的) users?
Ok, so the user could always choose '-' and never get forward or backwards.  If the user does not choose any number, then we could check whether there is actually a number more than twice and if so, punish the user (deduct a point, 扣一分).  How do we do that?

## 3.1 Counting which number occurs how often

We need a kind of bucket counting / bucket sort (桶排序), i.e. we iterate over all numbers and increment the bucket for the current number.  In the end if any bucket reaches more than 2, then we return this bucket, otherwise return -1.

An easy version of such buckets is a map (映射), i.e. `map<short, int>` which assigns to a `short` number an `int` amount.  We access the buckets with `[num]`.

```C++
  #include <map>
  ...

  short whichNumberOften(const vector<short>& numbers, int threshold =2)
  {
    map<short, int> buckets;
    for (short num : numbers)
    {
      buckets[num]++;
    }
    for (short num : buckets.keys())
    {
      if (buckets[num]>threshold)
      {
        return num;
      }
    }
    return -1;
  }
```

## 3.2 How do we use that in the main loop?

```C++
  ...
  int main()
  {
    ...
    while (true)
    {
      ...
      if ('0'<=input && input<='9')
      {
        ...
      }
      else
      {
      	short num = whichNumberOften(numbers);
        if (num>=0)
        {
          cout<<"You ignored "<<num<<"!  That costs you 1 point."<<endl;
          points--;
        }
        else
        {
          cout<<"You passed.  You have "<<points<<" points after "<<round<<" rounds."<<endl;
        }
      }
      ...
    }
    return 0;
  }
```


# 9. Try it for Yourself
We have been talking about a couple of points, but do you know if the game can be played well?  Did you test that the game can be won?  Did you ever lose in this game?

You won't know for sure, before you tried that.  Careful testing is part of software development.

In the Software industry there are at least 2 roles: coders and testers.  Both can be Software engineers and in many companies the same people have to take on both roles.  Even if you are just assigned the role of a coder, you should always test your code.  Think about each extreme case, think about what a user can do wrong, what could be misunderstood, ...
