After we have counted smurfs and guessed numbers last time, we want to print some numbers this time.

# 0. Our Goal

Here are 2 samples (样本会议):

```log
  Numbers in Chinese
  0：零、1：一 、2：二、3：三、4：四、5：五、6：六、7：七、8：八、9：九、10：十、11：一十一、
  20：二十、21：二十一、……、30：三十、40：四十，……
  200：二百、……
  2000：二千、……
  2‘0000：二万，……
  20’0000：二十万，……
  200‘0000：二百万，……
  2000’0000：二千万，……
  2‘0000’0000：二亿，……
  20‘0000’0000：二十亿，……
  无穷
```

For comparison, the English version looks as follows:

```log
  Numbers in English
  0: zero, 1:one, 2:two, 3:three, 4:four, 5:five, 6:six, 7:seven, 8:eight, 9:nine, 10:ten, 11:eleven, 12:twelve, 13:thirteen, 14:fourteen, 15:fifteen, 16:sixteen, 17:seventeen, 18:eighteen, 19:nineteen,
  20:twenty, 21:twentyone, ..., 30:thirty, 40:fourty, 50:fifty, 60:sixty, 70:seventy, 80:eighty, 90:ninety, 
  100:onehundred, 110:onehundredten,
  200:twohundred, 300:threehundred, ...
  1000:onethousand, 2000:twothousand, 3000:threethousand, ...
  20'000:twentythousand, ...
  200'000:twohundredthousand, ...
  2'000'000:two million, ...
  20'000'000:twenty million, ...
  200'000'000:twohundred million, ...
  2'000'000'000:two billion, ...
  infinity
```


# 1. Chinese numbers
So we wish to use that in something like the following main function:

```C++
  #include <iostream>
  #include <string>
  using namespace std;

  int main()
  {
    for (unsigned n=0; n<=20; n++)
    {
      cout<<n<<":"<<zhCount(n)<<", ";
    }
    cout<<endl;
    return 0;
  }
```

`unsigned` (无符号)  means that our numbers do not have any sign, like $-1$ or so.

`zh` stands for zhongguohua and is the international abbreviation for the Chinese language.  Other abbreviations are `en` for English (language) or `de` for Deutsch (German language).


## 1.1 Basic Settings

So we must write a function like

```C++
  string zhCount(unsigned n)
  {
    string result;
    ...
    return result;
  }
```

If you remember your fist year Chinese writing lectures, you have learned the following numberals (数词):

```C++
  static vector<string> zhDigits = { "零", "一", "二", "三", "四",  "五", "六", "七", "八", "九" };
  static vector<string> zhTens = { "", "十", "百", "千" };
  static vector<string> zhTenthousands = { "", "万", "亿" };
```

(OK, the last three probably in grade 4 or so.)


## 1.2 Extracting the digits

First, we wish to extract the digits (数位) from our number.  We can obtain the last digit if we compute `n%10`.  Then we divide the number by 10 and repeat the whole thing until we have reached 0.  We need to store the numbers in a vector (向量) as follows:

```C++
  #include <vector>

  typedef unsigned char  unsigned_byte;

  string zhCount(unsigned n)
  {
    string result;
    vector<unsigned_byte> digits;
    while (n>0)
    {
      digits.push_back(unsigned_byte(n%10));
      n /= 10;
    }
    ...
  }
```

A vector is a container in which an arbitrary number of elements can be stored, e.g. a `vector<int>` is a container of integers.  The order of the elements is preserved.  Vectors are efficient for storing, adding new elements at the end, and if a vector is sorted, then it is efficient to search for an element (see binary search in our lecture about the number guessing game).


### What is `unsigned_byte`?

Well as it suggests, a byte-sized number without sign.  One byte (字节) has 8 bits (位) and thus an `unsigned_byte` can capture integers from 0 up to $2^8-1=255$.  This is enough for digits, because they should only be 0..9.

Don't foget the underscore `_`.  In C++ you can introduce new type names, but they must be one "word" (no spaces allowed).


## 1.3 Printing the digits
Once, we have extracted all digits, we must print them one by one.  But this time, we know how many interations we need.  Therefore, we will use a for-loop.  But we want to go backwards (向后/后退) through the digits.  This can be done as follows:

```C++
  string zhCount(unsigned n)
  {
    ...
    for (int pos = (int) digits.size()-1; pos>=0; pos--)
    {
      auto digit = digits[pos];
      result += zhDigits[digit];
    }
    return result;
  }
```

Here the `pos--` means that we decrease (减少) the current position (位置) by one.  If you omit the `(int)`, then the `digits.size()` will be marked yellow (a sign for a warning, at least in my IDE CLion), because the computer is not sure if you wish to convert the `size_t size()` to an `int`.

Note that after the last run, the computer will decrease `pos` to `-1` which is negative and stop, because `-1>=0` is false.  Therefore, we need `pos` to be of type `int`, not `unsigned`.


## Test the Program

Before we continue, we should test the program.  There is already some main program, so we can just start it.

Please give it a try and see what it produces.  Is there any problem?

When I run the above program, I see the following:
```log
  0: , 1: 一, 2: 二, 3: 三, 4: 四, 5: 五, 6: 六, 7: 七, 8: 八, 9: 九: 10: 一零, 11：一一, 12: 一二, ..., 20: 二零, 

```

## 1.4 Special treatment of Zero

Obviously, there are 2 problems here: There is no result (an empty string) for 0, and starting from 10, it is not reasonable for (normal) numbers to just write the digits one behind the other.

We will start with the first problem.  The simplest solution is to have a special treatment in the beginning, maybe as follows:

```C++
  string zhCount(unsigned n)
  {
    if (n==0)
    {
      return zhDigits[0];
    }
    string result;
    ...
  }
```

This way, we get around the case 0.


## 1.5 Adding ten, hundred, thousand

Another problem was that 10 was printed as “一零” but should be something like “一十”, So whenever we are at the 10s digit, we need to append a "十".  For the 100s digit, we need to append "百", and for thousands, we need to append "千".  That is why I introduced the `zhTens`.  We can reach this as follows:

```C++
  string zhCount(unsigned n)
  {
    ...
    for (unsigned n= (int) digits.size()-1; pos>=0; pos--)
    {
      auto digit = digits[pos];
      result += zhDigits[digit] + zhTens[pos%4];
    }
    return result;
  }
```

The `%4` comes from the convention that Chinese numbers are named in groups of four, e.g. "1234'5678" becomes "一千二百三十四__五千六百七十八".  That is also, why I put the empty string `""` for the first `zhTens`.

Now, the output looks better, namely as follows:
```log
  0:零 , 1: 一, 2: 二, 3: 三, 4: 四, 5: 五, 6: 六, 7: 七, 8: 八, 9: 九: 10: 一十零, 11：一十一, 12: 一十二, ..., 20: 二十零, 
```

## 1.6 Omitting the zero digits

There is one problem left for the smaller numbers:  Nobody says "一十零", at most you would say "一十".  Therefore, we need to omit the 0s.  This can be reached as follows:

```C++
  string zhCount(unsigned n)
  {
    ...
    for (int pos=...; pos>=10; pos--)
    {
      auto digit = digits[pos];
      if (digit>0)
      {
        result += zhDigits[digit] +...;
      }
    }
    return result;
  }
```

## So how well does that work?

If you restart the program, you will see:

```C++
  0: 零, 1: 一, 2: 二, 3: 三, 4: 四, 5: 五, 6: 六, 7: 七, 8: 八, 9: 九: 10: 一十, 11：一十一, 12: 一十二, ..., 20: 二十, 
```

### But does it also work for bigger numbers?

Please do NOT increase the upper limit to `9999` with the small increment.  Instead, we modify the main program as follows:

```C++
  int main()
  {
    for (unsigned n=0; n<=20; n++)
    {
      cout<<n<<": "<<zhCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned n=30; n<=110; n+=10)
    {
      cout<<n<<": "<<zhCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned n=200; n<=1100; n+=100)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    cout<<endl;
    for (unsigned n=2000; n<=1'1000; n+=1000)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    cout<<endl;
    return 0;
  }
```

(Yes, you can but a separation mark `'` between groups of digits in C++.  If your compiler complains, this may be because the compiler is expecting the separation mark at the thousands, as it is done in Europe and America.)

The result looks almost as the example in the beginning.


## 1.7 tenthousand and hundred million

The previous program makes an error for 1'0000, namely it produces "一" and is omitting the "万".  Why did I not just put "万" behind "千" in the list of `zhTens`?

The answer lies at 10'0000, this is "一十万", not yet another number word.

When do we have to print "万"?

The answer is when `pos` is divisible by 4.  Therefore, we can modify the program as follows:

```C++
  string zhCount(unsigned n)
  {
    ...
    for (int pos=...; pos>=0; pos--)
    {
      auto digit = digits[pos];
      if (digit>0)
      {
        ...
      }
      if (pos%4 == 0)
      {
        result += zhTenthousands[pos/4];
      }
    }
    return result;
  }
```

## What is 亿?

Remember the sentence: 中国人口十四亿。

In other words, it is a quite big number.  To be more precise, when you multiply tenthousand by tenthousand (万万), then you get 亿.

## So, does that work in all cases?

I would say 99%, but to see where it fails, try the following:

```C++
  int main()
  {
    ...
    unsigned n = 1'0000'1000;
    cout<<n<<": "<<zhCount(n)<<endl;
  }
```

The last result will be something like "一亿万一千" which sounds strange.

Do you notice what went wrong?

The problem is that the character "万" is excessive (过度).  So how do we get rid of that?

We may only output the `tenThousands` if there is some value before it.  We can do that as follows:

```C++
  string zhCount(unsigned n)
  {
    ...
    bool value = false;
    for (pos...)
    {
      auto digit = digits[pos];
      if (digit>0)
      {
        ...
        value = true;
      }
      if (value && pos%4==0)
      {
        result += zhTenthousands[pos/4];
        value = false;
      }
    }
    return result;
  }
```

`bool` means a boolean value, i.e. either `true` (是/真) or `false` (否/不).  We need to remember whether there was a value before the tenthousands digit.  We will set the variable `value` to `true` if we printed some digit before the tenthousands.  Once, we have printed a tenthousands symbol, then we reset it to `value = false`.  The condition `value && pos%4==0` means that the computer tests for both conditions. First, it tests whether `value` is `true` and if that is given, it also tests for `pos%4==0`.  Therefore `&&` is read as "and also" (还有).


## 1.9 Careful testing

So does that work as expected?

Let us produce the results from the beginning.  Therefore we modify the main program as follows:

```C++
  int main()
  {
    ...
    for (unsigned n=2'0000; n<=11'0000; n+=1'0000)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    cout<<endl;
    for (unsigned n=20'0000; n<=110'0000; n+=10'0000)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    cout<<endl;
    for (unsigned n=200'0000; n<=1100'0000; n+=100'0000)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    cout<<endl;
    for (unsigned n=2'0000'0000; n<=1'1000'0000; n+=1'0000'0000)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    cout<<endl;
    for (unsigned n=20'0000'0000; n<=110'0000'0000; n+=10'0000'0000)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    cout<<endl;
    cout<<"无穷"<<endl;
    return 0;
  }
```

### What does "无穷" mean?

The English translation is infinity.  It means bigger than every number you can imagine.

So does the program work?  Is the result reasonable?

Give it a try.  What does your program produce?

Mine works mostly, but there is some strange behavior for the last line.  Namely behind 40'0000'0000 the program produces random numbers.  When you look carefully in your IDE (the dev++), then the number `110'0000'0000` or the comparison sign `<=` should be marked in yellow.  The problem is that even `unsigned int` has a maximal available number.  It is $2^{32}-1$ which is somewhat above `40'0000'0000`.  If we want to use higher numbers, then we need to modify our program, e.g. as follows:

```C++
  string zhCount(unsigned long n)
  {
    ...
  }

  int main()
  {
    ...
    for (unsigned long n=20'0000'0000; n<=110'0000'0000; n+=10'0000'0000)
    {
      cout<<n<<": "<<zhCount(n)<<", ";      
    }
    ...
  }
```

The good news is that the rest of the program does not depend on whether `n` is an `unsigned long` or an `unsigned int`.

So we are half through our project.



# 3. Chinese banking numbers

Instead of immediately proceeding to English number words, let us briefly talk about counterfait numbers (反法的数字).  A number e.g. on a bank transaction statement looks as follows.  Suppose you (as a rich billionary) want to transfer ¥1'2345'6789, then you need to write: "¥壹亿贰仟叁佰肆拾伍万陆仟柒佰捌拾玖人民币"  Ok, then ¥ and the 人民币 is because it is Chinese money.

## What is the reason not to write 一亿二千三百四十五万六千七百八十九？

Well, suppose you get such a cheque from your manager and you decide that you need double the amount.  What if you just added a dash (一) to the first digit?  After all, 二亿二千…… is almost double the amount, right?  Of course the bank and in particular your manager would not be amused.  But would the bank notice?

If you write digits as 一二三四五六七八九, then it is possible to fake some of them.  But if you use the above counter-fait digits, then it is nearly impossible.


## 3.1 Putting things in a function

```C++
  static vector<string> zhBankDigits = { "零", "壹", "贰", "叁", "肆",  "伍", "陆", "柒", "捌", "玖" };
  static vector<string> zhBankTens = { "", "拾", "佰", "仟" };
  static vector<string> zhBankTenthousands = { "", "万", "亿" };

  string zhBankCount(unsigned long n)
  {
    if (n==0)
    {
      return zhBankDigits[0];
    }
    string result = "";
    vector<unsigned_byte> digits;
    while (n>0)
    {
      digits.push_back(unsigned_byte(n%10));
      n /= 10;
    }
    for (int pos=(int) digits.size()-1; n>=0; n--)
    {
      auto digit = digits[pos];
      result += digit + zhBankTens[pos%4];
      if (pos%4 == 0)
      {
        result += zhBankTenthousands[pos/4];
      }
    }
    return result;
  }
```

Note that this time, we do not spare any digit.  This is the convention in filling a cheque and simplifies the parsing (and error correction) afterwards.


# 9. Time to Try for Yourself

So does your program work, i.e. does it compile and start?  Does it react as expected?  Does it terminate?


## 9.1 Polishing the grammar

I am not sure if I got all the Chinese numbering rules correctly.  It is also possible that there are some variations, e.g. 
101 can be written as 一百零一 or 1001 as 一千零一。 Maybe there are also other variants.

How would you implement such stuff?

What are the number signs beyond 万亿？ I heard that there is 兆, but are there further ones?
