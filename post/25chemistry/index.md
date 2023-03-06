Nachdem wir letztes Mal über störungssichere Übertragung gesprochen haben, wollen wir heute systematische (anorganische) Chemie betreiben.

# 0. Unser Ziel

Wir haben 2 Teilziele.  Zum einen wollen wir die (meisten) Elemente des Periodensystems systematisch erfassen, also die hier:

![Periodensystem der Elemente](/img/PeriodicTable-full.png)
Bild 1: Periodensystem der Elemente. CC Antonsusi 2019.

Und zum anderen wollen wir die Summenformeln von (einfachen) anorganischen Verbindungen ausrechnen, also so etwas wie NaCl oder Al$_2$O$_3$.


# 1. chemische Elemente darstellen

Wie man aus den Farben des obigen Periodensystems erkennt, kommen die Elemente in ca. 4 Klassen: Metalle, Halbmetalle, Nichtmetalle, Edelgase.  Da es nur ca. 118 Elemente gibt und keine neuen einfach erzeugt werden können, wollen wir die Elemente als Enums darstellen.  Entsprechend der 4 Klassen wollen wir 4 verschiedene Enums anlegen.  Trotzdem sollen alle gleichwertig als Atome erkennbar sein.

Als ersten Ansatz stellen wir die Atome durch ein Interface dar.  Jede konkrete Klasse (auch Enums) kann dieses Interface implementieren und die Bindungsfunktionen können dann ein `Atom` entgegen nehmen.  Vielleicht etwa so:

```Kotlin
  interface Atom {
  	val name :String
  	val valences :ByteArray
  	val oder :UByte
  	val symbol :String
  }
```

Wie kann man nun die konkreten Elemente eintragen?  Etwa so:
```Kotlin
  enum class NobleGas(override val order :UByte, symb :String? =null, override val valences :ByteArray =byteArrayOf()) :Atom {
  	Helium(2u), Neon(10u), Argon(18u), Crypton(36u, "Kr"), Xenon(54u), Radon(86u, "Rn");

  	override val symbol = symb ?: name.substring(0..1)
  }
```

Die Edelgase haben einige Gemeinsamkeiten:

* keine Valenzen,
* Symbol meist aus den ersten 2 Buchstaben.

Deshalb habe ich die im Konstruktor bereits eingetragen und die Default-Argumente nach hinten verschoben (sodass man die notwendigen Argumente am Anfang ohne Namen angeben kann).

Ganz ähnlich können wir mit den Nichtmetallen verfahren:

```Kotlin
  enum class Nonmetal(override val valences :ByteArray, override val order :UByte, symb :String? =null) :Atom {
  	Hydrogen(byteArrayOf(1), 1u),
  	Fluorine(byteArrayOf(-1), 9u), Chlorine(byteArrayOf(-1, 1,3,5,7), 17u, "Cl"), Bromine(byteArraysOf(-1, 1,3,5,7), 35u, "Br"), Iodine(byteArrayOf(-1, 1,3,5,7), 53u),
  	Oxygen(byteArrayOf(-2), 8u), Sulfur(byteArrayOf(-2, 4,6), 16u), Selenium(byteArrayOf(-2, 4,6), 34u, "Se"),
  	Nitrogen(byteArrayOf(-3, 5), 7u), Phosphorus(byteArrayOf(-3, 5), 15u),
  	Carbon(byteArrayOf(2, 4), 6u);

  	override val symbol = symb ?: name.substring(0..0)
  }
```
Hier liegen verschiedene Valenzen vor, sodass wir die nicht mehr standardmäßig initialisieren können.  Viele Symbole leiten sich aber aus dem ersten Buchstaben ab, sodass ich das Symbol unten berechnet habe und nur bei Chlor, Brom und Selen überschieben habe (dort besteht es aus 2 Buchstaben).

Ich denke, dass das Prinzip jetzt klar geworden ist.  Hier noch die Halbleiter/Halbmetalle:

```Kotlin
  enum class Semimetal(override val valences :ByteArray, override val order :UByte, symb :String? =null) :Atom {
  	Boron(byteArrayOf(3), 5u, "B"),
  	Silicon(byteArrayOf(4, -4), 14u), Germanium(byteArrayOf(4, -4), 32u),
  	Arsen(byteArrayOf(-3, 5), 33u, "As"), Antimony(byteArrayOf(-3, 5), 51u, "Sb"),
  	Tellurium(byteArrayOf(-2, 6), 52u), Polonium(byteArrayOf(-2, 6), 84u),
  	Astatine(byteArrayOf(7, -1), 85u, "At");

  	override val symbol = symb ?: name.substring(0..1)
  }
 ```

Am Aufwändigsten ist die Klasse der Metalle, weil die meisten chemischen Elemente Metalle sind.  Ich gebe die hier nur ausschnittsweise an, um auf ein paar Besonderheiten hinzuweisen:

```Kotlin
  enum class Metal(override val valences :ByteArray, override val order :UByte, symb :String? =null) :Atom {
  	Lithium(byteArrayOf(1), 3u), Sodium(byteArrayOf(1), 11u, "Na"), Potassium(byteArrayOf(1), 19u, "K"), ...
  	Beryllium(byteArrayOf(2), 4u), Magnesium(byteArrayOf(2), 12u, "Mg"), Calcium(byteArrayOf(2), 20u), ...
  	Scandium(byteArrayOf(1), 21u), ..., Lathanium(byteArrayOf(1), 57u), ...
  	Titanium(byteArrayOf(2,4), 22u), ...
  	...
  	Copper(byteArrayOf(1,2), 29u, "Cu"), Silver(byteArrayOf(1), 47u, "Ag"), Gold(byteArrayOf(1,2), 79u, "Au"),
  	...
  	Aluminium(byteArrayOf(3), 13u), ...
  	Stannium(byteArrayOf(2,4), 50u, "Sn"), Lead(byteArrayOf(2,4), 82u, "Pb"),
  	Bismuth(byteArrayOf(3,5), 83u);

  	override val symbol = symb ?: name.substring(0..1)
  }
```
Die restlichen Daten kann man der Wikipedia entnehmen.

Um die Daten zu testen, schreiben wir ein kleines Hauptprogramm:

```Kotlin
  fun main() {
    val elements = (Metal.values().toList<Atom>()+Semimetal.values()+NonMetal.values()+NobleGas.values())
        .orderedBy { it.order }
    var period = 0.toUByte()
    for (element in elements) {
      if (period<element.period) {
  	    print("\n$peridon. Periode:  ")
        period = element.period
      }
      print(element.describe()+", ")
    }
    println()
  }
```

Allerdings brauchen wir noch 2 Methoden für Atome:
```Kotlin
  interface Atom {
    ...
    val period :UByte
      get() = when(order.toInt()) {
        in 1..2 -> 1u
        in 3..10 -> 2u
        in 11..18 -> 3u
        in 19..36 -> 4u
        in 37..54 -> 5u
        in 55..86 -> 6u
        in 87..118 -> 7u
        else -> 8u
    }

    fun describe() = """${symbol}_$order"""
  }
```

Jetzt sollte die Ausgabe des Testprogramms etwa wie folgt aussehen:
```log
  1. Periode:  H_1, He_2,
  2. Periode:  Li_3, Be_4, B_5, C_6, N_7, O_8, F_9, Ne_10,
  3. Periode:  Na_11, Mg_12, Al_13, Si_14, P_15, S_16, Cl_17, Ar_18,
  4. Periode:  K_19, Ca_20, Sc_21, ..., Kr_36,
  5. Periode:  Rb_37, ..., Xe_54,
  6. Periode:  Cs_55, Ba_56, La_57, ..., Rn_86,
  7. Periode:  Fr_87, Ra_88, Ac_89, ..., Bh_107,
```

# 2. 2-Elementige Verbindungen
Dazu führen wir die Klasse `data class Bond2(val element1 :Atom, val element2 :Atom)` ein.  Aber offenbar brauchen wir noch 2 weitere Informationen:  Wieviele Atomrümpfe der jeweiligen Art machen ein neutrales Gebilde aus?  Also

```Kotlin
  data class Bond2(val element1 :Atom, val element2 :Atom, val amount1 :UByte, val amount2 :UByte) {}
```

Um eine Verbindung zu erzeugen, können wir entweder die Atomrümpfe und Stöchiometrien selbst angeben oder wir schreiben uns eine Funktion, die die ausrechnet.  Letzteres kann man mit den gegebenen Daten wie folgt tun:

```Kotlin
  fun bind(element1 :Atom, element2 :Atom) :Set<Bond2> {
    val result = mutableSetOf<Bond2>()
    if (element2==Nonmetal.Oxygen) when (element1) {
      Nonmetal.Hydrogen -> result.add(Bond2(element1, element2, 1u,1u))
      Nonmetal.Nitrogen -> {
         result.add(Bond2(element1, element2, 2u,1u))
         result.add(Bond2(element1, element2, 1u,1u))
         result.add(Bond2(element1, element2, 1u,2u))
      }
      else -> {}
    }
    for (valence1 in element1.valences) {
      for (valence2 in element2.valences) {
      	if (valence1*valence2<0) {
      	  val m = lcm(abs(valence1.toLong()), abs(valence2.toLong()))
          result.add(Bond2(element1, element2, (m/abs(valence1)).toUByte(), (m/abs(valence2)).toUByte()))
        }
      }
    }
    return result
  }
```

Allerdings müssen wir noch 3 Funktionen implementieren:

```Kotlin
  fun abs(x :Long) = when {
  	x>=0L -> x.toULong()
  	else -> (-x).toULong()
  }

  fun gcd(a :ULong, b :ULong) :ULong {
  	var a0 = a
  	var a1 = b
  	while (a1>0) {
  		val a2 = a0%a1
  		a0 = a1;  a1 = a2
  	}
  	return a0
  }

  val lcm(a :ULong, b :ULong) = a/gcd(a,b) * b
```
Was bedeuten die 3 Funktionen?

Der Betrag heißt im Englischen absolute value, abgekürzt mit `abs`.  Aus der Definition sieht man, dass der Betrag einer großen ganzen Zahl eine nicht-negative große ganze Zahl ist.

`gcd` steht für größter gemeinsamer Teiler (engl. greatest common divisor) und den brauchen wir, um `lcm` zu berechnen.

## 2.2 Was ist nun `lcm` und was hat das mit Stöchiometrie zu tun?

Betrachten wir 2 Elemente wie etwa Aluminium (Al) und Sauerstoff (O). Ersteres hat die Valenz +3 und letzteres die Valenz -2.  Wie müssen die also zusammenpassen?

1. eine Valenz muss negativ sein, die andere positiv (also das Produkt negativ);
2. wir müssen das kleinste gemeinsame Vielfache bilden, englisch least common multiple, und das macht die Funktion `lcm`;
3. die Stoffmengen ergeben sich aus diesem `lcm` geteilt durch den Betrag der Valenz.

## 2.3 Funktioniert das alles?

Um das zu testen, können wir entweder wieder ein Hauptprogramm schreiben oder eine Testbibliothek verwenden.  Der Vorteil des letzteren ist, dass wir Zusicherungen (engl. `assert`ions) überprüfen können und die Bibliothek sagt uns dann, wo etwas schiefgegangen ist.

Dazu müssten wir ggf. die Kompiler-Konfiguration anpassen, also die Datei "build.gradle.kts" (im Hauptverzeichnis des Projektes).  Der Abschnitt `dependencies` sollte folgende 3 Zeilen enthalten:
```Kotlin
  dependencies {
    ...
    testImplementation(kotlin("test"))
    testImplementation("org.junit.jupiter:junit-jupiter-api:5.9.0")
    testRuntimeOnly("org.junit.platform:junit-platform-engine:1.9.0")
  }

  tasks.test {
    useJUnitPlatform()
  }
```
Sowie auch ein zusätzlicher Abschnitt `tasks.test` mit diesem Eintrag vorhanden sein.  Falls du etwas geändert hast, erscheint oben rechts ein kleines Symbol mit einem grauen Elefanten und einem blauen Kreisel.  Dieses Symbol musst du klicken, damit die veränderte Konfiguration importiert wird.  Falls das Symbol mal nicht auftauchen sollte, kannst du auch rechts den Reiter "Gradle" (ebenfalls mit einem grauen Elefanten) öffnen und dort oben links im Unterfenster auf den grauen Kreisel (2 Pfeile, die im Kreis zeigen) klicken.  Danach kannst du das Gradle-Unterfenster und auch den Editor für die "build.gradle.kts" wieder schließen.

Es sollte links im Projektbaum unter Quellen "src" ein zweites Verzeichnis "test/kotlin" geben.  In diesem Verzeichnis legst du jetzt eine neue Kotlin Klasse "Bond2Tester" an:  Dazu im Projektbaum (links) den Reiter "test" aufklappen und dort auf "kotlin" mit der rechten Maustaste klicken.  Im Menü sollte der Punkt "Neu..." und darunter der Punkt "Kotlin Klasse/Datei" erscheinen, den du auswählst.  Danach sollte ein fliegendes Eingabefenster aufgehen, in dem du oben den Namen "Bond2Tester" eingibst.  Unten sollte bereits "Klasse" ausgewählt sein, sodass du einfach \<Enter\> drücken kannst.

Danach sollte sich ein neues Unterfenster im Editor öffnen, in dem eine Klasse `class Bond2Tester {}` angelegt ist.  Über dieser Klasse fügst du einen einzelnen Import ein: `import kotlin.test.*` (auf einer eigenen Zeile, ggf dahinter noch 1 Zeile freilassen).

Jetzt sind wir bereit, den ersten Test anzulegen:

In der Klasse `Bond2Tester` schreibst du auf eine neue Zeile `@Te`, dann sollte das Vorschlagsmenü aufgehen und die Zeile `@Test fun ` anbieten, damit geht es weiter.  Der Test sieht also wie folgt aus:

```Kotlin
  class Bond2Tester {
    @Test fun metalOxides() {
      for (metal in Metal.values()) {
      	val bonds = bind(metal, Nonmetal.Oxygen)
      	println("${metal.name}: $bonds")
      }
    }
  }
 ```
Um zu sehen, was der Test produziert, kannst du links im Editor auf den Doppelpfeil vor der Klasse rechts-klicken und aus dem Menü "Run 'Bond2Tester'" auswählen.  Danach sollte sich unter dem Editor ein Unterfenster öffnen, in dem du nach kurzer Übersetzungszeit rechts nur "BUILD SUCCESSFUL in ...s" und links einen grünen Haken mit "Test Results" siehst.  Wenn du links auf das Dreieck klickst, dann öffnet sich ein Unterzweig, in dem du die Klasse "Bond2Tester" sehen solltest (auch mit einem grünen Haken davor).  Auch hier auf das graue Dreieck klicken.  Dann sollest du einen dritten Haken mit dem Namen "metalOxides" sehen.  Wenn du auf diesen Namen klickst, kannst du im rechten Unterfenster die Metalloxide sehen:  Die Metalle sind der Reihe nach aufgelistet, jeweils vorn der Name und dahinter eine _Liste_ (eigentlich eine Menge), in der aber nur kryptische Namen "Bond2@2f23ab" stehen.  Um hier die Summenformel auszugeben, müssen wir noch die Klasse `Bond2` mit einer sinnvollen `fun toString()` versehen.  Dazu fügst du in die Klasse folgende Methode ein:

```Kotlin
  data class Bond2(...) {
  	override fun toString() = "${element1}_$amount1 ${element2}_$amount2"
  }
```
Um den Test erneut zu starten, musst du einfach auf den grünen Pfeil oberhalb des Editors klicken.  Jetzt sollten schon viele sinnvolle Summenformeln dastehen, z.B.
```log
  Lithium: [Lithium_2 Oxygen_1],
  ...
  Beryllium: [Beryllium_1 Oxygen_1],
  ...
  Aluminium: [Aluminium_2 Oxygen_3],
  ...
```

Falls dich die 1en auch stören, müssten wir die `toString()`-Methode wie folgt anpassen:

```Kotlin
  data class Bond2(...) {
    override fun toString() = """$element1${amount(amount1)}$element2${amount(amount2)}"""

  }

  internal fun amount(n :UByte) = if (n==1u) ""  else "_$n "
```

D.h. dass die Konvention unter Chemikern ist, wenn die Stoffmenge 1 ist, dann wird die weggelassen.

Außerdem verwendet man in der Summenformel üblicher Weise die Kurzsymbole.  Dazu passen wir die `toString()`-Methode der Enums an:
```Kotlin
  enum class Metal(...) {
    ...

    override val symbol = symb ?: name.substring(0..1)
    override fun toString() = symbol
  }

  enum class Semimetal(...) :Atom {
    ...

     override fun toString() = symbol
  }

  enum class Nonmetal(...) :Atom {
    ...

    override fun toString() = symbol
  }

  enum class NobleGas(...) :Atom {
    ...

    override fun toString() = symbol
  }
```

Jetzt kannst du erneut testen.


## 2.4 Nichtmetall-Oxide

Auch die kann unser Programm bereits.  Dazu musst du einfach eine 2. Testfunktion schreiben, wie folgt:
```Kotlin
  class Bond2Tester {
  	...

    @Test fun nonmetalOxides() {
      for (element in Nonmetal.values()) {
        val bonds = bind(element, Nonmetal.Oxygen)
        println("${element.name}:  $bonds")
      }
    }
  }
```

## 2.5 Keine Verbindungen mit Edelgasen

Auch das sollte unser Programm beherrschen.  Dazu schreiben wir folgende Testfunktion:

```Kotlin
  class Bond2Tester {
    ...

    @Test fun nobleGases() {
      for (nobleGas in NobleGas.values()) {
        val bonds = NonMetal.values().flatMap { element ->
           bind(nobleGas, element)
        }
        println("${nobleGas.name}: $bonds")
        assertEquals(emptySet(), bonds)
      }
    }
  }
```
Hier habe ich jetzt etwas Neues eingebaut.  Wir wollen nicht nur einfach die Ergebnisse anzeigen, sondern wir wollen auch sicherstellen, dass es da keine Verbindungen gibt.

Deshalb haben wir die Zeile `assertEquals(expected, actual)` hinzugefügt, wobei wir `emptySet()` erwarten (engl. `expected`) und `bonds` als aktuellen Wert (engl. `actual`) übergeben.

Wenn du das Programm ausführst, sollten inzwischen 3 grüne Haken unter der Klasse `Bond2Tester` erscheinen.

Wenn du sicherstellen willst, dass das auch wirklich testet, kannst du die Spezifikation für Helium abändern, etwa wie folgt:

```Kotlin
  enum class NobleGas(...) :Atom {
  	Helium(2u, valences= byteArrayOf(2)), ...
  }
```

Wenn du jetzt die Tests wieder ausführst, solltest du statt einem grünen Haken ein gelbes Kreuz sehen.  (Falls du sogar ein Ausrufezeichen in einem roten Kreis siehst, ist im Programm etwas gewaltig schiefgegangen.)  Wenn du im Testfenster links auf den fehlgeschlagenen Test klickst, solltest du rechts eine Beschreibung des Fehlers sehen, leider in Englisch.  Bei mir sieht das etwa so aus:

```log
  Helium:  [HeO]

  expected: <[]> but was: <[HeO]>
  Expected : []
  Actual   : [HeO]
  <Click to see difference>

  org.opentest4k.AssertionFailedError: expected: <[]> but was: <[HeO]>
  ...
  at app//BondTester.nobleGases(BondTester.kt:20)
  ...
```

Die erste Zeile kommt vom erfolgreichen Teil des Tests, d.h. er konnte irgendwelche Verbindungen bestimmen und hat diese ausgegeben (zusammen mit dem Namen Helium).

Dann steht da `expected: ...`  `but was: ...`  Das bedeutet, dass etwas nicht so war, wie es erwartet wurde.  Wenn du in der Fehlermeldung weiter unten suchst, dann gibt es eine Zeile, in der etwas blau markiert ist "Bond2Tester.kt:20".  Wenn du auf diesen blauen Text klickst, kommst du direkt zu der Stelle, an der etwas schiefging.  Das sollte die Zeile mit dem `assertEquals` sein.  Am Anfang der Fehlermeldung steht auch, was schiefgegangen ist, nämlich, dass keine Verbindungen erwartet wurden, es aber welche gab.

Wenn du nun die Spezifikation von Helium wieder zurück änderst und den Test erneut laufen lässt, dann sollten wieder alle Tests grün sein.


## 2.6 Oxide
Hier kann man nach Metallen, Halbmetallen und Nichtmetallen unterscheiden.  Entsprechend schreiben wir 3 Tests:

```Kotlin
  class Bond2Tester {
    ...

    @Test fun metalOxides() {
      for (metal in Metal.values()) {
        println(metal.name+": ${bind(metal, Nonmetal.Oxygen)}")
      }
    }

    @Test fun nonmetalOxides() {
      for (element in Nonmetal.values()) if (element!=Nonmetal.Oxygen) {
        println(element.name +": ${bind(element, Nonmetal.Oxygen)}")
      }
    }

    @Test fun semimetalOxides() {
      for (element in Semimetal.values()) {
        println(element.name +": ${bind(element, Nonmetal.Oxygen)}")
      }
    }
  }
```


## 2.7 Salze

Auch die kann unsere Methode bereits.  Dazu müssen wir nur noch auswählen, welche Elemente sich mit Metallen zu Salzen verbinden können:

```Kotlin
  class Bond2Tester {
    ...

    val halogens = listOf(Nonmetal.Fluorine, Nonmetal.Chlorine, Nonmetal.Bromine, Nonmetal.Iodine, Nonmetal.Nitrogen, Nonmetal.Sulfur)

    @Test fun salts() {
      for (metal in Metal.values()) {
        print("${metal.name}:  ")
        for (halogene in halogenes) {
          print(bind(metal, halogene)+", ")
        }
        println()
      }
    }
  }
```

Das sind ziemlich viele, und die können anhand der Valenzen alle stöchiometrisch korrekt gebildet werden.


## 2.8 Halbmetall-Verbindungen

Etwas schwieriger sieht es bei den Halbmetall-Verbindungen aus.  Das Programm kann nicht erkennen, wenn es Verbindungen, die stöchiometrisch funktionieren, aus Stabilitätsgründen gar nicht gibt.  Deshalb schränken wir den Test ein:

```Kotlin
  class Bond2Tests {
    ...

    val weakMetals = listOf(Metal.Sodium, Metal.Potassium, Metal.Rubidium, Metal.Caesium, Metal.Franzium, Metal.Magnesium, Metal.Calcium, Metal.Gallium)

    private val oxidizers = listOf(Nonmetal.Fluorine, Nonmetal.Chlorine, Nonmeta.Bromine, Nonmetal.Iodine, Nonmetal.Nitrogen, Nonmetal.Sulphur)

    @Test fun semimetalBonds() {
      for (element in Semimetal.values()) {
        print(element.name +":  ")
        for (metal in weakMetals) {
          print("${bind(metal, element)}, ")
        }
        for (oxidizer in oxidizers) {
          print("${bind(element, oxidizer)}, ")
        }
        println()
      }
    }
  }
```


# 3. Säuren und Basen
Wie kann man jetzt Säuren und Basen darstellen?

Dazu brauchen wir offenbar auch eine Klasse `Bond3`, die aus 3 Atomen zusammengesetzt ist, etwa wie folgt:

```Kotlin
  data class Bond3(override val element1 :Atom, override val element2 :Atom, val element3 :Atom,
   	  override val amount1 :UByte, override val amount 2 :UByte, val amount3 :UByte) :Bond {
  	override fun toString() = """$element1${amount(amount1)}$element2${amount(amount2)}$elemebt3${amount(amount3)}"""
  }
```
Und dann müssen wir eine Funktion zum Reagieren mit Wasser schreiben (englisch hydrate).  Allerdings gibt es noch 1 Problem:  Die Basen bestehen meist aus 3 Elementen, z.B. NaOH, aber Säuren können aus 2 oder aus 3 Elementen bestehen.  Deshalb wollen wir den Rückgabetyp `List<Bond>` schreiben, wobei `Bond` dann entweder `Bond2` oder `Bond3` sein kann.  Was kann man da machen?

Die Lösung ist oben bereits angedeutet:  Wir schreiben ein weiteres Interface:
```Kotlin
  interface Bond {
    val element1 :Atom
    val element2 :Atom
    val amount1 :UByte
    val amount2 :UByte
  }
```

Damit das Ganze funktioniert, müssen wir auch die Klasse `Bond2` entsprechend anpassen:

```Kotlin
  data class Bond2(override val element1 :Atom, override val element2 :Atom,
                   override val amount1 :UByte, override val amount2 :UByte) :Bond {
   	...
  }
```

Jetzt können wir die Methode implementieren:

```Kotlin
  fun hydrate(element :Atom) :List<Bond> = when (element) {
    is Metal, is Semimetal -> element.valences.mapTo { v -> 
        Bond3(element, Nonmetal.Oxygen, Nonmetal.Hydrogen, 1u, abs(v).toUByte(), abs(v).toUByte()) 
      }

    Nonmetal.Fluorine, Nonmetal.Oxygen -> bind(Nonmetal.Hydrogen, element)

    in setOf(Nonmetal.Chlorine, Nonmetal.Bromine, Nonmetal.Iodine) ->
      bind(Nonmetal.Hydrogen, element).toList<Bond>() +element.valences
        .filter { it>0 }
        .map { v -> Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygene, 1u,1u, (v/2+1).toUByte()) }

    in setOf(Nonmetal.Sulfur, Nonmetal.Selenium) ->
       bind(Nonmetal.Hydrogen, element).toList<Bond>() +element.valences
         .filter { it>0 }
         .map { v -> Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygene, 2u,1u, (v/2+1).toUByte()) }

    ...

    else -> emptyList()
  }
```
Ok, also für die Metalle und Halbmetalle ist es nicht so schwer, sie bilden einfach Hydroxide entsprechend ihrer Valenzen.

Für die Halogene unterscheidet es sich in Fluor und die restlichen (außer Astat, welches ich einfach als "Halbmetall" eingeteilt habe).  Bei Fluor gibt es nur Fluorwasserstoff, dessen Bindung(en) wir bereits mit `bind(...)` berechnen können (das Ergebnis ist ein `Bond2`).  Für die anderen gibt es noch die chlorige Säure, Chlorsäure und Perchlorsäure.  In diesen hat das Halogen eine positive Valenz.  Deshalb wird als erstes auf `{ it>0 }` gefiltert.  Entsprechendes gibt es für Brom und Iod.

Für die restlichen Elemente der Sauerstoffgruppe gibt es neben dem Schwefelwasserstoff noch 2 mögliche 3er Bindungen:  Schwefelige Säure (H$_2$SO$_3$) und Schwefelsäure (H$_2$SO$_4$), und entsprechendes für Selen.

Für Sticksoff gibt es neben NH$_3$ (Ammoniak) noch die salpetrige Säure (HNO$_2$) und die Salpetersäure (HNO$_3$).  Für Phosphor gibt es neben Phosphin (PH$_3$) noch hypophosphorige Säure (H$_3$PO$_2$), phosphorige Säure (H$_3$PO$_3$) und Phosphorsäure (H$_3$PO$_4$).

Schließlich gibt es für Kohlenstoff nur die Kohlensäure.  Diese 3 Teile müssen wir noch ergänzen:
```Kotlin
  fun hydrate(element :Atom) :List<Bond> = when (element) {
    ...

    Nonmetal.Nitrogen ->
      bind(element, Nonmetal.Hydrogen).toList() + listOf(
        Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygen, 1u,1u,2u),
        Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygen, 1u,1u,3u)
      )

    Nonmetal.Phosphorus -> listOf(
      Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygen, 3u,1u,2u),
      Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygen, 3u,1u,3u),
      Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygen, 3u,1u,4u)
    )

    Nonmetal.Carbon ->  listOf(
      Bond3(Nonmetal.Hydrogen, element, Nonmetal.Oxygen, 2u,1u,3u)
    )

    ...
  }
```

Jetzt kannst du das testen:

```Kotlin
  class AcidBaseTester {
    @Test fun bases() {
      for (metal in Metal.values()) {
      	val bases = hydrate(metal)
        println("$metal: $bases")
      }
    }

    @Test fun semiMetalAcids() {
      for (element in SemiMetal.values()) {
        val acids = hydrate(element)
        println("$element: $acids")
      }
    }

    @Test fun nonMetalAcids() {
      for (element in Nonmetal.values()) {
        val acids = hydrate(element)
        println("$element: $acids")
      }
    }
  }
```
Im Prinzip kannst du auch noch einen Test schreiben, dass sich Edelgase nicht hydratisieren lassen.

## 3.2 Was ist Säure und was ist Base?

Im Prinzip kann man das auch noch entscheiden.  Dazu müssen wir aber das Interface `Bond` modifizieren, etwa wie folgt:
```Kotlin
  interface Bond {
    ...
    val type :Type
      get() = when {
      	element1==Nonmetal.Hydrogen && element2==Nonmetal.Oxygen -> Type.Polar
        element1==Nonmetal.Hydrogen -> Type.Acid
        element1 is Metal && element2==Nonmetal.Oxygen && this is Bond3 && element3==Nonmetal.Hydrogen -> Type.Base
        element1 is Semimetal && element2==Nonmetal.Oxygen && this is Bond3 && element3==Nonmetal.Hydrogen -> Type.Acid
        element1==Nonmetal.Nitrogen && element2==Nonmetal.Hydrogen -> Type.Base

        element1 is Metal && element2 in oxidizers -> Type.Ionic
        element1 in setOf(Nonmetal.Carbon, Nonmetal.Phosphorus, Nonmetal.Selenium) && element2 in setOf(Nonmetal.Fluoride, Nonmetal.Oxygen, Nonmetal.Chlorine) -> Type.Polar
        else -> Type.Nonpolar
      }

    enum class Type {
      Acid, Base, Ionic, Polar, Nonpolar
    }
  }

  val oxidizers = setOf(Nonmetal.Fluorine, Nonmetal.Oxygen, Nonmetal.Chlorine, Nonmetal.Nitrogen, Nonmetal.Bromine, Nonmetal.Nitrogen, Nonmetal.Iodine)
```

Wenn du jetzt die `toString()`-Methoden abänderst, sodass sie den Bindungstyp in Klammern schreiben, dann kannst du die Unterscheidung im Test sehen:
```Kotlin
  data class Bond2(...) :Bond {
    override fun toString() = """$element1${amount(amount1)} $element2${amount(amount2)} ($type)"""
  }

  data class Bond3(...) :Bond {
  	override fun toString() = """$element1${amount(amount1)} $element2${amount(amount2)} $element3${amount(amount3)} ($type)"""
  }
```

# 9. Selber Probieren

Uff, das war eine ganz schöne Fleißarbeit.  Tja, Chemie ist viel Systematik, aber auch einige Ausnahmen.

Ich hoffe, dass du die Programme selbst probiert hast.

## 9.1 Organische Chemie

Neben der anorganischen Chemie gibt es auch die organische Chemie.  Hier kannst du vor allem mit den Alkanen anfangen, also Methan, Ethan, Propan, Butan, ... .

Als nächstes kannst du die Derivate bilden, also Säuren, Alkohole, Aldehyde, Alkene, Alkine.

Und dich schließlich mit Liganden (Resten, engl. moiety) versuchen.

Viel Spaß beim Probieren.
