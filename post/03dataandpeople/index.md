Heute soll es darum gehen, wie man Leute im Computer darstellt.

# 1. Was ist uns an Leuten wichtig?

Ein echter Mensch ist 2--200 kg schwer, besteht aus Haut, Muskeln, Knochen, ..., trägt Kleidung, ... .
Die werden wahrscheinlich nicht in den Computer passen.

Stattdessen überlegen wir uns, was wir von einem Menschen darstellen wollen, d.h. wir bilden uns ein Modell.

Also ein Mensch hat

* einen festen Vornamen,
* einen Familiennamen,
* ein Geburtsdatum,
* ein Geschlecht,
* einen Vater, eine Mutter,
* vielleicht eine/n Ehepartner/in,
* vielleicht Kinder.

# 2. Wie können wir diese Daten darstellen?

Vor- und Familiennamen können wir als Text darstellen, in Kotlin `String`s.  Das Datum stellen wir als `PartialDate` dar, d.h. für manche Menschen kennen wir nur das Geburtsjahr, für andere das genaue Datum (mit Jahr).

Beim Geschlecht gibt es offiziell 3 Möglichkeiten: männlich (`Male`), weiblich (`Female`) und divers (`Other`), d.h. das Geschlecht (`Gender`) kann genau 1 dieser 3 Fälle sein.  Das kann man in Kotlin durch eine Klasse mit aufzählbaren Werten (`enum class`, engl. enumerable heisst aufzählbar) darstellen:

```kotlin
  enum class Gender(val title :String) {
    Male("Herr"), Female("Frau"), Other("")
  }
```

Dabei bedeutet `val title :String`, dass es zu jedem Geschlecht noch einen Titel gibt.  Der Titel des jeweiligen Geschlechts steht in Klammern dahiner, für divers ist mir nichts eingefallen.

Wie sieht nun eine Person aus?

## 1. Versuch
```Kotlin
  data class Person(
    val givenName :String,
    var familyName :String,
    val gender :Gender,
    val birthday :PartialDate,
    var father :Person,
    var mother :Person,
    var spouse :Person? =null,
    val children :MutableList<Person> =mutableList()
  )
```

Was bedeutet `var spouse :Person? =null`?  Zunächst einmal dass der/die Partner/in (engl. spouse) sich ändern kann (`var` steht für engl. variable, änderbar).  Dann, dass es eine `Person` sein kann oder auch niemand (`?`) und schließlich, dass der Anfangswert `null` ist (also niemand).  Das ist sicherlich sinnvoll, z.B. wenn die Person gerade erst geboren ist.

Was bedeutet `:MutableList<Person> =mutableList()`?  Das die Kinder in einer Liste eingetragen werden und die sich im Laufe des Lebens ändern kann (engl. mutable heisst änderbar).   Am Anfang ist die Liste leer, weil man noch keine Kinder hat.

Wie erzeugt man jetzt eine Person?

```Kotlin
  val melli = Person("Melchior", "Grützmann", Gender.Male, PartialDate(1979,6,20), ?, ?)
```

Irgend etwas stimmt da noch nicht.  Was gebe ich für Vater und Mutter an?  Ok, ich habe einen Vater und eine Mutter, aber wenn ich die nicht angeben will?  Selbst wenn ich die angeben würde, was gebe ich für den Vater meines Vaters an?  Und für dessen Mutter?  Und für deren Eltern??

Wenn ich nicht die gesamte Menschheitsgeschichte angeben möchte, dann sollten auch Vater und Mutter optional sein, also:

## 2. Versuch

```Kotlin
  class Person(...,
    var father :Person? =null,
    var mother :Person? =null,
    ...
  )
```

Jetzt kann ich schreiben:
```Kotlin
  val melli = Person("Melchior", "Grützmann", Gender.Male, PartialDate(1979, 6, 20))
```
Die restlichen Eigenschaften haben einfach ihre Standardwerte, also keine Eltern, keine Partnerin, keine Kinder.

## Wie sieht so eine Person aus?
Die Standardausgabe ist ziemlich komisch, da steht "Person@2354345".  Wir hätten doch lieber "Herr Melchior Grützmann (*20.6.1979)".

Also müssen wir Kotlin sagen, wie es die Person in einen Text (`String`) umwandelt.  Das geht so:

```Kotlin
  Person (...) {
    override fun toString() = """${gender.title} $givenName $familyName (*$birthday)"""
  }
```
"""...""" bedeutet dabei, dass der lange Text zwischen den 2x3 Anführungszeichen ausgewertet wird (die $... durch die Werte ersetzt werden).

`"${gender.title}"` bedeutet, dass der Titel zum Geschlecht ausgegeben wird, also für männlich (`Male`) "Herr", für weiblich (`Female`) "Frau" und für divers (`Other`) "" (ein leerer String).

Und wenn wir das Geburtsdatum implementiert haben, dann kann man eine Person auch wirklich ausgeben.  Das sieht dann so aus:

```log
  Herr Melchior Grützmann (*20.6.1979)
```

## 3. Wie stellt man ein teilweise bekanntes Datum dar?

Nachdem wir jetzt schon etwas Erfahrung mit Personen haben, machen wir das ähnlich:

```Kotlin
  data class PartialDate(val year :Int, val month :Byte? =null, val day :Byte? =null)
```

`Byte` ist eine kleine Zahl, nur bis maximal 127.  Das reicht für den Tag im Monat oder den Monat im Jahr.

Wir haben das gleiche Problem mit der Ausgabe.  Diesmal können wir das aber nicht mit nur 1 String beschreiben.  Stattdessen haben wir 3 Fälle:  ein ganzes Datum (mit Monat und Tag), ein Datum mit Monat (ohne Tag), eine Jahreszahl.  Das kann man so darstellen

```Kotlin
  override fun toString() = if (month!=null)
      if (day!=null) """$day.$month.$year"""
      else """$month/$year"""
    else """$year"""
```

Achtung, weil der Schrägstrich '/' innerhalb des Strings ist, wird hier nicht dividiert, sondern ein Schrägstrich '/' ausgegeben.

Das gesamte Programm sieht etwa so aus:

```Kotlin
  data class PartialDate(val year :Int, val month :Byte? =null, val day :Byte? =null) {
    override fun toString() = if (month!=null)
        if (day!=null) """$day.$month.$year"""
        else """$month/$year"""
      else """$year"""
  }

  data class Person(
    val givenName :String,
    var familyName :String,
    val gender :Gender,
    val birthday :PartialDate,
    var father :Person? =null,
    var mother :Person? =null,
    var spouse :Person? =null,
    val children :MutableList<Person> =mutableList()
  ) {
    override fun toString() = """${gender.title} $givenName $familyName (*$birthday)"""
  }

  fun main() {
    val melli = Person("Melchior", "Grützmann", Gender.Male, PartialDate(1979, 6, 20))

    println("Hallo $melli!")
  }
```

Und jetzt ist die Ausgabe wirklich:
```log
  Hallo Herr Melchior Grützmann (*20.6.1979)!
```

Im zweiten Teil modellieren wir, wie eine Person heiraten, Kinder bekommen und sich scheiden lassen kann.
