Nachdem Ihr die Programmierumgebung aufgesetzt habt, fangen wir heute mit einem ersten kleinen Programm an: Das kleine 1x1.

# Das Ziel
Das Ziel soll etwa folgende Tabelle sein:

<pre>
   1  2  3  4  5  6  7  8  9 10
   2  4  6  8 10 12 14 16 18 20
   3  6  9 12 15 18 21 24 27 30
   4  8 12 16 20 24 28 32 36 40
   5 10 15 20 25 30 35 40 45 50
   6 12 18 24 30 36 42 48 54 60
   7 14 21 28 35 42 49 56 63 70
   8 16 24 32 40 48 56 64 72 80
   9 18 27 36 45 54 63 72 81 90
  10 20 30 40 50 60 70 80 90 100
</pre>

Was wollen wir haben:  Zeilen 1..10, die die Vielfache der jeweiligen Zahl enthalten.
In jeder Zeile die Zahl, dann ihr Doppeltes, dann das Dreifache, ..., das 10-fache.

# Welche Elemente brauchen wir?

1. Eine Variable kann einen bestimmten Wert enthalten, z.B. bedeutet
```Kotlin
  val x = 3
```
Das wir eine neue Variable mit Namen x anlegen, die den Wert 3 enthält.

Entsprechend wäre
```Kotlin
  val name = "Melchior"
```
eine Variable mit Namen `name`, die den Text Melchior (also meinen Namen) enthält.

Natürlich wollen wir die Vielfache nicht alle von Hand eintippen, schließlich sollte ein Rechner rechnen können.  Das geht etwa so:
```Kotlin
  x*y
```
Wenn wir davon ausgehen, dass die Variablen x und y Zahlen enthalten, dann bedeutet `x*y`, dass deren Produkt berechnet wird.  Entsprechend kann man mit `4+5` die Summe von 4 und 5 berechnen.

## Was soll mit den Zahlen pasieren?

Wir wollen sie auf der Kommandozeile ausgeben.  Im einfachsten Fall geht das mit

```Kotlin
  print(x*y)
```
Alles, was zwischen ( und ) steht, wird von print ausgegeben.  Print kommt aus dem Englischen und heißt wörtlich ausdrucken.  Entsprechend kommt auch `val` von value aus dem Englischen und bedeutet Wert, d.h. `val x = 3` ist eigentlich nur ein benannter Wert.

## Wir brauchen noch eine Zutat:

Wir wollen in jeder Zeile 10 Ausdrücke haben und das für 10 Zeilen wiederholen.

Das kann man für eine Zeile so erreichen

```Kotlin
  val y = 5
  for (x in 1..10) {
    print(x*y)
  }
  println()
```

Den ersten Ausdruck kennen wir schon `val y = 5` bedeutet, dass der Name `y` für den Wert 5 steht.
`for (x in 1..10)` bedeutet, dass der Ausdruck zwischen `{` und `}` für alle Zahlen von 1 bis 10 wiederholt wird. D.h. im ersten Durchlauf ist `x=1`, im zweiten Durchlauf `x=2` und so weiter bis einschließlich `x=10`.

Auch den Ausdruck `print(x*y)` kennen wir, hier wird das Produkt berechnet und an der Konsole ausgegeben.

## Wie kommen wir nun zur ganzen Tabelle?

Wir dürfen den obigen Block nicht nur für `y=5` ausführen, sondern müssen ihn für `y` in `1..10` widerholen, also wie folgt:

```Kotlin
  for (y in 1..10) {
    for (x in 1..10) {
      print(x*y)
    }
    println()
  }
```

## Was bedeutet `println()`?

Print gibt etwas auf der Konsole aus, `println()` gibt ein Zeilenende aus.

Wenn man also die Zeile `println()` weglässt, dann erscheint die ganze Tabelle
auf 1 Zeile (bis die Zeile überläuft).

## Wo fängt der Computer das Programm an?

Dazu muss man den ersten Teil des Programms zwischen die Zeilen

```Kotlin
  fun main() {
    ...
  }
```
schreiben.  Main heisst Hauptprogramm (so wie in main dish, Hauptgericht). Fun ist die Abkürzung für function, dass sind die Teile eines Programms.  Für jedes ausführbare Programm brauchen wir zumindest einen Hauptteil `fun main()`.

Insgesamt muss das Programm also wie folgt aussehen:

```Kotlin
  fun main() {
    for (y in 1..10) {
      for (x in 1..10) {
        print(x*y)
      }
      println()
    }
  }
```

# Wie kann ich das Programm ausprobieren?

Wenn die Entwicklungsumgebung richtig eingerichtet ist und das Programm in einem neuen Projekt geschrieben wurde, dann muss man jetzt nur noch auf den grünen Pfeil am linken Rand klicken und schon wird das Programm übersetzt und ausgeführt.

## Warum muss man das Programm übersetzen?

Die Wahrheit ist, dass der Computer nur Maschinensprache versteht.  Die kann man zum Beispiel in einer ausführbaren Datei speichern.  Leider versteht der Computer kein Kotlin.

Trotzdem ist es sinnvoll, in Kotlin zu programmieren, weil es ein Programm gibt (der Compiler, hier Bestandteil der Entwicklungsumgebung), dass das Kotlin-Programm in ein ausführbares Programm übersetzt.  Normaler Weise geht das ziemlich schnell, aber es ist eben notwendig.

## Wo sehe ich das Ergebnis?

Wenn man das Programm wie oben beschrieben gestartet hat, dann öfftnet sich ein neues Fenster, in dem man die Ausgabe des Programmes sieht (so wie das Bild am Anfang zeigt).  In diesem Fenster kommt viel Ausgabe heraus, vielleicht musst du etwas zurück rollen.


# Wie kann ich das Programm besser machen?

Wenn man sich die Ausgabe anschaut, stehen alle Zahlen einer Zeile direkt hintereinander ohne Leerzeichen.  Dabei nehmen die 1-stelligen Zahlen 1 Zeichen ein, die 2-stelligen 2 Zeichen und die 100, 3 Zeichen.

## Wie kann ich alle Zahlen 2-stellig machen?

```Kotlin
  print((x*y).toString().padStart(3))
```
`.toString(...)` bedeutet, dass das Argument (davor) in einen Text verwandelt wird,
`.padStart(3)` bedeutet, dass der text auf 3 Zeichen aufgefüllt wird (von links mit Leerzeichen). Das erste sollte immer ein Leerzeichen bleiben.

Q: Was passiert mit 100? (die ist offenbar selbst 3-stellig)

## Ich möchte das große 1x1 ausgeben, wie geht das?

Schau mal auf die Zeile `for (y in 1..10)`.  Die 10 bedeutet, dass der letzte Durchlauf mit `y=10` erfolgt.  Also was muss man als erstes tun?

### Wie bekomme ich die Tabelle nun wieder quadratisch?

Wir haben 2 for-Schleifen `for (y in ...)` und `for (x in ...)`, d.h. du musst auch die 2. for-Schleife anpassen.


Viel Spaß beim Probieren...

Beim nächsten mal erzähle ich über bedingte Ausführung, wie man Herr und Frau richtig zuordnet.
