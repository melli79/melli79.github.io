Heute wollen wir Schlümpfe zählen.  Das geht so:

0. Wir fangen mit den normalen Zahlen bei 1 an: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, ...

1. Dann ersetzen wir jede 5. Zahl durch Schlümpf entsprechend ihrem Vielfache:
```log
  1, 2, 3, 4, Schlümpf, 6, 7, 8, 9, 2 Schlümpf, 11, 12, 13, 14, 3 Schlümpf, 16, ...
```

3. die 25 ist Schlümpf-Schlümpf.

#  0. Zahlenreihe
Vom ersten Post erinnerst du dich bestimmt noch an for-Schleifen, mit denen man durchzählen kann, etwa so:

```Kotlin
  fun main() {
    for (n in 1..100) {
      print("$n, ")
    }
    println()
  }
```

# 1. Schlümpf
Offenbar müssen wir bei jeder Zahl prüfen, ob sie durch 5 teilbar ist.  Das macht man so:
```Kotlin
  n%5 == 0
```

Es bedeutet, dass man den Rest beim Dividieren von `n` durch 5 ausrechnet und dann schaut, ob der 0 ist.  Beispielsweise ist `1%5==1` also nicht gleich 0, `2%5==2`, `3%5==3`, `5%5==0`, `6%5==1` und so weiter.

Falls `n%5==0`, dann müssen wir "Schlümpf, " ausgeben, ansonsten "$n, ".  Wie bringen wir das dem Computer bei?  Kotlin ist an Englisch anlgelehnt und im Englischen heißt falls `if` (nicht when, da ja nicht klar ist, ob die Bedingung erfüllt wird), ansonsten heißt `else`, also:

```Kotlin
  if (n%5 == 0)
    print("Schlümpf, ")
  else
    print("$n, ")
```

Damit wir nicht alles ins Hauptprogramm stopfen, wollen wir das Schlumpfen in eine Funktion stecken:

```Kotlin
  fun smurfify(n :Int) :String {
    if (n%5 != 0)
      return n.toString()
    return "Schlümpf"
  }
```

`fun` kennst du sicherlich noch von der ersten Lektion, `fun main()` ist das Hauptprogramm.  Unser Teilstück heißt jetzt `smurfify` (Verschlumpfung).  Dazu muss man natürlich wissen, welche Zahl verschlumpft werden soll, also übergeben wir eine Zahl `(n :Int)`.  Die Zahl heisst im Teilprogramm `n` und `Int` gibt an, dass eine ganze Zahl (also nicht $0.5$ oder "Hallo") übergeben wird.

Natürlich wollen wir auch das Ergebnis zurück geben, also schreiben wir `:String`, d.h. es wird am Ende Text zurück gegeben.

Nun zu den 2 Fällen:  Es wird kontrolliert, ob `n` bei Division durch 5 nicht den Rest 0 ergibt, also nur 1, 2, 3 oder 4.  In diesem Fall wird, `n.toString()` zurück gegeben, also die Zahl als Text.  `return` bewirkt noch etwas anderes, nämlich, dass die Funktion hier beendet wird.  Die letzte Zeile wird in diesem Fall also gar nicht mehr ausgeführt.

Wenn `n` aber durch 5 teilbar ist (`n%5 == 0`), dann wird die erste `return`-Anweisung nicht ausgeführt.  Stattdessen geht es mit dem nächsten Befehl weiter.  Da wird dann "Schlümpf" zurück gegeben (und die Funktion beendet).

Jetzt sieht unser Programm etwa so aus:

```Kotlin
  fun smurfify(n :Int) :String {
    if (n%5 != 0)
      return n.toString()
    return "Schlümpf"
  }

  fun main() {
    for (n in 1..30) {
      val number = smurfify(n)
      print("$number, ")
    }
    println()
  }
```

Und das Ergebnis so:

```log
  1, 2, 3, 4, Schlümpf, 6, 7, 8, 9, Schlümpf, 11, 12, 13, 14, Schlümpf, 16, 17, 18, 19, Schlümpf, ...
```

Immerhin, jede 5. Zahl ist durch "Schlümpf" ersetzt.


# 2. Wie kann man die Vielfache ausgeben?

Eigentlich sollen wir für 10 aber "2 Schlümpf" schreiben.  Die 2 ist 10/5.  Das können wir so erreichen:

```Kotlin
  fun smurfify(n :Int) :String {
    if (n%5 != 0)
      return n.toString()

    var factor = n/5
    return "$factor Schlümpf"
  }
```

Wenn du das Programm wieder startest, dann sieht das Ergebnis so aus:

```log
  1, 2, 3, 4, 1 Schlümpf, 6, 7, 8, 9, 2 Schlümpf, 11, 12, 13, 14, 3 Schlümpf, 16, 17, 18, 19, 4 Schlümpf, 21, 22, 23, 24, 5 Schlümpf, 26, 27, 28, 29, 6 Schlümpf,
```

## 2.1 Die "1" kann weg

Ok, offenbar müssen wir noch einen Fall unterscheiden:  Wenn der `factor` 1 ist, dann nur "Schlümpf" ausgeben, also:

```Kotlin
  fun smurfify(n :Int) :String {
    if (n%5 != 0)
      return n.toString()

    var factor = n/5
    if (factor==1)
      return "Schlümpf"
    return "$factor Schlümpf"
  }
```

Jetzt aber:

```log
  1, 2, 3, 4, Schlümpf, 6, 7, 8, 9, 2 Schlümpf, 11, 12, 13, 14, 3 Schlümpf, 16, 17, 18, 19, 4 Schlümpf, 21, 22, 23, 24, 5 Schlümpf, 26, 27, 28, 29, 6 Schlümpf,
```

# 3. "25" ist Schlümpf-Schlümpf

Eine Sache stört noch:  Die 25 ist nicht einfach "5 Schlümpf", sondern der `factor` sollte 1 sein und die Endung "Schlümpf-Schlümpf".  Das kann man so erreichen:

```Kotlin
  fun smurfify(n :Int) :String {
    if (n%5 != 0)
      return n.toString()

    var factor = n/5
    var suffix = "Schlümpf"
    while (factor%5 == 0) {
      factor /= 5
      suffix += "-Schlümpf"
    }

    if (factor==1)
      return suffix
    return "$factor $suffix"
  }
```

`while (bedingung) { ... }` bedeutet, dass die `bedingung` getestet wird.  Wenn die wahr ist, wird der Teil zwischen `{` und `}` ausgeführt.  Dann wird die Bedingung wieder getestet und so weiter, bis die Bedingung irgendwann nicht mehr erfüllt ist.

Nun sieht die Ausgabe wie folgt aus:

```log
  1, 2, 3, 4, Schlümpf, 6, 7, 8, 9, 2 Schlümpf, 11, 12, 13, 14, 3 Schlümpf, 16, 17, 18, 19, 4 Schlümpf, 21, 22, 23, 24, Schlümpf-Schlümpf, 26, 27, 28, 29, 6 Schlümpf,
```

# 4. Selber probieren

Kannst Du eine Funktion `fun congee(n :Int) :String` schreiben, die jede 3. Zahl durch "Brei" (engl. congee) ersetzt?

Wie muss man die Funktion ins Hauptprogramm einbauen?

Kannst du die Funktion noch verfeinern?
