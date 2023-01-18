Nachdem wir letztes Mal Primzahlen gefunden haben und Multiplikation modulo $n$ betrieben haben, wollen wir heute Potenzen modulo $n$ betrachten.

# 1. Potenz-Tabelle

Analog zur Multiplikationstabelle, wollen wir diesmal potenzieren modulo einer positiven ganzen Zahl $n$.  Wenn wir $b^e\pmod{n}$ ausrechnen, haben wir mathematisch 2 Möglichkeiten:  Wir können entweder mit großen ganzen Zahlen rechnen und zunächst $b^e$ ausrechnen und dann den Rest modulo $n$ bestimmen.  Oder wir multiplizieren in einem fort, d.h. fangen bei $b^2\equiv b * b \pmod{n}$ an, bestimmen den Rest modulo $n$ und gehen dann weiter zu $b^3\equiv b^2 * b \pmod{n}$.  Im letzteren Fall können wir sicher sein, dass das Produkt zwischendurch nie den Wert $n^2$ übersteigt.

So erzeugen wir die Potenztabelle:

```kotlin
  fun main() {
    print("Bitte geben Sie eine positive ganze Zahl für den Modulus ein: ")
    var n :Int?
    while (true) {
      n = readlnOrNull()?.trim()?.toIntOrNull()
      if (n==null || n<=1) {
        print("Das ist keine positive Zahl größer 1.")
        continue
      }
      break
    }
    println("Potenzen modulo $n:")
    println((0..n!!).map { e -> "%2d".format(e) }.joinToString(" "))
    for (b in 0 until n) {
      var p = 1
      for (e in 0..n) {
        print("%2d ".format(p))
        p = (p*b)%n
      }
      println()
    }
  }
```

# 2. Experimentieren

Die Tabelle für $n=5$ sieht wie folgt aus:
```log
  Potenzen modulo 5:
  0  1  2  3  4  5
  1  0  0  0  0  0
  1  1  1  1  1  1
  1  2  4  3  1  2
  1  3  4  2  1  3
  1  4  1  4  1  4
```

Die Tabelle für $n=6$ sieht wie folgt aus:
```log
Potenzen modulo 6:
 0  1  2  3  4  5  6
 1  0  0  0  0  0  0
 1  1  1  1  1  1  1
 1  2  4  2  4  2  4
 1  3  3  3  3  3  3
 1  4  4  4  4  4  4
 1  5  1  5  1  5  1
```

Schließlich sieht die Tabelle für $n=9$ wie folgt aus:
```log
Potenzen modulo 9:
 0  1  2  3  4  5  6  7  8  9
 1  0  0  0  0  0  0  0  0  0
 1  1  1  1  1  1  1  1  1  1
 1  2  4  8  7  5  1  2  4  8
 1  3  0  0  0  0  0  0  0  0
 1  4  7  1  4  7  1  4  7  1
 1  5  7  8  4  2  1  5  7  8
 1  6  0  0  0  0  0  0  0  0
 1  7  4  1  7  4  1  7  4  1
 1  8  1  8  1  8  1  8  1  8
```

## 2.1 Wir können folgende Dinge beobachten:

1. Die Potenzen von 0 sind fast alle 0, denn $0 * 0\equiv0\pmod{n}$;
2. Die Potenzen von 1 sind alle 1, denn $1 * 1\equiv1\pmod{n}$;
3. Die Potenzen von 2 durchlaufen alle Reste (außer 0) modulo 5;
4. Die Tabelle für 5 sieht "ordentlich" aus, d.h. außer für $b=0$ tauchen keine 0en auf;
5. Die Tabellen für 5 und 6 sehen nett aus, z.B. $b^n\equiv b\pmod{n}$;
6. Die Tabelle für 9 sieht ziemlich "löchrig" aus, d.h. es gibt etliche 0en auch außerhalb von $b=0$.

## 2.2 Woran kann das liegen?

Wir haben bereits herausgefunden, dass 5 eine Primzahl ist.  Auch hatten wir gesehen, dass Multiplikation modulo 5 sich sehr regelmäßig verhält, d.h. es taucht keine 0 auf (außer wenn $x=0$ oder $y=0$) und in jeder Zeile kommt jeder Rest genau 1mal vor.  Dazu passt folgender Begriff:

Eine **Gruppe** ist eine Menge $G$ mit einer Operation $\circ$ (die 2 Elemente zu einem verknüpft), sodass
1. $(a \circ b) \circ c = a \circ (b \circ c)$, (Assoziativgesetz)
2. $a\circ e=a$, (neutrales Element $e$)
3. Für jedes $a$ existiert ein $b$, sodass $a\circ b=e$. (inverse Elemente)

Es gibt zahlreiche Gruppen, z.B. ist die Addition modulo $n$ stets eine Gruppe, das inverse Element zu $a$ ist $n-a$, denn $a+(n-a)\equiv n\equiv0\pmod{n}$.  Wenn du genau aufgepasst hast, dann ist hier etwas durcheinander geraten: Die "1" zur Addition heißt 0.  Deshalb spricht man bei Gruppen auch nicht von "1", sondern vom neutralen Element.  Für die Addition ist also 0 das neutrale Element.  Die Menge ist also z.B. $\mathbb Z/(5)=\\{[0], [1], [2], [3], [4]\\}$ und die Operation $+$, Addition, die 2 Zahlen (oder Restklassen) nimmt und zu einer Restklasse verknüpft.

Wir hatten ja oben auch gesehen, dass die Reste modulo 5 (außer dem Rest 0) eine Gruppe bilden.  Diese Gruppe bezeichnet man mit $\mathbb F_5^* $, die multiplikative Gruppe modulo 5.  Der obenstehende Stern erinnert daran, dass der Rest 0 auszuschließen ist.  Die Menge ist also $\mathbb F_5^* = \\{[1], [2], [3], [4]\\}$ und die Operation $*$, Multiplikation, die 2 Restklassen zu einer verknüpft.

## Warum schreibe ich die Zahlen manchmal mit eckigen Klammern und manchmal ohne?

Korrekter Weise müsste man in der Menge $\mathbb Z/(5)$ die Elemente $[0]_5$, $[1]_5$, $[2]_5$, $[3]_5$ und $[4]_5$ schreiben.  Denn man muss die von den Restklassen z.B. modulo 3 unterscheiden: $\mathbb Z/(3)=\\{[0]_3, [1]_3, [2]_3\\}$.  Das erkennt man daran, dass $1+1\equiv3\equiv0\pmod{3}$, also $[1]_3+[2]_3=[0]_3$ ist, während $1+2\equiv3\not\equiv0\pmod{5}$, also $[1]_5+[2]_5=[3]_5\ne[0]_5$ ist.
Je nachdem, wie genau man das aufschreiben muss, kann man also entweder $[1]_5$ oder $[1]$ oder auch einfach $1$ schreiben.  Im letzten Fall muss man erkennen, dass es sich um einen Rest modulo 5 handelt und nicht um die ganze Zahl 1.  Im mittleren Fall muss man erkennen, dass modulo 5 gerechnet wird.  Und im ersten Fall hat man viel zu schreiben (und umständlich zu tippen).


# 3. Zyklische Gruppen

Wenn wir uns an das Zählen erinnern, dann erkennen wir, dass in der Gruppe $\mathbb Z/(n)$ (bezüglich Addition) die Restklasse $[1]$ alle anderen Restklassen erzeugt, d.h. $[r]=[1]+\dotsm+[1]$, wenn man genau $r$ Summanden addiert.  Das nennen wir eine **zyklische** Gruppe, d.h. es gibt ein erzeugendes Element (hier die [1]) aus der sich durch wiederholtes Verknüpfen alle anderen Elemente erzeugen lassen.

Ein einfacher Satz der Gruppentheorie besagt, dass alle endlichen zyklischen Gruppen so aussehen wie die ganzen Zahlen modulo $n$.  Dabei nennt man $n$ die Ordnung der Gruppe, also die Anzahl der Elemente. "So aussehen wie" heißt in mathematischer Sprache isomorph (also von gleicher Struktur).

## 3.1 Geht das auch mit anderen Elementen?

Schauen wir uns die Vielfache modulo 6 doch mal genauer an:
```log
  Wiederholte Addition modulo 6:
  0  0  0  0  0  0  0
  0  1  2  3  4  5  0
  0  2  4  0  2  4  0
  0  3  0  3  0  3  0
  0  4  2  0  4  2  0
  0  5  4  3  2  1  0
```
Offenbar kann man statt der $[1]$ auch die $[5]$ verwenden.  Alle anderen Elemente erzeugen nur eine kleinere Teilmenge.  Deshalb schreiben wir $(\mathbb Z/(6))^* = \\{[1], [5]\\}$.  Dieser Stern bedeutet die Generatoren der additiven Gruppe.

Wie sieht das Modulo 5 aus?
```log
  Wiederholte Addition modulo 5:
  0  0  0  0  0  0
  0  1  2  3  4  0
  0  2  4  1  3  0
  0  3  1  4  2  0
  0  4  3  2  1  0
```

Offenbar kann man die ganze Gruppe $\mathbb Z/(5)$ mit jedem Element (außer der [0]) erzeugen.  Deshalb schreiben wir $(\mathbb Z/(5))^* = \\{[1], [2], [3], [4]\\}=\mathbb F_5^* $.

Spannender wird es mit der Multiplikation:  Wir hatten bereits gesehen, dass $\mathbb F_5^* $ eine Gruppe (bezüglich Multiplikation) ist.  Außerdem hatten wir aus den Potenz-Tabellen am Anfang erkannt, dass diese Gruppe von der $[2]$ erzeugt wird.  Damit ist es auch eine zyklische Gruppe.

## 3.2 Kann man die Gruppe $\mathbb F_5^* $ auch mit Addition schreiben?

Der naïve Versuch wäre $[1]+[1]=[2]$, $[1]+[2]=[3]$, $[3]+[1]=[4]$, $[4]+[1]=[0]$ -- halt! irgendetwas stimmt hier nicht.  Auf der Menge $\mathbb F_5^* = \\{[1], [2], [3], [4]\\}$ operiert die übliche Addition (modulo 5) nicht.

Die bessere Frage wäre, ob die Gruppe $\mathbb F_5^* $ isomorph zu einer der "üblichen" zyklischen Gruppen ist.  Dazu schauen wir uns nochmal die Potenzen von $[2]$ an:


|  *e*    |  0    |  1    |  2    |  3    |  4    |
|---------|-------|-------|-------|-------|-------|
| $[2]^e$ |  [1]  |  [2]  |  [4]  |  [3]  |  [1]  |


In dieser Abbildung gilt: $[2]^0=[1]$ und $[2]^{e+f}=[2]^e * [2]^f$, d.h. die Abbildung (von oben nach unten gelesen) erhält das neutrale Element und die Gruppenoperation.  Entsprechend kann man auch die inversen Elemente der Multiplikation ablesen: $[1]^{-1}=[1]$, $[2]^{-1}=[3]$, $[3]^{-1}=[2]$, $[4]^{-1}=[4]$.
So eine Abbildung nennt man einen **Gruppen-Homomorphismus**.

Aber die Abbildung hat noch eine weitere Eigenschaft:  Wenn man die Tabelle von unten nach oben liest, erkennt man, dass es auch eine Abbildung ist, d.h. das jedem Element der unteren Zeile genau 1 Element der oberen Zeile zugeordnet wird (unten fehlt kein Element und es taucht auch keines doppelt auf).  Das nennt man einen **Isomorphismus** und die Gruppen entsprechend isomorph.  So kann man also $\mathbb F_5^* $ in die additive zyklische Gruppe $\mathbb Z/(4)$ übersetzen.  Formell schreibt man das $\mathbb F_5^* \cong \mathbb Z/(4)$.  Dabei kann man nicht einfach $=$ schreiben, weil z.B. die Mengen $\mathbb F_5^* $ und $\mathbb Z/(4)$ verschiedene Zahlen enthalten.


## Wie kann man das Gleiche mit $\mathbb F_7^* $ machen?

Dazu muss man einen Generator für die Multiplikation modulo 7 finden.  Mit dem Programm vom Anfang geht das wie folgt:
```log
Potenzen modulo 7:
 0  1  2  3  4  5  6  7
 1  0  0  0  0  0  0  0
 1  1  1  1  1  1  1  1
 1  2  4  1  2  4  1  2
 1  3  2  6  4  5  1  3
 1  4  2  1  4  2  1  4
 1  5  4  6  2  3  1  5
 1  6  1  6  1  6  1  6
```
D.h. hier ist $[3]$ ein Generator für die Multiplikation modulo 7.  Der einzige andere Generator ist $[5]$.  Das kommt uns bekannt vor:  Die Addition modulo 6 hat auch 2 Generatoren ([1] und [5]).  Wo ist da der Zusammenhang?


|  *e*    |  0  |  1  |  2  |  3  |  4  |  5  |  6  |
|---------|-----|-----|-----|-----|-----|-----|-----|
| $[3]^e$ | [1] | [3] | [2] | [6] | [4] | [5] | [1] |

Also hier $\mathbb F_7^* \cong \mathbb Z/(6)$.

Insgesamt gilt der kleine Satz von Fermat: Für eine Primzahl $p$ ist
 $$ a^{p-1}\equiv 1 \pmod{p}$$
solange $a$ kein Vielfaches von $p$ ist.

Der Grund dafür ist die obige Beobachtung, dass die Multiplikation in $\mathbb F_p^* $ eine zyklische Gruppe der Ordnung $p-1$ ist (die 0 wird ja weg gelassen).


# 4. Kann man das Praktisch verwenden?

Ja, wenn man die folgende Beobachtung von Euler dazu ergänzt:

Corollar (Satz von Euler & Fermat, Teil 1):  Gegeben eine Primzahl $p$, dann gilt
 $$ a^p\equiv a \pmod{p} $$
für alle ganzen Zahlen $a$.

Wie kann man das nun verwenden?

Dazu müssen wir die Zahl $p$ in ein Produkt zerlegen.  Wir hatten ja gesehen, dass die Gruppe $\mathbb F_p^* \cong \mathbb Z/(p-1)$ ist, also die Ordnung $p-1$ hat.  Wenn wir jetzt
 $$ e * f \equiv p \pmod{p-1} $$
zerlegen können, dann haben wir eine *Verschlüsselung*.

Lass mich zuerst erklären, wie man damit verschlüsselt.

Wenn wir eine Primzahl, z.B. $n=29$, haben und uns einen Exponenten, z.B. $e=3$ nehmen, dann müssen wir nur einen zweiten Exponenten, z.B. $f=19$ finden, um $0\le a<p$ mittels $b:= a^e \\,\\%\\, p$ zu verschlüsseln.  Dann können wir anschließend $a=b^f \\,\\%\\, p$ wieder entschlüsseln.

Das heißt, dass wir ein asymmetrisches Verschlüsselungsverfahren haben:  Der öffentliche Schlüssel besteht aus $p$ und $e$.  Der geheime Schlüssel besteht aus $p$ und $f$.  Man kann nicht beliebig große Zahlen ver- und wieder korrekt entschlüsseln, aber wenn wir $0\le a<p$ berücksichtigen, dann können wir nach Euler-Fermat-Satz das $a$ wieder entschlüsseln.

Es ergeben sich noch 3 Fragen:

## 4.1 Wie kann man schnell Potenzieren (modulo $n$)?

Dazu kann man Binäre Exponentiation verwenden.  Die sieht etwa so aus:
```kotlin
  fun mpow(b :ULong, e :ULong, n :ULong) :ULong {
    if (e==0uL)
      return 1uL
    if (b==0uL || b==1uL)
      return b
    var result = 1uL
    var p = b % n
    var r = e
    while (r>0L) {
      if (r%2L>0L)
        result = result*p % n
      p = p*p % n;  r /= 2L
    }
    return result
  }
```

Man übergibt die Basis $b$ (z.B. unser $a$ oder die verschlüsselte Nachricht $b$), den Exponenten $e$ (oder $f$) und den Modulus $n=p$.

Die Idee dahinter ist, dass man den Exponenten, z.B. $e=3$ als Summe von 2er Potenzen schreibt, also $3=1+2$ und dann entsprechend $b^3 = b^1* b^2$ ausrechnet.  Die aufsteigenden 2er-Potenzen von $b$ erhält man, indem man von $p=b$ startet und fortlaufend quadriert, also zunächst $p=b^1$, dann $p=b^2$, dann $p=b^4=(b^2)^2$ und so weiter.  Jetzt muss man nur noch im richtigen Moment das Ergebnis mit $p$ multiplizieren.  Der richtige Momemnt ist, wenn $e\\%2^k>2^{k-1}$ ist.  Wenn wir fortlaufend $e$ durch 2 teilen, müssen wir nur jedes Mal schauen, ob der Rest von `e%2` gleich 1 (also größer als 0) ist.


## 4.2 Wie muss man das $e$ wählen?

Wir wollen $e * f \equiv p \equiv 1 \pmod{p-1}$ haben, also muss $e$ teilerfremd zu $p-1$ sein. $p$ ist eine Primzahl, oft auch ungerade (da die 2 die einzige gerade Primzahl ist und ziemlich klein).  Also darf $e$ keine gemeinsamen Teiler mit der geraden Zahl $p-1$ haben.  Dazu muss es offenbar wenigstens ungerade sein.  Man kann auch $p-1$ faktorisieren und die kleinste Primzahl wählen, die nicht in der Faktorisierung vorkommt.


## 4.3 Wie muss man $f$ wählen?

Also $e$ muss geeignet gewählt sein.  Dann brauchen wir ein modulares Inverses zu $e$, denn $e * f\equiv 1\pmod{p-1}$ bedeutet genau das.  Das kann man entweder durch probieren bestimmen (wenn $p$ nicht zu groß ist) oder indem man den Euklidischen Algorithmus modifiziert.  Im obigen Beispiel kann man kontrollieren, dass tatsächlich $e * f \\,\\%\\, (p-1) = 3 * 19 \\,\\%\\, 28 = 1$ ist.


## 4.4 Wie sieht das in einem Programm aus?

```kotlin
  fun main() {
    val n = 37uL
    val e = 3uL
    val x = n-1uL
    assert(gcd(x, e)==1uL)
    val f = 19uL
    assert(e*f%x==1uL)
    print("Verschlüsseln oder Entschlüsslen (V/E)? ")
    val choice = readlnOrNull()?.trim()
    if (choice==null||choice.isEmpty())
      return
    when (choice[0]) {
      'v', 'V' -> encrypt(n, e)
      'e', 'E' -> decrypt(n, f)
      else -> println("Der öffentliche Schlüssel ist n=$n und e=$e.")
    }
  }
```

Wenn du die Funktion `gcd`, größter gemeinsamer Teiler nicht hast, dann kannst du die 2 Zeilen mit `assert` und `gcd` weglassen/auskommentieren.  Das Programm läuft trotzdem.  Es kann nur sein, dass es Unsinn macht, wenn die Zahlen $n$, $e$ und $f$ nicht zusammen passen.

```kotlin
  fun encrypt(n :ULong, e :ULong) {
    while (true) {
      print("Bitte geben sie eine geheime Nachricht ein (ganze Zahl zwischen 0 und $n): ")
      var a :ULong?
      while (true) {
        a = readlnOrNull()?.trim()?.toUIntOrNull()
        if (a==null) {
          println("Ungültige Nachricht.")
          return
        }
        if (a>=p) {
          println("Die Nachricht ist zu groß.")
          continue
        }
        break
      }
      val b = mpow(a!!, e, n)
      println("Die Verschlüsselte Nachricht lautet $b.  Der öffentliche Schlüssel ist n=$n und e=$e.")
    }
  }
```

```kotlin
  fun decrypt(n :ULong, f :ULong) {
    while (true) {
      print("Bitte geben sie eine verschlüsselte Nachricht ein (ganze Zahl zwischen 0 und $n): ")
      var b :ULong?
      while (true) {
        b = readlnOrNull()?.trim()?.toUIntOrNull()
        if (b==null) {
          println("Ungültige Nachricht")
          return
        }
        if (b>=p) {
          println("Das kann nicht sein.")
          continue
        }
        break
      }
      val a = mpow(b!!, e, n)
      println("Die entschlüsselte Nachricht lautet $a.")
    }
  }
```

## 4.9 Praktische Fragen

### 1. Was, wenn die Nachricht aus Buchstaben besteht?

Man kann mittels ASCII-Code (oder auch Unicode) jedes Zeichen in eine Zahl übersetzen.  Beim ASCII-Code ist diese Zahl zwischen 0 und 127 (im erweiterten ASCII-Code zwischen 0 und 255) und beim Unicode wachsen die Zahlen noch, aber für europäische Sprachen reicht eine Zahl zwischen 0 und $2^{16}$, also eine 2-Byte-Nachricht.  Man müsste dann umgekehrt die entschlüsselte Zahl wieder in ein Zeichen übersetzen.  Oh, und zusätzlich braucht man auch eine genügend große Primzahl (also größer als $2^{16}$).

### 2. Was, wenn die Nachricht aus mehreren Buchstaben besteht?

Dann kann man mehrere Zahlen der Reihe nach verschlüsslen.  Dabei ist die verschlüsselte Nachricht genauso lang wie die geheime Nachricht.

### 3. Ist das auch sicher?

Gute Frage.  Wenn du in das Programm schaust, dann steht da der geheime Schlüssel, sodass jeder der das Programm hat, auch den geheimen Schlüssel kennt und die verschlüsselten Nachrichten wieder entschlüsseln kann.

Das kann man dadurch lösen, dass man die Schlüssel aus einer Datei oder einem sicheren Schlüssel-Speicher ausliest.  Dann gibt man dem Kommunikationspartner nur das Programm (ohne Schlüssel) und den öffentlichen Schlüssel.  Damit kann er dann für einen geheime Nachrichten verschlüsseln.

### 4. Wenn ich $p=n$ kenne, dann kann ich doch auch $x$ ausrechnen. Kann ich nicht aus $x$ und $e$, $f$ berechnen?

In der Tat, das ist eine der Schwachstellen dieses Prototypen.

Deshalb haben sich Rivest–Shamir–Adleman auf der Basis des Euler-Fermat-Satzes auch Gedanken gemacht, wie man das besser handhaben kann.  Das ganze nennt man den RSA-Algorithmus.  Dazu schreibe ich im nächsten Artikel mehr.


# 9. Zeit zum selber probieren

Nachdem diese Lektion ziemlich lang war, hast du jetzt Gelegenheit, das ganze erst einmal zu verdauen und mit den Beispielprogrammen selbst auszuprobieren.

Viel Spaß dabei.
