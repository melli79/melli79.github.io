After we have written numbers in Chinese last time, we want to write some numbers in English this time.

Haken vs Hacken
# 0. Our Goal

Here again the second sample:

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

Note that I put the separation mark `'` every 3 groups.  That is standard in Western numbers.


# 2. English numbers

Implementing the English number words is similar to implementing the Chinese number characters, but quite some things are different.


First, we reduce our main program a bit, e.g. as follows:

```C++
  int main()
  {
    for (unsigned n=0; n<=20; n++)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    return 0;
  }
```

So we need to write a function as follows:

```C++
  string enCount(unsigned n)
  {
     string result;
    ...
    return result;
  }
```


## 2.1 English numerals

Similar to the Chinese numerals (数词), we split the special words up as follows:

```C++
  static vector<string> enDigits = { "zero", "one", "two", "three", "four",  "five", "six", "seven", "eight", "nine",
         "ten", "eleven", "twelve", "thirteen", "fourteen", "fifteen", "sixteen", "seventeen", "eighteen", "nineteen" };
  static vector<string> enTens = { "", "ten", "twenty", "thirty", "fourty", "fifty", "sixty", "seventy", "eighty", "ninety", "hundred" };
  static vector<string> enThousands = { "", "thousand", " million ", " billion ", " trillion " };
```

As an initial step, we can just use the numerals upto 19 as follows:

```C++
  string enCount(unsinged n)
  {
    if (n<20)
    {
      return enDigits[(short)n];
    }
    string result;
    return result;
  }
```

The problem is that the numerals for 0..19 are all different.  But with the above snippet, we already handle those.

You can test that with the main program.


## 2.2  Upto 99

Here we split away the 10s and print 10s and 1s separately, maybe as follows:

```C++
  string enCount(unsinged n)
  {
    if (n==0)
    {
      return enDigits[0];
    }
    string result;
    vector<unsigned_byte> digits;
    while (n>0)
    {
      digits.push_back(unsigned_byte(n%10));
      n /= 10;
    }
    unsigned_byte offset = 0;
    for (int pos=(int) digits.size()-1; pos>=0; pos--)
    {
      auto digit = digits[pos];
      if (digit>0 || offset>0)
      {
        if (pos==1)
        {
          if (digit==1)
          {
            offset = 10;
          }else
          {
            result += enTens[digit];
          }
        }else
        {
          result += enDigits[digit+offset];
          offset = 0;
        }
      }
    }
    return result;
  }
```

This should give correct results upto 99 as we can check with the following main program:

```C++
  int main()
  {
    for (unsigned n=0; n<20; n++)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned n=20; n<30; n++)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned n=30; n<110; n+=10)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    return 0;
  }
```


## 2.3 Up to 999
So for the 100s, we have to add "hundred", i.e. `enTens[10]`.

```C++
  string enCount(unsinged n)
  {
    ...
    for (int pos=...)
    {
      auto digit = digits[pos];
      if (digit>0 || offset>0)
      {
        if (pos==2)
        {
          result += enDigits[digit] + enTens[10];
        }else if (pos==1)
        {
          ...
        }else
        {
          ...
          offset = 0;
        }
      }
    }
    return result;
  }
```

In order to test that, we can modify the main program as follows:

```C++
  int main()
  {
    ...
    cout<<101<<": "<<enCount(101)<<", ";
    for (unsigned n=200; n<1100; n+=100)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    return 0;
  }
```


## 2.4 The thousands, millions and more

This is similar to "万" in Chinese, i.e. we add it if we have value before it and only at positions divisible by 3.

```C++
  string enCount(unsinged n)
  {
    ...
    bool value = false;
    for (int pos=...)
    {
      auto digit = ...;
      if (digit>0 || offset>0)
      {
        ...
        value = true;
      }
      if (value && pos%3==0)
      {
        result += enThousands[pos/3];
        value = false;
      }
    }
    return result;
  }
```

And in order to test that:

```C++
  int main()
  {
    ...
    for (unsigned n=2'000; n<=11'000; n+=1'000)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned n=20'000; n<=110'000; n+=10'000)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned n=200'000; n<=1'100'000; n+=100'000)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned n=2'000'000; n<=11'000'000; n+=1'000'000)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    for (unsigned long n=2'000'000'000; n<=11'000'000'000; n+=1'000'000'000)
    {
      cout<<n<<": "<<enCount(n)<<", ";
    }
    cout<<endl;
    return 0;
  }
```

In the latter case, we also need to adapt the function to receive `unsigned long`:

```C++
  string enCount(unsigned long n)
  {
    ...
  }
```

And now it should work as expected.

## 2.8 Adding "and"

Sometimes you hear rules, such as $101$ should be called "hundredandone".  How can we implement that?

One possibility is that we may need "and" before printing a digit.  We encode that with `bool needsAnd` which starts at `false`.  Whenever we pass some digit that is 0, we need to set `needsAnd=true`.  In particular $1001$ is "onethousand and one", not ~~"onethousandandandone"~~.

```C++
  string enCount(unsinged long n)
  {
    ...
    bool needsAnd = false;
    for (int pos=...)
    {
      auto digit = digits[pos];
      if (digit>0 || offset>0)
      {
        if (needsAnd)
        {
          result += " and ";
        }
        if (pos==1)
        {
        ...
        }
        needsAnd = false;
      } else if (pos%3==2 || pos%3==1)
      {
        needsAnd = true;
      }
      if (pos%3==0)
      {
        if (value)
        {
      	  result += ...
          hasValue = false;
        }
        needsAnd = false;
      }
    }
    return result;
  }
```


# 3. Paper Cheques
These also exist(ed) in the USA/Europe.  The way they were protected against counterfeit was by writing the numbers twice, once in the position system and once in text form, e.g. in order to transfer $\\$123'456'789$ you first wrote these digits and in the line below, you wrote "onehundredtwentythree million fourhundredfiftysix thousand sevenhundredeightynine".  For those people who could not properly spell the millions and thousands it was also Ok, to just write the digits as words, i.e. "ONE-TWO-THREE FOUR-FIVE-SIX SEVEN-EIGHT-NINE" (small letters or capital letters).

I am not sure about the USA, but in Europe these times are over, i.e. if you have to make a transfer, you do that via online banking where you type the numbers and they are much harder to fake.


# 9. Time to try for yourself

Ok, I hope you tried the program(s) whenever we had implemented a step.  If not, now it is high time to try some stuff.


## 9.2 German numbers

This is a real nightmare for most programmers, they are written as follows:
```log
  0:null, 1:eins, 2:zwei, 3:drei, 4:vier, 5:fünf, 6:sechs, 7:sieben, 8:acht, 9:neun,
  10:zehn, 11:elf, 12:zwölf, 13:dreizehn, 14:vierzehn, 15:fünfzehn, 16:sechzehn, 17:siebzehn, 18:achtzehn, 19:neunzehn,
  20:zwanzig, 21: einundzwanzig, 22:zweiundzwanzig, ...
  30:dreißig, 31: einundreißig, ...
  40: vierzig, 50: fünftzig, 60: sechzig, 70: siebzig, 80: achtzig, 90: neunzig,
  100: (ein)hundert, 200: zweihundert, ...,
  1000: eintausend, 1'001: eintausend eins, 2000: zweitausend, 3000: dreitausend, ...
  10'000: zehntausend, 10'001: zehntausend eins, 11'000:elftausend, ..., 20'000: zwanzigtausend, 21'000: einundzwanzigtausend, ...,
  100'000: einhunderttausend, 200'000: zweihunderttausend, ...
  1'000'000: eine Million, 1'001'001: eine Million eintausend eins, 2'000'000: zwei Millionen, 3'000'000: drei Millionen, ...
  10'000'000: zehn Millionen, 11'000'000: elf Millionen, ..., 20'000'000: zwanzig Millionen, ...,
  100'000'000: einhundert Millionen, ...
  1'000'000'000: eine Milliarde, 1'001'000'000: eine Milliarde eine Million, 2'000'000'000: zwei Milliarden, 3'000'000'000: drei Milliarden, ...
  1'000'000'000'000: eine Billion, 2'000'000'000'000: zwei Billionen, ...
  unendlich.
```

But French numbers are equally bad.
