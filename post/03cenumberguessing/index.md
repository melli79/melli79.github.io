After we have counted smurfs last time, we want to guess some numbers this time.

# 0. Our Goal

Here are 2 sample sessions (样本会议):

```log
  Number guessing Game
  Do you want to guess my number or should I guess your number (y/I)? I

  Ok, I have chosen a number between 1 and 100 (both inclusive).
  What is your guess? 50
  My number is bigger.
  What is your guess? 75
  My number is smaller.
  What is your guess? 62
  Yes, that's it!  You got it right!!
```

```log
  Number guessing Game
  Do you want to guess my number or should I guess your number (y/I)? y

  Ok, choose a number between 1 and 100 (both inclusive), but don't tell me yet.  Instead I ask:
  Is it 50 (<=>)? <
  Is it 25 (<=>)? >
  Is it 37 (<=>)? =
  Wow, I guessed it.  Your number is 37!
```

# 1. You guess my number

We will write this first case in a function which will then be called by the `main` part.  We do not have to return anything and as a first guess we do not need any programmatic arguments.  Therefore we start with:

```C++
  void letGuess()
  {
    cout<<"Ok, I have chosen a number between 1 and 100 (both inclusive)."<<endl;
    ...
  }
```
Here `void` means that we are returning nothing (specifically), just the control (程序流程).  Therefore, we do not need any `return` statement.

## 1.1  Choosing a secret number

So, first we need to choose a number.  We want to make sure it is a different number every time.  Therefore, we initialize with a random number.  While the function `rand()` gives an integer between 0 and `MAX_RANDOM`, we want to use a more C++ way.  There are 2 classes in C++ (library "random") that allow us to perform that:  First there is the class `mt19937_64` that produces a random sequence (随机序列), and second there is `uniform_int_distribution` that produces integers between a minimum (最低) and a maximum (最高).

The problem with (pseudo) random numbers is that they always require seeding (播种), i.e. the careful initialization (来初始化) such that you do not always obtain the same sequence.  Therefore we will assume that a random number generator is passed to our function.

How do we make a number between 1 and 100?

The answer is `uniform_int_distribution rollDice(1, 100)`.  This object `rollDice` (掷骰子) has one method, namely the call `rollDice(random)` produces an integer between 1 and 100 both ends inclusive.

```C++
  void letGuess(Random random)
  {
    uniform_int_distribution rollDice(1, 100);
    int secret = rollDice(random);
    cout<<...
  }
```

Why did I name it `secret` (秘密) and not `a`?  The compiler does not care whether you name it `a`, or `secret` or `i32`, but your fellow programmers (同事/同学) do mind.  I.e. we want another reader (and that could be yourself in 6 months) to know what the variable was good for.  Therefore I call it `secret`.

## 1.2 Your input
Next, we need to ask the user for a number.  In C++ this is done via `cin>>guess;` but first we need to define a variable, here `guess`, that takes the input.

```C++
  fun letGuess()
  {
    ...
    cout<<...

    cout<<"Your guess: ";
    int guess=0;
    cin>>guess;
    ...
  }
```

## 1.3 My Answer
Then we need to tell the user whether the actual number is bigger, smaller or exactly the guess.  We can do that with 3 if branches as follows:

```C++
  fun letGuess()
  {
    ...
    cin>>guess;
    if (guess<secret)
    {
      cout<<"My number is bigger"<<endl;
    } else if (guess>secret)
    {
      cout<<"My number is smaller."<<endl;
    } else
    {
      cout<<"  Yes, that's it!  You got it right!!"<<endl;
    }
    ...
  }
```

## 1.4 Loop (循环)
If the user cannot guess the number in the first run, then they probably want to guess again (the same number).  So we need to put a loop around it.  But how many times?

Since we don't know how many times it will take the user, we should do a "forever" loop (永远循环), e.g. as follows:

```C++
  void letGuess()
  {
    ...
    cout<<...
    while (true)
    {
      cout<<"Your guess: ";
      ...
    }
  }
```


# 2. So does that work? -- We need a main program
I don't know (or honestly, I do not yet want to tell you where it fails).  So let us give it a try.  Therefore we need a `main` program.  At the moment, the most important thing, the main program has to do is call our function `letGuess()`.  This can be achieved as follows:

```C++
  #include <iostream>
  #include <random>
  #include <chrono>
  using namespace std;

  typedef mt19937_64 Random;

  Random newRandom() {
    uint64_t now = chrono::high_resolution_clock::now().time_since_epoch().count();
    seed_seq seed{uint32_t(now & 0xffffffff), uint32_t(now>>32)};
    return Random(seed);
  }

  ...

  int main()
  {
    Random random = newRandom();
    cout<<"  Number guessing Game"<<endl
        <<"Do you want to guess my number or should I guess your number (y/I)? I"<<endl;
    letGuess(random);
    return 0;
  }
```

So what happens when we try that program?

The first steps seem promising, i.e. the program asks us for a number and then tells us whether its secret number is smaller or bigger.

But what happens when we finally guess the number?  The program clearly acknowledges (确认) that, but then it keeps asking for our guess and giving the same answer again and again, the program does not want to terminate (不完).

What went wrong?

We forgot to tell the compiler that the "forever" loop should be stopped.  This can be achieved in the following way:

```C++
  void letGuess(Random random)
  {
    ...
    while (true)
    {
      ...
      {
        cout<<"Yes, that's it!  You got it right!!"<<endl;
        break;
      }
    }
  }
```

The statement `break;` means to break out of the loop (退出循环).

Instead you could also have written `return;` i.e. a return statement without any argument.  Because our function is sufficiently small, i.e. there is nothing else to be done after the loop, the effect is the same.


## 3. I guess your number
Similar to before, we want to write that in a function.  So we start as follows:

```C++
  void guess()
  {
    cout<<"Think of a number between 1 and 100 (both inclusive), but do not yet tell it to me.  I will ask you a couple of guesses."<<endl;
    ...
  }
```

## 3.1 Knowing the bounds

So what does the computer know about your number?  The lowest possible number is 1 and the highest possible number is 100.  We should store these values.

```C++
  void guess()
  {
    cout<<...
    int low = 1;  int high = 100;
    ...
  }
```

Now we probably need to take a couple of guesses and think about the answers.  Therefore, we put the remaining code in a loop.  But should it be infinite?  We don't know the answer as long as `low<high`.  Once we have reached `low==high`, there is a unique solution.  If even `low>high`, then something is wrong.  So we formulate:

```C++
  void guess()
  {
    ...
    while (low<high)
    {
      ...
    }
  }
```

## 3.2 What to do with the Answer?

If the answer is "right", then we have guessed the number.  If the answer is "bigger", then we know that the new lower limit (`low`, 下限) is our guess plus 1.  If the answer is "smaller", we know that the new `high` (上限) is the guess minus 1.  So let us put that together.

```C++
  void guess()
  {
    ...
    while (low<high)
    {
      int guess = ...;
      cout<<"Is it "<<guess<<" (<=>)? ";
      string input;
      cin>>input;
      if (input.length()>0 && (input[0]=='<' || input[0]=='l'))
      {
        high = guess - 1;
      } else if (input.length()>0 && (input[0]=='>' || input[0]=='b'))
      {
        low = guess + 1;
      } else if (input.length()>0 && (input[0]=='=' || input[0]=='r'))
      {
        cout<<"Wow, I guessed it!  Your number is "<<guess<<"!!"<<endl;
        return;
      } else
      {
        cout<<"Sorry, I did not understand your answer."<<endl;
      }
    }
  }
```

## 3.3 Having a good guess -- binary search

So, given the lowest possible number and the highest possible number, what is a reasonable guess?  We could ask whether it is the the lowest possible number.  Then the 2 possible answers are "right" or "bigger".  But it is impossible that the answer is "less", because we already asked for the lowest possible answer.

So how can we do better?

We could ask for `low+1` and then at least expect all 3 answers.  But what would you do if you are standing in front of an alphabetically sorted (按字母顺序排列的) bookshelf (书架) and you are looking for a book by "Rowling"?

You would best look in the *middle of the bookshelf* and decide whether the book there is (written by an author) before "Rowling" or after.  And that is the best idea:  We take a guess in the middle (平均数).

```C++
  void guess()
  {
    ...
    while (low<high) {
      int guess = (low+high)/2;
      cout<<...
    }
  }
```

So what happens if the sum `low+high` is odd (奇)?  The answer is that the computer rounds it to the lower integer, e.g. if the low is 1 and the high is 6, then their sum is 7 and half of it is `7/2 == 3`.


## 3.4 After the loop (while循环后)

What happens if we leave the loop (I mean other than via `return`)?

The answer is that it depends:  If `low` is bigger than `high`, then there is certainly something wrong, e.g. the user cheated (受骗了).

If `low==high`, then we also know the number.  So in total we conclude:

```C++
  void guess()
  {
    ...
    while (low<high)
    {
      ...
    }
    if (low>high)
    {
      cout<<"That is not possible, because there is no number between "<<low<<" and "<<high<<"."<<endl;
    }else
    {
      cout<<"So your number must be "<<low<<"."<<endl;
    }
  }
```


## 4. Adjusting the main program

If you just want to test the second function `guess()`, then you could modify the main program, e.g.
```C++
  ...
  #include <string>
  ...

  int main()
  {
    cout<<"... (y/I)? y"<<endl;
    guess();
    return 0;
  }
```

You should try your implementation a couple of times in this configuration.  Does it work as expected?  Does it terminate (完好)?  What happens if the user just hits \<Return\> (返回键)?

Once you have convinced yourself that the function `guess()` is working, you should complete the main program.  Instead of giving the answer, we should actually ask the user for their choice and then decide to either call `guess()` or `letGuess(...)`.  We can do that, e.g. as follows:

```C++
  int main()
  {
    ...
    cout<<"Do you ... (y/I)? ";
    string choice;
    cin>>choice;
    if (choice.length()>0 && (choice[0]=='y' || choice[0]=='Y'))
    {
      cout<<endl;
      guess();
    }else if (choice.length()>0 && (choice[0]=='I' || choice[0]=='i'))
    {
      cout<<endl;
      letGuess(random);
    } else
    {
      cout<<"I did not understand your answer."<<endl;
    }
    return 0;
  }
```

# 9. Time to Try for Yourself

So does your program work, i.e. does it compile and start?  Does it react as expected?  Does it terminate?

Does the user know what to do?  You can test the program with somebody else, e.g. one of your parents.
