Nachdem wir letztes Mal mit Primzahlen und dem kleinen Satz von Fermat verschlüsselt haben, wollen wir heute die Grundlagen des RSA Algorithmus kennen lernen.

# 1. Potenzen modulo $n$, noch einmal

Erinnern wir uns noch einmal an die Potenzen der Reste modulo $n$.  Beim letzten Mal hatten wir dazu folgende Tabellen produziert:

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

sowie

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

Im Vergleich zum letzten Mal fällt uns jetzt vielleicht auch das Folgende auf:

1. $a^6\equiv 1\pmod{7}$ solange $a\not\equiv0\pmod{7}$.
2. $a^7\equiv a\pmod{7}$.
3. $a^3\equiv a\pmod{6}$.

Stimmt das für alle Module?

Leider nein, wenn wir uns an die Tabelle für 9 erinnern, sehen wir, dass

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

Aber:

4. solange $a$ teilerfremd zu $9$ ist, gilt $a^6\equiv 1\pmod{9}$ und auch $a^7\equiv a\pmod{9}$.

Insgesamt kann man das mit folgendem Satz und Corollar beschreiben:

Satz (Euler-Fermat): Wenn $a$ teilefremd zu $n$ ist, dann gilt
 $$ a^{\phi(n)}\equiv 1 \pmod{n}.$$

Dabei ist offenbar für eine Primzahl $p$, $\phi(p)=p-1$ und $\phi(6)=2$ und $\phi(9)=6$.  Euler hat auch eine allgemeine Vorschrift für diese Funktion $\phi$ gefunden, die man ihm zu Ehren Eulersche phi-Funktion nennt.[^1] Ich erläutere die gleich im nächsten Abschnitt.

Corollar (Euler-Fermat): Wenn $n$ (eine Primzahl oder) das Produkt verschiedener Primzahlen ist, dann gilt für jedes $a$
 $$ a^{\phi(n)+1} \equiv a \pmod{n}. $$

Zur Erinnerung nochmal:  Für jede Primzahl $p$ gilt
  $$ a^p \equiv a \pmod{p}. $$

[^1]: Man spricht das $\phi$ phi [fi:] aus.


# 2. Zerlegung in Primfaktoren und Eulersche $\phi$-Funktion

Proposition (Euler):  Gegeben eine positive ganze Zahl $n$ und ihre Faktorisierung in Primzahlen
  $$ n = p_1^{e_1}p_2^{e_2}\dots p_k^{e_k} $$
dann ist
  $$ \phi(n) = (p_1-1)p_1^{e_1-1}(p_2-1)p_2^{e_2-1}\dots(p_k-1)p_k^{e_k-1}. $$

Stimmt das?

Ok, wenn man eine Primzahl $p$ hat, dann hat die keine Teiler (außer 1 und $p$), d.h. $\phi(p)=(p-1)p^0= p-1$.  Das kommt schon mal hin.

$6=2 * 3=2^1 * 3^1$, also $\phi(6)=(2-1) * 2^0 * (3-1) * 3^0=2$ und wir hatten gesehen, dass der Satz und das Corollar mit $e=2$, bzw. $e=3$ funktioniert.

## 2.1 Was bedeutet $\phi(n)$?

Dazu betrachten wir nochmal die ganzen Zahlen modulo $n$ unter Addition:

```log
  Vielfache modulo 6:
   0  0  0  0  0  0
   0  1  2  3  4  5
   0  2  4  2  4  2
   0  3  0  3  0  3
   0  4  2  0  4  2
   0  5  4  3  2  1
```
D.h. nur die Vielfache von 1 und von 5 durchlaufen alle Restklassen modulo 6.  Das sind 2 und $\phi(6)=2$.

Bei einer Primzahl, z.B. 5 sieht es einfacher aus:

```log
  Vielfache modulo 5:
   0  0  0  0  0
   0  1  2  3  4
   0  2  4  1  3
   0  3  1  4  2
   0  4  3  2  1
```

D.h. alle Reste außer 0 erzeugen die Gruppe $\mathbb Z/(p)$.  Wenn man das mit $\phi(p)=p-1$ vergleicht, sieht man, dass auch hier $\phi$ die Anzahl der Generatoren von $\mathbb Z/(n)$ berechnet.

Definition:  Gegeben eine positive ganze Zahl $n$, dann ist $\phi(n)$ die Anzahl der teilerfremden Zahlen zwischen 0 und $n$.

Offenbar können Reste $a$, die nicht teilerfremd zu $n$ sind, nicht die ganze Gruppe $\mathbb Z/(n)$ erzeugen, sondern nur die Vielfache vom größten gemeinsamen Teiler zwischen $a$ und $n$.

Im Beispiel von $a=2$ und $n=6$ ist das 2 und tatsächlich sind die Vielfache von 2 alle Vielfache von 2 (trivialer Weise).  Für $a=4$ und $n=6$ ist der größte gemeinsame Teiler auch 2.  In der obigen Tabelle modulo 6 sehen wir, dass die Vielfache von 4 auch alle Vielfache von 2 durchlaufen.

Wenn also $a$ und $n$ teilerfremd sind, dann ist der größte gemeinsame Teiler 1 und somit durchlaufen die Vielfache von $a$ alle Reste modulo $n$.  Damit haben wir einen Generator der Gruppe $\mathbb Z/(n)$.

Die Formel von Euler erhält man nun durch einfaches Durchzählen der Vielfache der Primteiler von $n$.


## 2.2 Wie berechnet man die Primfaktor-Zerlegung einer positiven ganzen Zahl?

Bevor wir die berechnen, sollten wir vielleicht sicherstellen, dass die überhaupt eindeutig ist.

Dazu erinnern wir uns an die Definition von Primzahl:  Eine positive ganze Zahl $p$ heißt Primzahl, wenn sie genau 2 Teiler (1 und $p$) hat.

Offenbar kann man die Primzahlen eindeutig als Produkt von Primzahlen darstellen (nämlich sich selbst).

Als nächstes brauchen wir eine interessante Eigenschaft:

Lemma (Primzahl):  Ein Produkt $n=a*b$ ist genau dann durch eine Primzahl $p$ teilbar, wenn $a$ durch $p$ teilbar ist oder wenn $b$ durch $p$ teilbar ist (oder beides).

Jetzt müssen wir nur noch fragen, ob jede positive ganze Zahl als Produkt von Primzahlen geschrieben werden kann (oder ob es vielleicht unendlich viele Faktoren gibt).  Das ist nicht ganz offensichtlich, wenn wir bedenken, dass etwa $1=1 * 1=1 * 1 * 1=1 * 1 * 1 * 1=\dots$ ist.  Aber 1 ist *keine* Primzahl, sondern die Einheit.  Die 1 ist als Produkt aus 0 Primzahlen darstellbar.  Ein leeres Produkt hat den Wert 1 (das neutrale Element der Multiplikation), per Konvention.

Für alle Primzahlen gilt aber, dass sie größer als 1 sind.  Außerdem gilt für ein Produkt aus 2 Zahlen, die jede größer als 1 ist, dass das Produkt größer als jeder der Faktoren ist.

Das bedeutet, wenn wir eine Zahl $n$ in ein Produkt aus einer Primzahl $p$ und einem 2. Faktor $f$ zerlegt haben, dann ist $f$ kleiner als $n$ und wir können $f$ weiter zerlegen und werden in endlich vielen Schritten so zu einer Primzahl (oder zu 1) kommen.

In einem Programm sieht das wie folgt aus:

```kotlin
  fun factor(n :ULong) {
    var f = n
    var p = 2uL
    while (f>=p*p) {
      while (f%p==0uL) {
        print("$p, ")
        f /= p
      }
      if (p==2uL)
        p = 3uL
      else
        p += 2uL
    }
    if (f>1uL)
      println("f")
  }
```

Wenn wir also das Programm mit 2023 aufrufen
```kotlin
  fun main() {
    factor(2023uL)
  }
```

dann erhalten wir:
```log
  7, 17, 17,
```

Mit einem Taschenrechner kannst du nachprüfen, dass das tatsächlich stimmt.

Frage:  Eigentlich wird $p$ auch irgendwann 9 (zumindest, wenn dann $f$ noch größer als 9 ist), warum wird 9 nie als Primfaktor ausgegeben?

Für die Berechnung der Funktion $\phi$ brauchen wir diesen Algorithmus in einer leicht abgewandelten Form:

```kotlin
  fun factor(n :ULong) :Map<ULong, UByte> {
    if (n<=1uL)
      return emptyMap()
    val result = mutableMapOf<ULong, UByte>()
    var f = n
    var p = 2uL
    while (p*p<=f) {
      var e = 0
      while (f%p==0uL) {
        e++
        f /= p
      }
      if (e>0)
        result[p] = e.toUByte()
      if (p==2uL)
        p = 3uL
      else
        p += 2uL
    }
    if (f>1uL)
      result[f] = 1u
    return result
  }
```
Statt also die Primfaktoren auszugeben, zählen wir in `e` wie oft der aktuelle Primfaktor `p` vorkommt.  Wenn sich `f` nich weiter durch `p` teilen lässt, aber wenigstens 1 Faktor `p` vorkam, speichern wir unter `p` den Wert `e` ab.

### A. Was ist eine `Map`?

Englisch map heißt Karte oder Abbildung.  Gemeint ist hier das letztere, d.h. eine Abblidung oder Zuordnung.  `Map<ULong, UByte>` heißt, dass großen positiven ganzen Zahlen (`ULong`) kleine positive ganze Zahlen zugeordnet werden.  Wenn wir mit einer großen Zahl wie `65537` anfangen, dann können die Primfaktoren selbst auch groß werden.  Die Anzahl der Primfaktoren bleibt aber moderat.

Beispiel:  Wir hatten mit dem letzten Algorithmus gesehen, dass `2023: 7, 17, 17` ist.  In diesem Beispiel bedeutet dies,  dass das Ergebnis von `factor` die Zurodnung `[7:1, 17:2]` ergibt, also 1 Faktor 7 und 2 Faktoren 17.

Um die Funktion besser zu verstehen, können wir sie in folgendem Programm einbinden:

```kotlin
  fun main() {
    print("Bitte geben Sie eine positive ganze Zahl ein: ")
    var n = readlnOrNull()?.trim()?.toULongOrNull()
    if (n==null || n<1uL)
      println("Keine gültige Eingabe.")
      return
    }
    val factors = factor(n)
    println("$n = "+ factors.entries.joinToString("*"){ (p, e) -> "$p^$e" })
  }
```

Wenn wir also am Prompt 65537 eingeben, erhalten wir

```log
  Bitte geben Sie eine positive ganze Zahl ein: 65537
  35537 = 65537^1
```
Das bedeutet, dass 65537 eine Primzahl ist.

Probier das Programm doch gleich mal mit anderen Zahlen aus, z.B. 65535 oder 131072 oder ...

## 2.3 Wie berechnet man nun $\phi$?

Das können wir mit der obigen Faktorisierung und der Formel für $\phi$ tun:

```kotlin
  fun phi(n :ULong) :ULong {
    if (n<=1uL)
      return 0uL
    val factors = factor(n)
    return factors.map { p, e -> (p-1uL)*ipow(p, e-1u) }.fold(1uL){ p, f -> p*f }
  }
```

Dazu brauchen wir noch eine Funktion, um Potenzen ganzer Zahlen zu berechnen (nicht modulo n).  Diese Funktion habe ich `ipow` genannt (integer powers) und kann analog zu `mpow` mit binärer Exponentiation berechnet werden:

```kotlin
  fun ipow(b :ULong, e :UByte) :ULong {
    if (e==0u)
      return 1uL
    if (b<=1uL)
      return b
    var result = 1uL
    var p = b
    var r = e.toInt()
    while (r>0) {
      if (r%2>0)
        result *= p
      p *= p
      r /= 2
    }
    return result
  }
```

### Was bedeutet `map {...}` und `fold(n){ p, f -> p*f }`?

Das englische Wort fold bedeutet falten.  Wenn du dich erinnerst, dann ist `factors` eine Map mit den Primfaktoren (und ihren Exponenten) von `n`.  Mit `.map {...}` werden diese Primfaktoren nun zu Produkten umgewandelt.  Wie in Eulers Formel angegeben, muss man $p-1$ mit $p^{e-1}$ multiplizieren.

Das Ergebnis von `factors.map { ... }` ist also eine Liste von Produkten, also positiven ganzen Zahlen.  Die müssen wir jetzt noch in (beliebiger) Reihenfolge miteinander multiplizieren.  Genau das macht `.fold(1uL){ p, f -> p*f }`.  Wir fangen mit $1$ an und immer wenn wir ein Teilprodukt `p` haben und einen weiteren Faktor `f`, dann multiplizieren wir diese.  Das Ergebnis am Ende ist das große Produkt, welches wir in Eulers Formel brauchen.

Mit Falten ist in diesem Fall also gemeint, dass eine Liste von Zahlen zu einer einzigen Zahl zusammengefaltet wird.  In userem Beispiel entsteht das Produkt.  Man könnte die Liste von Zahlen aber genausogut zur Summe zusammenfalten.

Am besten probierst du auch dieses Programm mal aus:

```kotlin
  fun main() {
    print("Eulersche phi-Funktion.\n  Bitte geben Sie eine positive ganze Zahl ein: ")
    val n = readlnOrNull()?.trim().?toULongOrNull()
    if (n==null || n<=1uL) {
      println("Ungültige Eingabe.")
      return
    }
    val factors = factor(n)
    println("$n = "+ factors.entries.joinToString("*"){ (p, e) -> "$p^$e" })
    println("phi($n) = ${phi(n)}")
  }
```

Der Unterschied zwischen `"phi($n)"` und `"${phi(n)}"` ist, dass das erstere das Wort "phi" ausgibt und die Zahl $n$ in Klammern, während das letztere die Funktion `phi(n)` aufruft und das Ergebnis ausgibt.  Man kann also in Strings auch rechnen.


# 3. RSA Algorithmus

Wann kommen wir endlich zum RSA-Algorithmus?

Ok, damit können wir jetzt anfangen.  Wie bereits letztes Mal geschrieben, ist dieser Algorithmus nach R. Rivest, A. Shamir und A. Adleman benannt.  Die Idee ist, den Satz von Euler-Fermat (bzw. sein Corollar) zu verwenden.

Letztens hatten wir bereits gesehen, wie man mit 1 Primzahl ver- und entschlüsseln kann, indem man eine Lösung $e * f \equiv1\pmod{\phi(n)}$ findet für die Primzahl $n=p$ und dann $0\le a<n$ mit $a^e\\,\\%\\,n$ verschlüsselt und eine verschlüsselte Zahl $b$ mittels $b^f\\,\\%\\,n$ wieder entschlüsselt.

Wenn man das obige Corollar versteht, dann kann man statt $n=p$ auch einfach $n=p_1 * p_2$ verwenden.  Alles, was wir brauchen, ist $\phi(n)$ zu berechnen und eine Lösung zu $e * f\equiv1\pmod{\phi(n)}$ zu finden.

## 3.1 Wie kann man die Gleichung $e * f\equiv1\pmod{n}$ lösen?

Es gibt eine Einschränkung.  Wenn es zu einem $e$ ein solches $f$ geben soll, dann muss $e$ teilerfremd zu $n$ sein.

Wie kann man den größten gemeinsamen Teiler zweier Zahlen finden?

Dazu hat Euklid von Alexandria einen interessanten Algorithmus gefunden.  Betrachten wir diesen zunächst an einem Beispiel:  Sei $a=12$ und $b=18$.  Wir wollen den größten gemeinsamen Teiler von $a$ und $b$ finden.

### Was ist ein größter gemeinsamer Teiler?

Betrachten wir zunächst alle Teiler von $a=12$ und $b=18$:

| $a_k$ | Teiler            |
|-------|-------------------|
| 12    | 1, 2, 3, 4, 6, 12 |
| 18    | 1, 2, 3, 6, 9, 18 |

Die gemeinsamen Teiler von 12 und 18 sind offenbar 1, 2, 3 und 6 und der größte von diesen ist 6.  Also sollte das Ergebnis 6 sein.

Aber man muss dazu nicht erst alle Teiler von $a$ und von $b$ durchprobieren.  Stattdessen kann man auch folgendes tun:

0. Fangen wir mit $a_0=b=18$ und $a_1=a=12$ an.

1. Wir teilen $a_0$ durch $a_1$ und bestimmen den Rest der Division $a_2:= a_0\\,\\%\\,a_1$ -- hier also $a_2=6$

2. dann fahren wir fort mit $a_1$ und $a_2$ bis irgendwann der Rest 0 wird -- also $a_3 := a_1\\,\\%\\,a_2 = 12 \\,\\%\\,6 = 0$.

Die letzte Zahl vor der 0 ist der größte gemeinsame Teiler, hier also ggT(12, 18) = $a_2=6$.

In Kotlin kann man den wie folgt berechnen:

```kotlin
  fun gcd(a :Long, b :Long) :Long {
    if (a==0L)
      return b
    var a0 = max(abs(a),abs(b));  var a1 = min(abs(a),abs(b))
    while (a1>0) {
      val a2 = a0 % a1
      a0 = a1;  a1 = a2
    }
    return a0
  }
```

Wenn du das mit folgendem Hauptprogramm aufrufst
```kotlin
  fun main() {
    print("Größter gemeinsamer Teiler\n  Bitte geben Sie 2 ganze Zahlen ein: ")
    val input = readln().trim().split("[\\s,]+".toRegex())
    if (input.size!=2) {
      println("Das waren keine 2 Zahlen.")
      return
    }
    val a = input[0].toLongOrNull();  val b = input[1].toLongOrNull()
    if (a==null) {
      println("Das erste ist keine ganze Zahl.")
      return
    }
    if (b==null) {
      println("Das zweite ist keine ganze Zahl.")
      return
    }
    val d = gcd(a, b)
    println("Der größte gemeinsame Teiler von $a und $b ist: gcd($a, $b) = $d")
  }
```

und hier z.B. "12 18" eingibst, erhältst du:
```log
  Größter gemeinsamer Teiler
    Bitte geben Sie 2 ganze Zahlen ein: 12 18
    Der größte gemeinsame Teiler von 12 und 18 ist: gcd(12, 18) = 6
```

Probier's doch mal mit 24 und 36 aus.  Was passiert, wenn eine der Zahlen 0 ist?  Was passiert, wenn beide Zahlen 0 sind?

Ok, wir können jetzt also überprüfen, ob unsere Wahl von $e$ sinnvoll ist, indem wir den größten gemeinsamen Teiler zwischen $e$ und $\phi(n)$ ausrechnen und überprüfen, dass der 1 ist.

### Wie kann man $f$ bestimmen?

Aber mit dem Euklidischen Algorithmus kann man noch mehr: Wenn $a_2 = a_0 \\,\\%\\, a_1$ ist, dann können wir auch $a_2$ als Kombination aus $a_0$ und $a_1$ schreiben, etwa so:

1. $f_0=1$, $f_1=0$ -- also $a_0=f_0a_0 +f_1a_1$
2. $g_0=0$, $g_1=1$ -- also $a_1 = g_0a_0 +g_1a_1$,
3. $d_2 = a_0/a_1$ und $a_2 = a_0 \\,\\%\\, a_1$ -- also $a_0 = d_2a_1+a_2$ oder $a_2 = a_0 -d_2a_1$
4. $h_0=f_0 -d_2g_0$ und $h_1 = f_1 -d_2g_1$ -- also auch $a_2 = h_0a_0 +h_1a_1$
5. und so weiter, bis irgendwann $a_k=0$ ist.

Dann sind die vorletzten Koeffizienten die gesuchten Faktoren.

In Kotlin sieht das wie folgt aus:
```kotlin
  fun euclid(a :Long, b :Long) :Triple<Long, Long, Long> {
    var a0 = a;  var f0 = 1L;  var f1 = 0L
    var a1 = b;  var g0 = 0L;  var g1 = 1L
    while (a1!=0) {
      val d2 = a0/a1;  val a2 = a0 -d*a1
      val h0 = f0 -d*g0;  val h1 = f1 -d*g1
      a0 = a1;  f0 = g0;  f1 = g1
      a1 = a2;  g0 = h0;  g1 = h1
    }
    return Triple(f0, f1, a0)
  }
```

Sicherheitshalber solltest du dieses Programm testen, da man sich leicht vertippen kann, es aber vielleicht trotzdem kompiliert.  Das letzte Hauptprogramm musst du dazu wie folgt abändern:
```kotlin
  fun main() {
    print("Euklidischer Algorithmus\n  Bitte geben Sie 2 ganze Zahlen ein: ")
    val input = readln()...
    if (...) {...}
    val a = input[0].toLongOrNull();  val b = input[1].toLongOrNull()
    if (...) {...}
    val fs = euclid(a, b)
    println("ggT($a, $b) = ${fs.third} = ${fs.first}*$a +${fs.second}*$b")
  }
```

Wenn du das mit 18 und 12 ausprobiertst, sollte
```log
  Euklidischer Algorithmus
    Bitte geben Sie 2 ganze Zahlen ein: 18 12
  ggT(18, 12) = 6 = 1*18 +-1*12
```
Immerhin nicht ganz falsch.  (Die Zeichen "+-1*" muss man als "-1*" lesen.)  Sicherheitshalber solltest du es auch noch mit "12 18" probieren (es sollte genauso 6 herauskommen).

Damit kann man jetzt das modulare Inverse ausrechnen, wie folgt:
```kotlin
  fun genPrivateKey(p1 :ULong, p2 :ULong, e :ULong) :Pair<ULong, ULong> {
    val n = p1*p2
    val x = (p1-1uL)*(p2-1uL)
    val fs = euclid(x.toLong(), e.toLong())
    assert(fs.third==1L){ "Der Exponent $e ist nicht teilerfremd zu phi(n)" }
    val f = fs.second.toULong()
    return Pair(n, f)
  }
```

Das rufst du mit folgendem Hauptprogramm auf:
```kotlin
  fun main() {
    print("Erzeugung eines RSA Schlüsselpaares\n  Bitte geben Sie 2 verschiedene Primzahlen ein: ")
    val input = readline().trim().split("[\\s,]+".toRegex())
    assert(input.size==2){ "Das waren keine 2 ganzen Zahlen." }
    val p1 = input[0].toULongOrNull();  val p2 = input[1].toULongOrNull()
    if (p1==null || p1<2uL || p1%2==0uL) {
      println("$p1 ist keine Primzahl.")
      return
    }
    if (p2==null || p2<2uL || p2%2==0uL || p2%p1!!==0uL || p1%p2==0uL) {
      println("$p2 ist keine Primzahl.")
      return
    }
    assert(p1!=p2){ "p1 und p2 müssen verschieden sein." }
    var e = 3uL
    while (e<p1&&e<p2) {
      try {
        val n, f = genPrivateKey(p1, p2, e)
        println("Ein öffentlicher Schlüssel ist n= $n, e= $e.  Der Private Schlüssel dazu ist f= $f.")
        return
      } catch(_ :AssertError) {
        e += 2uL
      }
    }
    println("Erzeugung eines Schlüsselpaares leider nicht möglich.  Vielleicht sind $p1 und $p2 keine Primzahlen?")
  }
```

Zum Testen brauchst du nur noch 2 Primzahlen, z.B. 29 und 37.
```log
Erzeugung eines RSA Schlüsselpaares
  Bitte geben Sie 2 verschiedene Primzahlen ein: 29 37
  Der öffentliche Schlüssel ist n= 1073, e= 3.  Der private Schlüssel ist f= 358
```

Diese Zahlen kannst du nun in das Programm vom letzten Mal einfügen und erhältst ein Ver- & Entschlüsselungsprogramm:
```kotlin
  fun main() {
    val n = 1073uL
    val e = 3uL
    val f = 358uL
    print("RSA Ver- & Entschlüsselung\n  Öffentlicher Schlüssel: n= $n, e= $e\n  Wollen sie Ver- oder Entschlüsseln (v/e)? ")
    val choice = readlnOrNull()?.trim()
    if (choice.isNullOrBlank())
      return
    when (choice[0]) {
      'v', 'V' -> encrypt(n, e)
      'e', 'E' -> decrypt(n, f)
      else -> println("unbekannte Wahl.")
    }
  }
```

# 4. Weiterführende Fragen

## 4.1 Woher bekommt man Primzahlen?

Zu Testzwecken kann man kleine Primzahlen verwenden.  Die kann man aus dem Sieb des Erathostenes gewinnen, so wie wir das beim vorletzten Mal getan haben.

Zur sicheren Anwendung braucht man aber große und hinreichend verschiedene Primzahlen.  Dazu muss man sich Gedanken machen, wie man große (wahrscheinliche) Primzahlen erzeugt und wie man von einer großen Zahl testet, ob es eine Primzahl ist.

Bei ersterem kann Euklids Beweis helfen, denn Euklid zeigte, dass es unendlich viele Primzahlen gibt:

Nehmen wir an, es gäbe nur endlich viele Primzahlen.  Diese zählen wir als $p_1, p_2, \ldots, p_n$ auf.  Dann bilden wir das Produkt $P = p_1 * p_2 * \dotsm * p_n$ und addieren 1.  Wir behaupten, dass $P+1$ durch keine der Primzahlen teilbar ist, denn bei jeder der bekannten Primzahlen lässt es den Rest 1.  Andererseits wissen wir, dass sich $P+1$ in Primzahlen zerlegen lässt.  Also ist entweder $P+1$ selbst eine Primzahl oder es gibt eine weitere Primzahl $p_{n+1}$, sodass $P+1$ durch $p_{n+1}$ teilbar ist.

Heuristisch kann man also ein handvoll Primzahlen nehmen, diese multiplizieren und 1 addieren.  Dann hat man entweder bereits eine Primzahl oder man muss noch kleinere Faktoren abspalten, um eine weitere Primzahl zu erhalten.  Da außer 2 alle Primzahlen ungerade sind, ist es sinnvoll, das Produkt aus 2 und einer handvoll ungerader Primzahlen zu bilden, denn dann ist es gerade und der Nachfolger ungerade.  Damit hat der Nachfolger eine Chance, eine Primzahl zu sein (oder zumindest nicht durch 2 teilbar zu sein).

Man sollte aber unbedingt testen, ob die erhaltene Zahl wirklich eine Primzahl ist.  Man bedenke, dass die Zahl recht groß sein wird (und hoffentlich nicht durch 2, 3, ... teilbar).  Dazu eignet sich der [Pollard-Rho-Test](https://de.wikipedia.org/wiki/Pollard-Rho-Methode).  Leider ist das nur ein heuristischer Test, d.h. wenn es eine Primzahl ist, dann wird das nicht widerlegt, aber falls es keine Primzahl ist, dann kann es sein, dass er trotzdem kein Gegenbeispiel findet.  Wenn man das Verfahren aber mehrfach wiederholt, kann man ziemlich sicher sein, dass es sich um eine Primzahl handelt (oder hat einen Faktor gefunden).

Es gibt auch inoffizielle Wettbewerbe, [große Primzahlen zu finden](https://www.zeit.de/1996/21/prim.txt.19960517.xml/komplettansicht).

## 4.2 Wie kann man sicher mit diesem Algorithmus umgehen?

Dazu sollte man den Algorithmus von den Schlüsseln getrennt aufbewahren.  Der Algorithmus und seine Implementation ist öffentlich zugänglich.  Ein Schlüsselpaar kann man auf dem eigenen Rechner erzeugen.  Dann darf man nur den öffentlichen Schlüssel herausgeben (den privaten Schlüssel immer gut unter Verschluss halten).

So, wie bei dem Prototyp vom letzten Mal kann man auch längere Texte verschlüsseln, indem man die Zeichen zunächst mittels UTF-8 in Zahlen umwandelt und die Zahlen der Reihe nach verschlüsselt.  Wichtig ist dabei auch, dass man keine Spuren hinterlässt, also nicht etwa die geheime Nachricht irgendwo herum liegen lässt.

## 4.3 Wie sicher ist der Algorithmus wirklich?

Wenn man einen Super-Computer hat (oder zu kleine Primzahlen verwendet), dann kann man die Verschlüsselung knacken.  Das einzige was man braucht, ist der öffentliche Schlüssel, also $n$ und $e$ und genügend Rechenzeit, um $n$ zu faktorisieren.

Das liegt daran, dass wir aus $\phi(n)$ und $e$ leicht den privaten Schlüssel $f$ ausrechnen können.  Zur Berechnung der Eulerschen phi-Funktion benötigen wir aber die Primfaktor-Zerlegung von $n$.

Mit einem Laptop kann man Zahlen bis etwa 14 Stellen innerhalb einer halben Stunde faktorisieren.  Das  geht zwar nicht mit dem einfachen Faktorisierungsalgorithmus vom Anfang, aber zusammen mit der Pollard-Rho-Methode kann man das schaffen.  Man muss also wesentlich größere Primzahlen verwenden.

2020 wurde zu wissenschaftlichen Zwecken ein [829bit RSA-Schlüssel innerhalb von ein paar Monaten geknackt](https://en.wikipedia.org/wiki/RSA_Factoring_Challenge).  $2^{829}\approx 10^{249}$ also eine 124- und eine 125-stellige Primzahl.

Heutzutage verwendet man RSA-Schlüssel von mindestens 2048 Bit, besser 4096 Bit.

Leider gibt es noch ein paar andere Angriffsmöglichkeiten.  Wenn man etwa zwei nah beieinander liegende Primzahlen verwendet, dann kann man einfach die Wurzel aus $n$ ziehen und nach Faktoren von $n$ in der Nähe dieser Wurzel suchen.  Andererseits ist klar, dass die Faktorisierung von $n$ nur so schwer ist, wie das Finden des kleinsten Faktors.  D.h. man verwendet am besten zwei ähnlich große Primfaktoren, z.B. einer 1023 Stellen und einer 1024 Stellen, dann ist deren Produkt etwa 2048 Stellen groß (Ok, eigentlich braucht man nur Stellen im 2er-System).

## 4.9  Ich habe einen PGP/GPG-Schlüssel erzeugt, aber der enthält gar nicht 2 Zahlen

Ja, auch das obige Programm ist nur ein Prototyp.  Tatsächlich kann man sich das Speichern des öffentlichen Schlüssels vereinfachen, wenn man für $e$ immer eine Primzahl verwendet, von der man sicher ist, dass $\phi(n)$ zu ihr teilerfremd ist.  Eine solche Primzahl wäre etwa 65537.  Dann muss der öffentliche Schlüssel nur aus 1 Zahl, nämlich $n$ bestehen.

Der private Schlüssel ist oft länger als der öffentliche Schlüssel, das kann daran liegen, dass dort eben neben $f$ auch der Modulus $n$ gespeichert ist.

Schließlich werden die Zahlen nicht im Dezimalsystem gespeichert, sondern eventuell im Hexadezimalsystem, also zur Basis 16, weil der Computer mit 2er-Potenzen leichter rechnen kann ($2^4=16$).  Die typischen Ziffern für das Hexadezimalsystem sind 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F.

Auch ist es klug, wenn man in der Datei mit dem Schlüssel vermerkt, dass es ein Schlüssel ist, und vielleicht noch eine Prüfsumme, ...

Für die praktische Anwendung empfiehlt sich der [Gnu Privacy Guard](https://gnupg.org/), eine Open-Source-Implementierung des RSA-Algorithmus.

# 9. Selber probieren

Ich hoffe, dass du nicht nur den Text gelesen hast, sondern auch die Programme in die IDE kopiert hast und ausprobiert hast.  Falls du das noch nicht getan hast, wäre jetzt eine gute Gelegenheit, das auszuprobieren.

Du kannst auch mal im Internet suchen, wie man GPG auf deinem Rechner installiert und verwendet.  Viele eMail-Programme lassen sich heute mit RSA-Verschlüsselung verwenden.

Viel Spaß beim Probieren.
