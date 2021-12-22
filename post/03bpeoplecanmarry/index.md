Heute wollen wir schauen, was Personen tun können.

# 3. Zwei Personen können heiraten.

Wenn wir schon ein Modell von Menschen haben, dann sollten die auch wie Menschen leben können.  Als erstes wollen sie heiraten können.

Was sind die Voraussetzungen zum Heiraten?  Man braucht einen Mann, eine Frau (oder 2 Männer oder 2 Frauen), die müssen beide ledig sein und alt genug und dann werden sie Mann und Frau (oder Mann und Mann oder Frau und Frau).

## 3.1 Woran erkennt man, dass jemand ledig ist?
Naja, ledig ist man, wenn man nicht verheiratet ist, also keine/n Ehepartner/in (engl. spouse) hat.

Dem Computer erklärt man das so

```Kotlin
  Person(...) {
    ...

    fun isSingle() = spouse==null
  }
```

Das bedeutet, dass `isSingle` eine Funktion ist und die wird berechnet als `spouse==null`, also dass man keine/n Ehepartner/in hat.

Moment, früher haben wir doch Funktionen so geschrieben:
```Kotlin
  fun isSingle(p :Person) :Boolean {
    return p.spouse == null
  }
```

Ja, es gibt 2 Unterschiede:  Zum einen schreiben wir die Person davor, also `melli.isSingle()`, deshalb ist unsere Funktion innerhalb der Person definiert und nimmt kein weiteres Argument entgegen.  Solche Funktionen heißen Methoden.

Zum anderen ist doch vom Ausdruck `spouse==null` klar, dass es ein `Boolean` ist und das wir den Wert zurückgeben wollen.  Zur Erinnerung `Boolean` heisst, dass es `true` (wahr) oder `false` (falsch) sein kann.  Das haben sich die Entwickler von Kotlin auch gedacht und deshalb diese verkürzte Variante der Definition von Funktionen (oder Methoden) erfunden.  Sehr praktisch, oder?


## 3.2 Wie kann man nun heiraten?

Also im wahren Leben ist das komplizierter, aber im Computer können wir uns das wie folgt vorstellen:

```Kotlin
  fun Person.marry(s :Person, weddingDay :PartialDate) {
    if (!isSingle())
      throw IllegalStateException("Du bist nicht ledig, damit kannst du nicht mehr heiraten.")
    if (!s.isSingle())
      throw IllegalStateException("Dein/e Partner/in ist nicht ledig.  Den/die kannst du nicht heiraten.")

    if (age(weddingDay)<18)
      throw IllegalStateException("Du bist zu jung zum Heiraten.")
    if(s.age(weddingDay)<18)
      throw IllegalStateException("Dein/e Partner/in ist zu jung zum Heiraten.")

    // und wenn dann alles passt
    this.spouse = s
    s.spouse = this
  }
```

Hier haben wir einen Kompromiss, die Funktion muss nicht in der Klasse `Person` definiert werden, aber sie operiert auf einer Person.  Diese Person steht dann implizit dahinter (z.B. bei `isSingle()` oder bei `age()`).  Wenn man die Person explizit braucht, dann kann man `this` schreiben (engl. für dieses).

Was bedeuten die vielen `IllegalStateException`s?  Naja, in diesem Fall kann man halt nicht einfach heiraten.

## 3.3 Was ist mit dem Familiennamen?

Ok, den kann man bei der Hochzeit festlegen, z.B. behält eine Person immer ihren Namen, die andere Person kann den Familiennamen entweder behalten oder den des/der Partner/in annehmen oder einen Doppelnamen bekommen.

```Kotlin
  enum class FamilyNameRule {
    KeepName, SpousesName, DoubleName
  }

  fun Person.marry(s :Person, weddingDay :PartialDate, familyNameRule :FamilyNameRule =FamilyNameRule.SpousesName) {
    ...

    when (familyNameRule) {
      FamilyNameRule.Partners -> s.familyName = this.familyName
      FamilyNameRule.DoubleName -> s.familyName += "-"+this.familyName
      else -> {}
    }
  }
```

`when` (wenn oder sobald) bedeutet so ähnlich wie `if`, dass es mehrere Fälle geben kann.  Die einzelnen Fälle werden dann mit `->` aufgelistet, also wenn `familyNameRule==FamilyRule.Partners`, dann `s.familyName = this.familyName`. Wenn `familyNameRule==FamilyNameRule.DoubleName`, dann ... .

## 3.4 Wie alt ist eine Person?

Das hängt vom Geburtstag und vom aktuellen Datum ab.  Wir können das als Methode zur Person hinzufügen.

```Kotlin
  Person(...) {
    ...

    fun age(today :PartialDate) = today.year - birthday.year
  }
```

Aber ist das korrekt?

Wenn die Person am 31.12.2010 geboren ist und heute der 1.1.2011 ist, dann ist die Person doch noch nicht 1 Jahr alt.  D.h. eigentlich müssen wir noch 1 Jahr abziehen, wenn heute (engl. today) vor dem Geburtsmonat ist.  Der 2. Versuch wäre:
```Kotlin
  fun age(today :PartialDate) = today.year - birthday.year -
    if (month!=null&&today.month!=null&&month<today.month) 1
    else 0
```

Ist das korrekt?

Tja, wenn die Person am 31.12.2021 geboren ist und heute der 1.12.2022, dann ist die Person immer noch nicht 1 Jahr alt.  Es gibt also noch einen weiteren Fall.  3. Versuch:
```Kotlin
  fun age(today :PartialDate) = today.year - year -
    if (month!=null&&today.month!=null)
      if (month<today.month ||
          month==today.month&&day!=null&&today.day!=null&&day<today.day)
        1
      else 0
    else 0
```

Zum Glück brauchen wir nur das Alter in Jahren.

Funktioniert das nun?

```Kotlin
  fun main() {
    val melli = Person("Melchior", "Grützmann", Gender.Male, PartialDate(1979, 6, 20))
    val fanli = Person("Fanli", "Lin", Gender.Female, PartialDate(1991, 1, 17))

    println("$melli heiratet $fanli:")
    fanli.marry(melli, PartialDate(2050,1,1), FamilyNameRule.DoubleName)

    println("$melli ist verheiratet mit $fanli.")
  }
```

```log
  Herr Melchior Grützmann (*20.6.1979) heiratet Frau Fanli Lin (*17.1.1991):
  Herr Melchior Grützmann-Lin (*20.6.1979) ist verheiratet mit Frau Fanli Lin (*17.1.1991).
```

## 4. Selbst experimentieren

Ok, nun ist Zeit, selber zu probieren:

1. Wie bekommst du heraus, wie alt Melchior am 30.6.2050 ist?

2. Kann jemand, der am 2.2.2010 geboren ist, jemanden, die am 1.3. 2011 geboren ist, am 1.1.2028 heiraten?

Überlege dir das erst einmal theoretisch und probiere es dann mit dem Programm aus.

3. Wie sieht es am 5.2.2018 aus?

4. Und am 1.1.2030?
