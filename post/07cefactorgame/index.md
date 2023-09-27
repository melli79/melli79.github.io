After we have tested people's attention last time, we want to play some combinatorial game this time.

# 0. Our goal

```log
  Factor game

  Please enter the limit (between 5 and 100): 10

  Players are drawing numbers alternatingly.  The first number must be even.
  Every following number must be either a multiple or a factor of the previous number.
  If you cannot draw any such number, you lose.

  The board: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, Pick an even number
  Player 1's move: 4

  The board: 1, 2, 3, 5, 6, 7, 8, 9, 10, 
  Player 2's move: 8

  The board: 1, 2, 3, 5, 6, 7, 9, 10, 
  Player 1's move: 2

  The board: 1, 3, 5, 6, 7, 9, 10, 
  Player 2's move: 6

  The board: 1, 3, 5, 7, 9, 10, 
  Player 1's move: 3

  The board: 1, 5, 7, 9, 10, 
  Player 2's move: 9

  The board: 1, 5, 7, 10, 
  Player 1's move: 1

  The board: 5, 7, 10, 
  Player 2's move: 7

  The board: 5, 10, 
  Player 1 lost, because there is no choice left.

```


# 1. Basic interaction Loop

```C++
#include <iostream>
#include <set>
using namespace std;

set<int> computeDivisors(int number);

int main() {
    cout << "Factor Game.\n\n";
    ushort limit = 1;
    while (limit<5 || limit>100) {
        cout<<"Please enter the limit (between 5 and 100): ";
        cin>>limit;
    }
    set<int> board;
    for (uint i=1; i<=limit; i++) {
      board.insert(i);
    }
    cout<<"Players are drawing numbers alternatingly.  The first number must be even.\n"
        <<"Every following number must be either a multiple or a factor of the previous number.\n"
        <<"If you cannot draw any such number, you lose.\n";
    short currentPlayer = 1;
    uint lastNumber = 0;
    for (;;) {
        cout<<"The board: ";
        for (uint n : board) {
            cout<<n<<", ";
        }
        if (lastNumber==0) {
            cout<<"\nPick an even number.\n";
        }
        uint choice = 0;
        while (board.find(choice)==board.end() || ...) {
            cout<<"\nPlayer "<<currentPlayer<<"'s move: ";
            cin>>choice;
        }
        board.erase(choice);  lastNumber = choice;
        currentPlayer = currentPlayer==1 ? 2 : 1;
    }
    return 0;
}
```

We start by showing the name of and describing the game.

Then we ask for the board size, i.e. the upper limit (最高限额).  For a reasonable game, we limit to 5..100 numbers.

Before entering the interaction loop, we initialize the `currentPlayer` to 1 and the `lastNumber` to 0.  `0` will be a special marker that no number has yet been entered.

The loop will run indefinitely (永奔), therefore `for (;;) { ... }`.  In the beginning, we will show the remaining numbers.  Therefore, we have inserted all numbers into the `set<int> board` (棋盘) and now loop over this (odered) set.

If this is the first move (and the `lastNumber` is 0), then we will remind the player to pick an even number (偶数).  We will then loop over picking such a `choice`, but we need to work out the condition.  The first part is obvious, i.e. the `choice` must still be on the board, but we need more conditions.  The first condition is already sufficient to repeat the loop, so all following conditions must be connected with `||`.

If `lastNumber==0`, then we also need to test whether the choice is even.  So we add `lastNumber==0&&choice%2!=0`, because we have to repeat if the choice is not even.  If the `lastNumber>0`, then we need to test whether `choice` is not a multiple of `lastNumber` or `choice==0` or `lastNumber` is not a multiple of `choice`.  In either case, we have to loop.  Also the third case (`lastNumber>0`) is an alternative, so we need to connect with `||`.  In total, we modify the `while (...)`-condition as follows:

```C++
    uint choice = 0;
    while (board.find(choice)==board.end() ||
          lastNumber==0&&choice%2!=0 ||
          lastNumber!=0 && (choice%lastNumber!=0) && (choice==0||lastNumber%choice!=0)) {
       ...
```

It is now possible to play the game.  Please give it a try (试试).  Do you notice any problem?

## 1.2 Testing for Game Over (游戏结束)

The beginning looks nice, but once I have no further options, the input keeps me in an infinite loop.  How can we detect that the game is over?

Well, if the `lastNumber` is not 0, then we should check whether there is either a multiple (倍数) or a divisor/factor (整除数) left.  We can do this as follows:

```C++
  ...

  int main() {
  	...
    for (;;) {
      ...
      if (lastNumber==0) {
        ...
      } else {
        bool hasOption = false;
        for (uint m=2*lastNumber; m<=limit; m+=lastNumber) {
          if (board.find(m)!=board.end()) {
            hasOption = true;
            break;
          }
        }
        if (!hasOption)  for (uint d : computeDivisors(lastNumber)) {
          if (board.find(d)!=board.end()) {
            hasOption = true;
            break;
          }
        }
        if (!hasOption) {
          cout<<"Player "<<currentPlayer<<" lost the game, because there is no choice left.\n";
          break;
        }
      }
      ...
    }
    return 0;
  }
```

We start with double the `lastNumber`, because the `lastNumber` was already removed from the board.  Also we only need to check as long as the multiple `m<=limit`, because there are no higher numbers on the board.

It remains to compute the divisors of `lastNumber`
```C++
  set<int> computeDivisors(int n) {
    set<int> result;
    for (int d = 1; d<number; d++) {
      if (n%d==0) {
        result.insert(d);
      }
    }
    return result;
  }
```

You should carefully check the play (试试得很好) again in order to verify that the game indeed works well.

## 1.3 Cleaning up the Code (清理代码) a bit

We have a running program, but remember that the goal of writing a computer program consists not only of writing a compiling (可以编译) and running (可以运行) program.  Firstly, we want to make sure we and hopefully also others understand (可以看懂) our programs.  Secondly, we wish to make obvious (明显) that the program works correctly.

One option is to insert enlightening comments where they are useful.  The following is *not* useful:

```C++
  // This function computes the divisors of a number
  set<int> computeDivisors(int n) {
    ...
  }
```

Why?, because if we choose a meaningful name (有意义的名称) for the function such as `computeDivisors`, then it already says that it will compute the divisors.  Also, we don't have to write “divisors of a number”, because the argument is `int n`, so the number is an integer (and we would rightfully assume that it is used).

What about the following block:

```C++
  bool hasOption = false;
  for (uint m=2*lastNumber; m<=limit; m+=lastNumber) {
    if (board.find(m)!=board.end()) {
      hasOption = true;
      break;
    }
  }
  if (!hasOption)  for (uint d : computeDivisors(lastNumber)) {
    if (board.find(d)!=board.end()) {
      hasOption = true;
      break;
    }
  }
  if (!hasOption) {
    cout<<"Player "<<currentPlayer<<" lost the game, because there is no choice left.\n";
    break;
  }
```

Well it breaks down to 2 parts.  The first part seems to be concerned with finding out whether the board has any options left.  And the last `if`-statement seems to react to the result.  What if we made `hasOption` into a function not just a variable?

We can reach that maybe as follows:

```C++
  bool hasOption(int lastNumber, const set<int>& board);
  set<int> computeDivisors(int number);

  int main(int, char**) {
    ...
    for(;;) {
      ...
      if (lastNumber==0) {
        ...
      } else {
        if (!hasOption(lastNumber, board)) {
          cout<<"Player "<<currentPlayer<<" lost the game, because there is no choice left.\n";
          break;
        }
      }
      ...
    }
    return 0;
  }

  bool hasOption(int lastNumber, const set<int>& board) {
    if (board.empty()) {
      return false;
    }
    const uint limit = *board.rbegin();
    for (uint m=2*lastNumber; m<=limit; m+=lastNumber) {
      if (board.find(m)!=board.end()) {
        return true;
      }
    }
    for (uint d : computeDivisors(lastNumber)) {
      if (board.find(d)!=board.end()) {
        return true;
      }
    }
    return false;
  }
```

Note that this way, we have been able to get rid of the variable `bool hasOption`, because as soon as we have found an option, we simply `return true`.  The default value (默认值) `hasOption = false` is returned as a last resort (最后手段, at the end of the function) if we haven't found any option by then.

We can capitalize on this function in the second part.

# 2. Playing against a Computer

First, we need to modify the interaction a bit, namely we extract the interaction, maybe as follows:

```C++
  ...

  uint interact(int lastNumber, const set<int>& board);
  bool hasOption(...)
  ...

  int main(int, char**) {
    ...
    for (;;) {
      ...
      uint choice = 0;
      if (currentPlayer==1) {
        choice = interact(lastNumber, board);
        if (choice==0) {
          break;
        }
      } else {
        // the AI part
      }
      board.erase(choice);
      lastNumber = choice;
      currentPlayer = currentPlayer==1 ? 2 : 1;
    }
    return 0;
  }

  uint interact(uint lastNumber, const set<int>& board) {
    if (lastNumber==0) {
      cout<<"Pick an even number.";
    } else {
      if (!hasOption(lastNumber, board)) {
        cout<<"\nYou lost, because there is no choice left.\n";
        return 0;
      }
    }
    uint choice = 0;
    while (board.find(choice)==board.end() ||
           lastNumber==0&&choice%2!=0 ||
           lastNumber!=0 && (choice%lastNumber!=0) && (choice==0||lastNumber%choice!=0)) {
      cout<<"\nYour move: ";
      cin>>choice;
    }
    return choice;
  }
```

If you started the game now, then you would see that every second move nothing happens.  That is because we have not yet implemented the computer's part.  We will do that next.  But first, let us see how it fits in:

```C++
  ...
  set<int> computeDivisors(uint number);
  uint goodMove(uint lastNumber, const set<int>& board);

  int main(int, char**) {
    ...
    for (;;) {
      ...
      uint choice = 0;
      if (currentPlayer==1) {
        choice = interact(lastNumber, board);
        ...
      } else {
        choice = goodMove(lastNumber, board);
        if (choice==0) {
          cout<<"I give up, because there is no move left.\n";
          break;
        }
      }
      ...
    }
    return 0;
  }

  ...

  uint goodMove(uint lastNumber, const set<int>& board) {
    return 0; // or do better
  }

```

So it is clear that we are given the board and the `lastNumber` and that we need to decide for a good move.  If there is no option left, we should `return 0`, and that will trigger the game over.

So, what can we do as a good move?  Maybe something along the following lines:
```c++
  uint goodMove(uint lastNumber, const set<int>& board) {
    if (board.empty()) {
      return 0;
    }
    const uint limit = *board.rbegin();
    if (lastNumber==0) {
      return int(drand48()*(limit/2))*2+2;
    }
    vector<uint> options;
    set<int> nextBoard(board);
    for (uint choice : board) {
      if (lastNumber%choice==0 || choice%lastNumber==0) {
        nextBoard.erase(choice);
        if (!hasOption(choice, nextBoard))
          return choice;
        nextBoard.insert(choice);
        options.push_back(choice);
      }
    }
    if (options.empty()) {
      return 0; // we lost
    }
    if (options[0]==1) {
      options.erase(options.begin());
    }
    if (options.empty()) {
      return 1; // we will lose
    }
    // any option, except 1 will be Ok.
    return options[int(drand48()*options.size())];
  }
```

So what it does are 6 things:

0. if there is nothing left on the board, then we cannot choose and lose.
1. We run through the whole board and consider every number that is either a multiple or a divisor and

    a. if it leaves the `nextBoard` with having lost, then we choose it immediately,

    b. otherwise, we consider in `options` (可考虑).

2. If no options are left, we lost.
3. We remove 1 from the options and
4. if there is no other option left, then we return 1 (and will lose)
5. otherwise we return randomly from the remaining options.

## 2.1 Completing the Strategy

If you start playing the game, you will notice that the computer always does the same steps (assuming you do the same steps).  The reason is that we did not initialize (忘记初始化) the random number generator.  We can do that as follows:

```C++
  #include <cstdlib>
  #include <ctime>
  ...

  int main(int, char**) {
    srand48(time(nullptr));
    ...
  }
```

Now, we get at least varying behavior (不同的反应) of the computer.


# 9. Try it for Yourself
We have been talking about a couple of points, but do you know if the game can be played well?  Did you test that the game can be won?  Did you ever lose in this game?

You won't know for sure, before you tried that.  Careful testing is part of Software development (软件开发).

In the Software industry there are at least 2 roles: coders (代码员) and testers (软件测试员).  Both can be Software engineers and in many companies the same people have to take on both roles.  Even if you are just assigned the role of a coder, you should always test your code.  Think about each extreme case, think about what a user can do wrong, what could be misunderstood, ...

One thing you should note is that you always have to move first.  You can change that if after line 27: `ushort currentPlayer = 1;`  you ask the user to choose who shall start.  You have probably observed that when we start with `currentPlayer=1`, then the user will move first.  When we start with `currentPlayer=2`, the computer will move first.

If you have tested the game well for yourself (阿尔法测试), it is time to ask some beta testers.  E.g., you could ask your father (or mother or ...) if they want to have a try.  You should start the program for them, but then you should observe how they interact with the program.  Do they always know what to do?  Do they notice when they have to enter something?  Do they understand when and why they won/lost?

If you have any questions, feel free to ask me.
