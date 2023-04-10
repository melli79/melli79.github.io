After we have written numbers in Chinese and English last time, we want to represent people in the computer this time.


# 0. Our Goal

```log
  Mr. Melchior Grützmann (*1979-6-20)
    father D. Grützmann (*1954)
    mother B. Grützmann (*1955)
```

# 1. What makes a Person?

Well if I look at myself, I have a body, clothes, an apartment, a name, a day of birth, a father, a mother and many more things.  Which of those things do we want to model and represent in the computer?

I decided for the given names, family name, day of birth, father (when he is known), mother (when she is known) and some more.  So let us start with these:

```c++
  #include <iostream>
  #include <string>
  using namespace std;

  class Person {
    string familyName;
    vector<string> givenNames;
    const Gender gender;
    const PartialDate birthday;
    vector<string> titles;
  public:
    Person(string familyName0, vector<string> givenNames0, Gender gender0, PartialDate birthday0, vector<string> titles0 =vector<stirng>())
     :familyName(familyName0), givenNames(givenNames0), gender(gender0), birthday(birthday0), titles(titles0) {}
  };
```

Here we face a couple of new concepts.  First a `class` (类) means a definition of some family of objects that should all have certain properties (财产) and behave (行为) in a certain way.  Our class is called `Person`, because it represents a natual person.

Then we name a couple of properties, i.e. the features every person should have like a `familyName` (性), `givenNames` (名字), a `gender` (性别), a `birthday` (出生日) and maybe `titles` (称号).  Some of these properties are constant, i.e. marked with `const`, because they never change in the life of a person, e.g. the gender, the birthday, the given names.  Other properties can change, e.g. the family name (in the western world when 2 people marry, one of them can change their family name to represent the name of the other as well).

Then we write `Person()` again.  This is the constructor (构造函数).  It tells us how to make up a person.  In this case we do that by giving all the properties of a person.  These values are then handed to the respective property fields in the Person.

Don't forget the semicolon `;` after the class definition.

## 1.1 What Genders are there?

There are only finitely many Genders, so we can enlist or enumerate (枚举) them.  For that we use the C++ type `enum`.

```C++
  enum Gender {
  	Male=1, Female, Other
  };
```

## 1.2 What is a PartialDate

A full birthday consists of the year, the month and the day of the month.  But maybe for some people we only know (or want to present) parts of this date.  Therefore, we use the following version:

```C++
  typedef unsigned char  byte;

  class PartialDate {
  public:
    const short year;
    const byte month;
    const byte day;
    PartialDate(short year0, byte month0 =0, byte day0 =0) :year(year0), month(month0), day(day0) {}

    ...
  };
```

So clearly, a PartialDate contains the year, month and day.  We can construct a PartialDate from a year, month, day.  But it is also possible to omit the day which will then be assumed as 0.  It is also possible to omit the month (and day) which will then (both) be assumed as 0.

How do we print dates?

In the main program, we want to write something like:

```C++
  int main()
  {
    PartialDate today(2023,4,7);
    cout<<"Today is "<<today<<endl;

    return 0;
  }
```

Therefore, we need to implement some function as: `ostream& operator <<(ostream& out, const PartialDate& date);`  But we want this function to be related to the PartialDate.  Therefore, we declare it as `friend` as follows:

```C++
  class PartialDate
  {
    ...

    friend ostream& operatror <<(ostream& out, const PartialDate& d) {
      if (d.month==0)
      {
        return out<<d.year;
      }
      if (d.day==0)
      {
        return out<<d.year<<"/"<<d.month;
      }
      return out<<d.year<<"-"<<d.month<<"-"<<d.day;
    }
  };
```

We can access (访问) the properties of an object via `.`-notation, i.e. `p.year` gets the `year` of the object `p` if it is a PartialDate.

In other words, we only output as much as is known in the PartialDate.


## 1.3 How to print a Person?

We can do that similar to a PartialDate:

```C++
  class Person
  {
    ...
  public:
    ...
    friend ostream& operator <<(ostream& out, const Person& p)
    {
      return out<<p.titles<<p.givenNames<<p.familyName<<" (*"<<p.birthday<<")";
    }
  };
```

### Does that compile?

The answer is no, because the computer does not know how to print a `vector<string>`.  Therefore, we have to implement that before the class, e.g. as follows:

```C++
  ostream& operator <<(ostream& out, const vector<string>& names)
  {
    for (auto name : names)
    {
      out<<name<<" ";
    }
    return out;
  }

  class PartialDate {
  	...
  };

  class Person {
  	...
  };
```

Now it should compile, run and prints the Person, e.g. in the following main program:

```C++
  int main()
  {
    using operator""s;
    auto melli = Person("Grützmann"s, {"Melchior"s}, Male, PartialDate(1979,6,20));
    cout<<melli<<endl;

    return 0;
  }
```

## 1.4 What about the Gender and addressing?

Well, the addressing depends on the gender maybe as follows:

```C++
  class Person
  {
    ...
  public:
    ostream& printAddressing(ostream& out) const {
      switch(gender) {
        case Male: return out<<"Mr. ";
        case Female:  return out<<"Ms. ";
        default:  return out;
      }
    }
  };
```

We can use this method in the `friend operator <<(...)` as follows:

```C++
  friend ostream& operator <<(ostream& out, const Person& p) {
    return p.printAddressing(out)<<titles<<...;
  }
```

Now the result of the same last main program looks better, e.g.
```log
  Mr. Melchior Grützmann (*1979-6-20)
```

# 2. Adding Parents
How can I add my parents?

The naïve attempt would be something as the following:

```C++
  class Person
  {
    ...
    vector<Person> parents;
  public:
    Person(string familyName0, vector<string> givenNames0, ..., vector<Parents> parents0) :..., parents(parents0) {}
    ...
  }
```

But there are 2 problems with that:  First it does not compile.  The compiler complains with something like `... type Person is incomplete in instantiating constructor or vector<T>()`.  We could somehow work around that with a forward declaration.  But the second problem is that we do not wish to copy (拷贝) the parents, instead we want to reference (参考) the parents.

In modern C++ this is done via `shared_ptr` where "ptr" stands for pointer (指针).  It means that the child can point to their parents, which is reasonable when the child is old enough and the parents are around.

With that idea, the class definition should look something like the following:

```C++
  class Person
  {
    ...
    vector<shared_ptr<Person>> parents;
  public:
    Person(string name, ..., const vector<shared_ptr<Person>>& parents) :... {}
    ...
  };
```

But how do we use that?

We would have to make the `Person` in a shared pointer.  This is best done as follows:

```C++
  class Person;
  shared_ptr<Person> makePerson(string name, vector<string> givenNames, Gender gender, PartialDate birthday, vector<shared_ptr<Person>> parents);

  class Person
  {
    ...
    Person(string name0, ...) :name(name0), ..., parents(parents0) {}
  public:
    friend shared_ptr<Person> makePerson(string name, vector<string> givenNames, Gender gender, PartialDate birthday, vector<shared_ptr<Person>> parents) {
      return make_shared(Person(name, givenNames, ..., parents));
    }
    ...
  };
```

With that, we write our main program as follows:
```C++
  int main()
  {
    using operator""s;
    auto father = makePerson("Grützmann"s, {"D."s}, Male, PartialDate(1954));
    auto mother = makePerson("Grützmann"s, {"B."s}, Female, PartialDate(1955));
    auto melli = makePerson("Grützmann"s, { "Melchior"s }, Male, PartialDate(1979,6,20), { father, mother });
    cout<<melli<<endl;
    return 0;
  }
```

Here `auto` means that the compiler should choose automatically (自动地) the type of the variables `father`, `mother` and `melli`.

The problem is that this does not compile.  The compiler complains that there is ``no operator <<(ostream&, shared_ptr<Person>&)`` and that is true, we have not defined any.

For that, we split the definition of the friend `operator <<` as follows:

```C++
  class Person;
  ...
  {
  public:
    ostream& show(ostream& out) const {
      return printAddressing(out)<<titles<<givenNames<<familyName<<" (*"<<birthday<<")";
    }
  };

  ostream& operator <<(ostream& out, const shared_ptr<Person>& p) {
    return p->show(out);
  }
```

Now, the program should compile and start.  Unfortunately, it does not show the parents.

Instead of cramming this all in the method `show(...)`, we write a new method `describe(...)` (描述) that prints all details about the Person, maybe as follows:

```C++
  class Person;
  ...
  {
    ...
  public:
    ...

    ostream& describe(ostream& out) const
    {
      show(out);
      for (auto& parent : parents)
      {
        out<<endl<<"  "<<parent;
      }
      return out;
    }
  };
```

## 2.1 Printing Father and Mother

Currently my parents are just printed as Mr. and Ms., respectively.  If we want to change them to "father" and "mother", then we should mark them as `Parent` when showing them.  Therefore, we introduce another enum:

```C++
  enum Relation { // 关系
    Self=0, Parent
  };
```

and incorporate that into `printAdressing(...)`:
```C++
  class Person
  ...
  {
    ostream& show(ostream& out, Relation relation =Self) const {
      return printAddressing(out, relation)<<titles<<...;
    }

    ostream& printAddressing(ostream& out, Relation relation) const {
      switch (relation)
      {
        case Self:  switch (gender)
        {
          case Male:  return out<<"Mr. ";
          case Female:  return out<<"Ms. ";
          default:  return out;
        }
        case Parent:  switch (gender)
    	{
    	  case Female: return out<<"mother ";
    	  default:  return<<out<<"father ";
    	}
      }
      return out;
    }

    ostream& describe(ostream& out) const
    {
      show(out);
      for (auto& parent : parents)
      {
        parent->show(out, Parent);
      }
      return out;
    }
  };
```

Finally the main program is modified as follows:

```C++
  int main()
  {
    ...
    melli->describe(cout)<<endl;
    return 0;
  }
```

# 9. Time to try for yourself

Now is the latest time that you should try the examples for yourself.  Does the last program compile?  Does it start?  Does it produce the results you expected?

If you wish to try something else, you could e.g. add children to the Person, maybe with a field

```C++
  vector<shared_ptr<Person>> children;
```
 
But if you want to describe the children as well, you may have to add another case of `Relative`, e.g. `Child`.

Please let me know if you succeeded or if there are any questions.

## 9.1 Adding a spouse

Suppose I want to add a spouse (配偶) to the Person, how can I do that?

You may want to introduce another `shared_ptr<Person> spouse` 
