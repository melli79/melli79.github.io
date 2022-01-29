# 4. Frauen Gebären

Woher kommen die Kinder?

Das soll jetzt kein Vortrag über das Kinderkriegen werden, aber wenn unser Modell Menschen beschreibt, dann sollte es auch Kinder produzieren können.

Im Einfachsten Fall bekommt eine Frau ein Kind und ihr Ehepartner ist der Vater.  Die Signatur sieht etwa so aus:

```Kotlin
  fun Person.giveBirth(givenName :String, birthday :PartialDate, gender :Gender) :Person {
    ...
  }
```

Offenbar brauchen wir auch das Geschlecht und den Geburtstag des Kindes.  Das Ergebnis sollte ein Kind sein, also eine Person.

Was brauchen wir für das Kind?

Das gleiche wie für die meisten Personen: einen Familiennamen, einen Vornamen, ein Geburtsdatum, ein Geschlecht, Eltern (soweit bekannt).

Den Familiennamen können wir von der Mutter nehmen (man kann den ja hinterher auch nochmal ändern).

Was sollte man nun beim Gebären beachten?

```Kotlin
  fun Person.giveBirth(givenName :String, birthday :PartialDate, gender :Gender) :Person {
    if (age(birthday)<15)
      throw IllegalStateException("Das kann nicht sein, die Eltern sind zu jung.")

    val mother :Person;  val father :Person
    if (this.gender==Gender.Female) {
      mother = this
      father = spouse
    } else { // sollte eigentlich nicht vorkommen
      father = this
      mother = spouse
    }

    val child = Person(givenName, familyName, gender, birthday, father=father, mother=mother)
    children.add(child)
    if (spouse!=null)
      spouse!!.children.add(child)

    return child
  }
```

Damit Kinder keine Kinder bekommen, habe ich willkürlich eine Grenze von mindestens 15 Jahren gezogen.  Man kann die auch höher ansetzen.

Die Mutter muss weiblich sein, d.h. wenn die übergebene Person weiblich ist, dann ist das die Mutter, ansonsten versuchen wir die Partnerin.

Der Familienname wird von der Person übernommen, die gebärt.  Das muss nicht immer stimmen, aber man kann den Familiennamen auch nach der Geburt ändern.

`father=father` bedeutet, dass die Person in der Variable `father` als `Person.father` verwendet wird.  Damit müssen wir uns nicht merken, an welcher Stelle der Vater steht.  Bei `mother=mother` gilt das entsprechend.

## 4.1 Funktioniert das?

```Kotlin
  fun main() {
    val father = Person("Wolfgang Dieter", "Grützmann", PartialDate(1954), Gender.Male)
    val mother = Person("Barbara", "Kulisch", PartialDate(1955), Gender.Female)
    father.marry(mother, PartialDate(1978))
    println("$father heiratete $mother.")

    val melli = mother.giveBirth("Melchior", PartialDate(1979, 6, 20), Gender.Male)
    println("$melli war geboren.")
  }
```

Ausgabe:
```log
  Herr Wolfgang Dieter Grützmann (*1954) heiratet Frau Barbara Grützmann (*1955).
  Herr Melchior Grützmann (*20.6.1979) wurde geboren.
```

Also klappt es.


## 4.2 Was passiert, wenn die Frau nicht verheiratet ist?

Dann wird offenbar kein Vater zugewiesen.  Das ist aber kein Problem, da man den Vater auch noch später zuweisen kann, wenn er bekannt ist.

```Kotlin
  fun main() {
    val tanja = Person("Tanja", "Mayer", PartialDate(1978), Gender.Female)

    val tina = tanja.giveBirth("Tina", PartialDate(1985, 1, 1), Gender.Female)
    println("${tina}s Eltern sind ${tina.mother} und ${tina.father}.")
  }
```

```log
  Tina Mayer (*1.1.1985)s Eltern sind Tanja Mayer (*1978) und null.
```

Hmm, irgendwie müssten wir statt "null" eher "unbekannt" schreiben.


# 5. Wie kann man sich scheiden lassen?

Naja, das kommt nicht immerzu vor, muss aber im Programm auch eingeplant werden.

Zum Scheiden muss man offenbar verheiratet sein.  Eigentlich müssen die 2 Ehepartner die Scheidung beantragen, aber damit das Programm einfacher zu bedienen ist, reicht uns eine Person, die dann von ihrem/ihrer Ehepartner/in geschieden wird, etwa so:

```Kotlin
  fun Person.divorce() {
    if (spouse==null)
      throw IllegalStateException("Alleinstehende können nicht geschieden werden.")

    spouse!!.spouse = null
    this.spouse = null
  }
```

So einfach kann das im Computer gehen.

## Was bedeutet das `!!`?

Hmm, normalerweise kann der/die Partner/in `null` sein.  Dieser Fall ist aber oben bereits ausgeschlossen.  Trotzdem erkennt der Compiler nicht immer, dass das gegeben ist.  Deshalb kann man ihn mit `!!` überzeugen, dass das so ist.

## Was passiert, wenn ich den Compiler belüge?

Tja, dann mach er im guten Glauben, wovon du ihn überzeugt hast, und wahrscheinlich geht es irgendwann schief ("NullpointerException").


# 9.b Selber probieren

So jetzt bist du dran.  Versuch doch mal, deinen Familienstammbaum zu rekonstruieren.  Dazu fängst du am besten bei dem/der ältesten Verwandten an, den/die du kennst, z.B. deinem Opa.

```Kotlin
  fun main() {
    val grandpa = Person("Klaus Gottfried", "Grützmann", PartialDate(1926), Gender.Male)
    val grandma = Person("Susanne", "Friedemann", PartialDate(1929), Gender.Female)
    grandpa.marry(grandma, PartialDate(1953))

    val daddy = grandma.giveBirth("Wolfgang Dieter", PartialDate(1954), Gender.Male)
    val mommy = Person("Barbara", "Kulisch", PartialDate(1955), Gender.Female)
    daddy.marry(mommy, PartialDate(1978))

    val melli = mommy.giveBirth("Melchior", PartialDate(1979, 6, 20), Gender.Male)
  }
```

Wenn du Personen nicht kennst (z.B. die Großmutter deiner Mutter), dann kannst du sie vielleicht weglassen.  Wenn du nur den Vornamen nicht kennst, kannst du vielleicht die Verwandtschaftsbeziehung schreiben, z.B. `Person("Oma", "Haberda", ...)`.

Also, kennst du deine Verwandten?  Kannst du sie dem Computer erklären?


# 6. Verwandte höheren Grades

## 6.1 Warum haben wir die Geschwister nicht im Modell eingetragen?

Naja, weil das Modell dann noch unübersichtlicher wird.  Andererseits kann man die Geschwister auch rekonstruieren, wenn die Eltern bekannt sind:  Die Kinder, außer mir selbst, von meinen Eltern sind meine Geschwister.

```Kotlin
  fun Person.computeSiblings() :Set<Person> {
    val siblings = mutableSetOf<Person>()

    if (mother!=null)
      siblings.addAll(mother!!.children)

    if (father!=null)
      siblings.addAll(father!!.children)

    siblings.remove(this)
    return siblings
  }
```

### Was ist ein `Set`?

Englisch "Set" heisst Menge auf Deutsch.  Das benutzen wir, um jedes Geschwister nur einmal aufzuzählen.  Wenn nämlich die Eltern schon immer verheiratet waren, dann haben sie die gleichen Kinder.  Wenn wir also erst alle Kinder der Mutter und dann alle Kinder des Vaters hinzufügen, dann hätten wir jedes Kind doppelt.  Außerdem heißt `mutableSetOf`, dass wir die Menge auch verändern wollen, z.B. die verschiedenen Kinder hinzufügen.

Warum steht da `siblings.remove(this)`?

Naja, wenn wir `melli.siblings()` aufrufen, dann ist `this==melli`, d.h. erst fügen wir alle Kinder von Mutter und Vater hinzu, und dann entfernen wir `melli` wieder.  Klar, oder? (`melli` ist nicht sein eigenes Geschwister.)

# Was kommt denn in deinem Familienstammbaum heraus?

## 6.2 Warum stürzt das bei mir ab?

Oh, gut, dass du es ausprobiert hast.  Ich habe den Fehler gefunden:  Wenn man `Person`en in Mengen organisieren will, müssen sie ein paar wichtige Eigenschaften erfüllen:

```Kotlin
  class Person(...) {
    ...
    override fun equals(other :Any?) :Boolean {
      if (other !is Person)
        return false

      return givenName==other.givenName && gender==other.gender && birthday==other.birthday
    }

    override fun hashCode() = givenName.hashCode() +31*gender.hashCode() +997*birthday.hashCode()
  }
```

Also eine Person ist gleich einem anderen Objekt, wenn das andere Objekt eine Person ist und wenn Vorname, Geschlecht und Geburtstag übereinstimmen. (`&&` bedeutet 'und', also dass zum einen das Linke wahr sein muss und dann noch getestet wird, ob das Rechte wahr ist.)

### Warum vergleichen wir nicht den Familiennamen?

Weil sich der Familienname im Laufe des Lebens ändern kann.  (Das Geschlecht hoffentlich nicht.)

### Wieso `hashCode`?

Tja, die Implementation von `Set` muss jedes Objekt in einen HashCode umwandeln und sortiert dann zunächst diese ein.  Die sollten weitgehend, müssen aber nicht streng verschieden sein, z.B. gibt es mehr Vornamen als ganze Zahlen auf dem Computer.

### B) Es funktioniert aber immer noch nicht :-(

Dann sollten wir für den zweiten Typ `PartialDate` die gleichen Funktionen noch definieren, etwa so:
```Kotlin
  class PartialDate(...) {
    ...
    override fun hashCode() = year +(if (month!=null) 13*month else 0) +
            (if (day!=null) 13*37*day else 0)

    override fun equals(other :Any?) :Boolean {
        if (other !is PartialDate)
            return false
        return year==other.year && (month==null||other.month==null||month==other.month) &&
          (day==null||other.day==null||day==other.day)
    }
  }
```

### Was bedeuten die vielen Fälle bei `equals`?

Wenn alle Daten vollständig bekannt sind, dann wird einfach Jahr, Monat und Tag jeweils verglichen.  Wenn der Tag in einem Datum nicht definiert ist (also `null`), dann wird der ignoriert.  Wenn der Monat in einem Datum nicht definiert ist, dann wird auch der ignoriert. `||` bedeutet 'oder', d.h. es wird erst das Linke überprüft und wenn es wahr ist, dann ist der ganze Ausdruck wahr, ansonsten wir noch das Rechte überprüft und wenn das wahr ist, ist das ganze wahr, ansonsten falsch.


## 6.3 Wie kann man die Großeltern finden?

Also Großeltern sind die Eltern der Eltern, also muss man von der Person die Eltern finden (falls die bekannt sind) und dann deren Eltern sammeln.  Das geht etwa so:

```Kotlin
  fun Person.computeGrandparents() :Set<Person> {
    val grandparents = mutableSetOf<Person>()

        if (mother!=null) {
          if (mother!!.mother!=null)
            grandparents.add(mother!!.mother!!)
          if (mother!!.father!=null)
            grandparents.add(mother!!.father!!)
        }

    if (father!=null) {
      if (father!!.mother!=null)
        grandparents.add(father!!.mother!!)
      if (father!!.father!=null)
        grandparents.add(father!!.father!!)
    }

    return grandparents
  }
```

Und im Hauptprogramm schreibt man dann:

```Kotlin
  fun main() {
    ...
    val melli = mother.giveBirth("Melchior", PartialDate(1979, 6, 20), Gender.Male)

    val grandparents = melli.computeGrandparents()
    println("$mellis Großeltern sind $grandparents.")
  }
```

```log
  Herr Melchior Grützmann (*20.6.1979)'s Großeltern sind [Frau Irmtraud Kulisch (*1931), Frau Susanne Grützmann (*1929), Herr Klaus Gottfried Grützmann (*1926)].
```

## Wie sieht es mit deinen Großeltern aus?

Viel Spaß beim Probieren.


# Datenschutz

Wenn du das Programm nur für dich schreibst oder für dich und deine Familie, dann ist es im Prinzip egal, welche Daten du dort eintippst.  Aber bitte die Geburtsdaten deiner Eltern oder Großeltern nicht im Internet veröffentlichen.
