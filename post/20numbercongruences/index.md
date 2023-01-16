Im Dritten Teil der Serie, Einführung in das Programmieren, wollen wir weitere mathematische Spiele betreiben.  Heute wollen wir mit Zahlenkongruenzen rechnen.

# 1. Etwas Theorie

Du erinnerst dich vielleicht noch an das erste Programm, in dem wir die Multiplikationstabelle (also das kleine 1x1) dargestellt haben.  Diesmal wollen wir das aber etwas abändern.  Statt dem ganzen Produkt geben wir nur den Rest bei Division durch die ganze Zahl $n$ aus.  Das bedeutet folgendes:  Wenn wir $3 * 5$ ausrechnen und $n=6$, dann dividieren wir das Zwischenergebnis $15$ durch $6$ und erhalten $2$ Rest $3$.  Das heißt, dass $2 * 6+3=15$ ist.  Wir wollen also hier die $3$ ausgeben.

In Kotlin (oder allen C-ähnlichen Programmiersprachen) kann man den Rest bei Division ausrechnen, indem man `p%n` schreibt.  Dabei müssen `p` und `n` ganze Zahlen sein (`Int` oder `Long`).


# 2. Experimentieren
Wir modifizieren also das kleine 1x1 aus der ersten Lektion wie folgt:

```kotlin
  fun main() {
    print("Bitte geben Sie eine positive ganze Zahl für den Modulus ein: ")
    var n :Int?
    while (true) {
      n = readlnOrNull()?.trim()?.toIntOrNull()
      if (n==null || n<=0) {
        println("Das war keine positive Zahl.")
        continue
      }
      break
    }
    println("Multiplikation modulo $n:")
    for (y in 0 until n!!) {
      for (x in 0 until n) {
        print("%2d ".format(x*y%n))
      }
      println()
    }
  }
```

Wenn du das Programm fehlerfrei abgetippt hast, startest und am Prompt `6` eingibst, erhältst du etwa folgende Tabelle:
```log
  Multiplikation modulo 6:
  0  0  0  0  0  0
  0  1  2  3  4  5
  0  2  4  0  2  4
  0  3  0  3  0  3
  0  4  2  0  4  2
  0  5  4  3  2  1
```

Wenn du stattdessen $n=7$ eintippst, erhältst du folgendes:

```log
Multiplikation modulo 7:
  0  0  0  0  0  0  0
  0  1  2  3  4  5  6
  0  2  4  6  1  3  5
  0  3  6  2  5  1  4
  0  4  1  5  2  6  3
  0  5  3  1  6  4  2
  0  6  5  4  3  2  1
```

Wenn man beide Tabellen vergleicht, fallen folgende Gemeinsamkeiten auf:

1. Die Tabellen sind symmetrisch bezüglich der Diagonale, d.h. $y * x\equiv x * y\pmod{n}$.
2. Die nullte Zeile und nullte Spalte besteht nur aus 0, d.h. $0*x\equiv0\pmod{n}$.
3. Die erste Zeile und erste Spalte enthält die ursprünglichen Reste, d.h. $1*x\equiv x\pmod{n}$.
4. Es gibt ein paar Zeilen (Spalten), in denen alle Reste auftauchen, z.B. 1,5 für $n=6$ und alle (außer 0) für $n=7$.

Vielleicht können wir das Programm vereinfachen und nur die Zeilen `1..(n-1)` ausgeben, etwa so:
```kotlin
  for (y in 1 until n) {
    for (x in 1 until n) {
      print("%2d ".format(x*y%n))
    }
    println()
  }
```

Jedenfalls bleiben für $n=6$ die Nullen bei 2, 3 und 4.  Was haben diese Zahlen besonderes im Zusammenhang mit Division durch 6?

Nun, 2 lässt sich durch 2 teilen genauso wie 6, d.h. 2 ist ein gemeinsamer Teiler (der größer als 1 ist).  Was passiert mit so einem gemeinsamen Teiler?

Nennen wir ihn $g=2>1$, dann ist $f:=n/g=6/2=3<6=n$ ein Rest modulo $n$ und $x=2$ multipliziert mit diesem Rest ist durch $n$ teilbar, also $fx\equiv0\pmod{n}$.  So einen Fall nennt man Nullteiler, weil wir offenbar $0$ durch diesen Teiler $f$ teilen können (modulo $n$) und einen positiven Rest erhalten.  (Umgekehrt ist $x$ dann auch ein Nullteiler, weil wir $n$ durch $x$ teilen können und modulo $n$ einen positiven Rest erhalten.)

Wenn also der Rest $x$ und der Modulus $n$ einen gemeinsamen Teiler $g>1$ haben, dann ist $x$ ein Nullteiler, d.h. es tauchen Nullen in der Multiplikationszeile auf (auch außerhalb von $f=0$).  Im anderen Fall, also wenn $1$ der einzige gemeinsame Teiler zwischen $x$ und $n$ ist, dann enthält die entsprechende Zeile alle Reste (und 0 nur für $f=0$).  In diesem Fall sagt man, dass $x$ und $n$ teilerfremd sind.

Das erklärt auch, warum die Zeilen für 3 und 4 ebenfalls Nullen enthalten modulo 6.  Die Zeilen mit 1 und 5 entsprechend nicht.  Bei Rechnung Modulo 7 enthält offenbar keine Zeile (außer für $x=0$) weitere Nullen.  Woran liegt das?

Nun offenbar sind die einzigen Teiler von 7 die 1 und die 7.  Das beudetet, dass 7 mit jedem Rest modulo 7 teilerfremd ist.  Bei 6 war das nicht der Fall.  Bei $n=5$ ist das entsprechend wieder der Fall:

```log
Multiplikation modulo 5:
  1  2  3  4
  2  4  1  3
  3  1  4  2
  4  3  2  1
```

Zahlen, die genau 2 positive Teiler haben, nennt man Primzahlen.  Somit sind 5 und 7 Primzahlen, aber 6 ist keine Primzahl.

# Primzahlen finden (Sieb des Erathostenes)

Wie kann man nun Primzahlen finden?

Eine Möglichkeit besteht darin, für eine gegebene Zahl $n$ zu testen, ob sie durch irgendeine der Zahlen $1<r<n$ teilbar ist, d.h. den Rest 0 lässt.  Wenn man sich überlegt, dass $n=rf$ sein soll, dann muss $r$ oder $f$ kleiner als $n/2$ sein.  Bei genauerer Untersuchung sieht man, dass es auch reicht, von $1<r$ bis $r*r\le n$ zu suchen.

Statt aber jede Zahl $1<n\le N$ einzeln zu überprüfen, hatte Erathostenes eine andere Idee:  Wir fangen einfach von unten an: 1 ist die Einheit, d.h. $1 * 1=1$ und somit keine Primzahl.  Die nächste Zahl wurde noch nicht gestrichen, ist also eine Primzahl $p=2$.  Jetzt streichen wir alle echten Vielfache von $p$ bis zur Grenze $N$ und suchen anschließend nach der nächsten verbleibenden Zahl, hier $p=3$.  Mit dieser verfahren wir dann entsprechend, bis wir bei einer Zahl $p$ angekommen sind mit $p*p> N$.  Dann können wir mit Streichen aufhören und die verbleibenden Zahlen sind die Primzahlen.

Wie kann man das den Computer machen lassen?

Nun, offenbar müssen wir zunächst eine obere Grenze, z.B. $N=100$ festlegen und uns dann alle Zahlen von 1  bis $N$ merken.  Also legen wir ein Feld (engl. `Array`) oder eine Liste (engl. `List`) an mit Indizes `0..N` (Indizes fangen immer bei 0 an) und müssen uns merken, ob die Zahl gültig (`true`) oder ungültig (`false`) ist.  Einen solchen Datentyp nennt man Boolschen Wert (engl. `Boolean`) nach George Boole.

```kotlin
  fun main() {
    val N=100
    val isPrime = (0..N).map { true }.toMutableList()
    ...
  }
```

D.h. dass die Liste/das Array `isPrime` Werte enthalten wird, ob die jeweilige Zahl eine Primzahl oder eine zusammengesetzte Zahl ist.  Wir fangen offenbar mit lauter wahr (engl. `true`) an.  Das kann aber nicht die korrekte Antwort sein, also müssen wir die Liste entsprechend veränderbar machen (engl. mutable).

Jetzt fangen wir wie oben beschrieben an und streichen die 0 (die Null) und die 1 (die Einheit):

```kotlin
  ...
  isPrime[0] = false;  isPrime[1] = false
  var p=2
  while (p*p<=N) {
    ...
  }
```

Ok, die erste (aber sicherlich nicht die letzte) Primzahl ist $p=2$.  Wie oben beobachtet müssen wir nur so lange streichen, bis wir bei $p*p>N$ angekommen sind.  Wie funktioniert das Streichen?

```kotlin
  while (...) {
    var m = 2*p
    while (m<=N) {
      isPrime[m] = false
      m += p
    }
    p++
    while (p<=N && !isPrime[p])
      p++
  }
```
D.h. dass wir vom ersten echten Vielfache $m=2p$ anfangen, dieses streichen (`isPrime[m] = false`) und dann zum nächsten Vielfache übergehen, d.h. $m$ um $p$ erhöhen.  Genau das bedeutet `m += p`.  Wichtig ist dabei, dass wir `m` als veränderbar (`var m`) definiert haben.

Wenn wir alle echten Vielfache von `m` gestrichen haben, dann gehen wir zur nächsten Primzahl über, d.h. wir erhöhen $p$ um 1 (`p++`) und testen dann, ob zum einen $p$ selbst noch $p\le N$ ist, und zum anderen, ob $p$ noch eine Primzahl sein kann.  Solange es keine Primzahl ist (`!isPrime[p]`) erhöhen wir $p$ weiter.

Wenn wir sehen wollen, wie weit das Programm schon gekommen ist, dann sollten wir jede gefundene Primzahl ausgeben, z.B. so:

```kotlin
  ...
  while (p*p<=N) {
    print("$p, ");  out.flush()
    var m = ...
  }
```

Wenn du das Programm jetzt startest, solltest du ziemlich schnell die Zahlen

```log
  2, 3, 5, 7,
```

sehen.  Danach beendet sich das Programm.  Was ist mit den restlichen Primzahlen bis 100?

Naja, die müssten wir auch noch ausgeben, etwa so:

```kotlin
  while (...) {
    ...
    p++
    while (p<=N && !isPrime[p])
      p++
  }
  while (p<=N) {
    while (p<=N && !isPrime[p])
      p++
    print("$p, ")
    p++
  }
  println("...")
```

Jetzt sollte die Ausgabe für `N=100` so aussehen:

```log
  2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, ...
```

Das bedeutet, dass nicht nur 5 und 7, sondern auch 2 und 3 unter 10, sowie 11, 13, ... Primzahlen unter 100 sind.

Du kannst dich davon überzeugen, indem du dir mit dem ersten Programm die Multiplikationstabelle modulo, z.B. 37, ausgeben lässt.

# 9. Selbst probieren

Jetzt bist du dran.  Funktionieren die Programme bei dir auch?  Kommen die gleichen Ergebnisse heraus?

## Frage 1:  Wie kannst du das erste Programm abändern, um eine Additionstabelle modulo $n$ zu erzeugen?

Hinweis 1:  In `(x+y)%n` darfst du die Klammern nicht vergessen.


## Frage 2:  Wie kannst du überprüfen, ob 2021 eine Primzahl ist?  Was ist mit 2023?

Frage 2b:  Wie lässt sich eine Zahl $n$ in ihre Primfaktoren zerlegen?

Hinweis 2:  Du könntest die Primzahlen bis `val N = isqrt(n)` finden, und testen, durch welche sie teilbar ist, jeweils dividieren und mit dem Quotienten weiter arbeiten, bis du alle Primzahlen $<N$ überprüft hast.

Bemerkung:  Falls es die Funktion `isqrt` (integral square root) nicht gibt, kannst du die wie folgt implementieren:
```kotlin
  fun isqrt(x :Int) = sqrt(x.toDouble()).toInt()
```

Hinweis 3:  Du könntest auch gleich bei $p=2$ anfangen, für die aktuelle "Prim"-Zahl $p$ schauen, wie oft $n$ durch $p$ teilbar ist und jedes mal $n$ verkleinern (`n /= p`) und $p$ ausgeben.  Wenn es mit $p$ nicht weitergeht, dann musst du die 2 auf $p=3$ erhöhen und für alle anderen "Prim"-Zahlen einfach 2 addieren (`p += 2`).  Du kannst abbrechen, wenn $p*p>n$ ist.  Falls der letzte Quotient $n>1$ ist, ist es ebenfalls ein Primteiler (also auch ausgeben).

Frage 2c:  Warum werden im letzen Beispiel nur Primzahlen ausgegeben, obwohl $p$ alle ungeraden Zahlen $>1$ durchläuft?

Hinweis 4:  Du kannst dir das am Beispiel $p=9$ überlegen:  Überlege zunächst, warum 9 keine Primzahl ist.  Überlege dir dann, was mit den Primteilern von 9 passiert.

Viel Spaß beim Probieren.
