Nachdem wir letzte Wochen das Mandelbrot und den Baum des Pythagoras gemalt haben, wollen
wir heute weitere Fraktale produzieren.

# Das Ziel
![Iteriertes Funktionensystem](/img/ifs.png)

# 0. Programm aufsetzen
Das kennst du schon:  Ein neues Kotlin-Projekt anlegen und das Hauptprogramm
ausfüllen, etwa so:

```kotlin
  class MyWindow(content :JComponent) :JFrame("Iteriertes Funktionensystem") {
    init {
       layout = BorderLayout()
       contentPane = content
       defaultCloseOperation = EXIT_ON_CLOSE
       setSize(800, 600)
    }
  }

  fun main() {
     val window = MyWindow(IFS(System.currentTimeMillis()))
     window.isVisible = true
  }
```

## 0.1 Komponente zum Malen
Auch diesen Teil solltest du schon einmal gesehen haben.  Wir fangen etwa so an:

```kotlin

  class IFS(val seed :Long) :JComponent() {

    val random = Random(seed)
    val n = 3
    val fs :List<Aff>
    val ps :List<Double>
    val size :Rect
    private lateinit var scale :Rect
    private var p = Point2D.origin

    init {
      fs = (1..n).map {
        var f = random.nextAff()
        ...
        f
      }
      ps = ...
      (1..10_000).forEach {
        p = iterate(p)
      }
      size = bbox()
    }

    fun iterate(p :Point2D) :Point2D {
      ...
    }

    override fun paint(g :Graphics?) {
      g ?: return
      scale = scaling(size, width, height)
      (1..width*height/10).forEach {
        p = iterate(p)
        g.drawRect(scale.px(p), scale.py(p), 1, 1)
      }
    }

    fun scaling(size :Rect, width :Int, height :Int) :Rect {
      val dx = min(width/size.dx, height/size.dy)
      return Rect(size.x0-(size.dx-dx/width)/2, size.y1+(size.dy-dx/height)/2, dx, -dx)
    }

    data class Rect(val x0 :Double, val y0 :Double, val dx :Double, val dy :Double) {
      val y1 = y0+dy

      fun px(p :Point) = ((p.x-x0)*dx).roundToInt()
      fun py(p :Point) = ((p.y-y0)*dy).roundToInt()
    }

    ...

    data class Point2D(val x :Double, val y :Double) {
      companion object {
        val origin = Point2D(0.0, 0.0)
      }
    }
  }
```

Also die Hälfte davon sollte dir bekannt vorkommen.  `Rect` stellt ein (axenparalleles) Rechteck dar, mit dem sowohl die wahre Größe des Fraktals `size` (engl. für Größe) als auch die Umrechnung von virtuellen Koordinaten in Bildschirmkoordinaten erfolgt (`px` und `py`).

Mit `paint` wird wieder etwas gemalt.  Das Malen besteht darin, dass wir 10% der Anzahl maximal möglicher Punkte im Fenster berechnen (einer nach dem anderen) und diese dann im Fenster als kleine Quadrate eingezeichnet werden.  D.h. dass wir für größeres Fenster mehr Punkte berechnen (und bei kleinerem Fenster uns die Arbeit sparen).

Interessant ist, dass wir die Klasse selbst noch initialisieren müssen:  Das geschieht in der Methode `init` (Achtung: ohne Klammern `()`).  Zunächst rechnen wir eine handvoll Abbildungen aus, dann ein paar Wahrscheinlichkeiten `ps`, anschließend initialisieren wir den Punkt `p`, indem wir 10'000 mal iterieren.  Am Schluss bestimmen wir noch die Größe (`bbox` ist englisch für bounding box und heißt umspannendes Rechteck).


# 1. Lineare/Affine Abbildungen
Iteriertes Funktionensystem heißt, dass wir eine hanvoll Abbildungen der Ebene auf sich selbst haben, die dann in zufälliger Reihenfolge auf einen Punkt angewendet werden.  Wenn der Punkt im Fraktal liegt (z.B. nach ca. 10'000 Anfangs-Iterationen), dann hüpft er zufällig durch das ganze Fraktal.  Man kann das Fraktal erkennen, wenn man den Weg des Punktes einzeichnet.

## Affine Abblidungen
Im einfachsten Fall entscheiden wir uns dafür, affine/lineare Abbildungen zu verwenden.  Wenn wir den Punkt in der Ebene mit 2 Koordinaten $p=(x, y)$ beschreiben, dann ist eine affine Abbildung eine affine Kombination dieser 2 Koordinaten:
  $$ (x,y) \mapsto (a_{00}x+a_{01}y+b_0, a_{10}x+a_{11}y+b_1) $$

In der Schule nennt man dies manchmal lineare Abbildung, da der 1-dimensionale Fall $x\mapsto ax+b$ eine Gerade darstellt (Anstieg $a$ und kreuzt die y-Achse bei $y=b$), aber das ist nicht richtig, weil linear eigentlich heißt, dass $f(x+x')=f(x)+f(x')$ und $f(cx)=c\\,f(x)$.  Das wäre nur für $b=0$ erfüllt.

D.h. eine affine Abbildung hat 6 Parameter und man muss diese 2 Formeln verwenden.  Das sieht in Kotlin etwa so aus:

```kotlin
  class IFS(...) :JComponent() {
    ...
    data class Aff(val a00 :Double, val a01 :Double, val a10 :Double, val a11 :Double,
        val b0 :Double, val b1 :Double) {
      companion object {
        val id = Aff(1.0,0.0,0.0,1.0, 0.0,0.0)
        fun translate(vx :Double, vy :Double) = Aff(1.0, 0.0,0.0, 1.0, vx, vy)
        private fun rotate(c :Double, s :Double) = Aff(c,-s, s,c, 0.0, 0.0)
        fun rotate(alpha :Double) = rotate(cos(alpha), sin(alpha))
        val flipX = Aff(-1.0,0.0,0.0,1.0, 0.0,0.0)
        val flipY = Aff(1.0,0.0,0.0,-1.0, 0.0,0.0)
      }

      val det = a00*a11 -a01*a10

      fun map(p :Point2D) = Point2D(a00*p.x+a01*p.y+b0, a10*p.x+a11*p.y+b1)

      fun chain(b :Aff) = Aff(a00*b.a00+a01*b.a10, a00*b.a10+a01*b.a11,
          a10*b.a00+a11*b.a10, a11*b.a11,
          b0 +a00*b.b0+a01*b.b1, b1 +a10*b.b0+a11*b.b1)
    }

    fun Random.nextAff() = Aff(2.2*nextDouble()-1.1, 2.2*nextDouble()-1.1,
      2.2*nextDouble()-1.1, 2.2*nextDouble()-1.1,
      20*nextDouble()-10, 20*nextDouble()-10)

    ...
  }
```

Ich habe noch 2 weitere Operationen programmiert:  Zum einen `chain`, eine Verkettung zweier solcher Abbildungen und zum anderen `det` die Determinante der Abbildung.  Die Determinante gibt an, wie groß das Bild wird (im Vergleich zum Original), d.h. eine Determinante von $>1$ bedeutet, dass die Abbildung (insgesamt) vergrößert, eine Determinante zwischen 0 und 1, dass sie verkleinert, eine Determinante $<0$, dass sie umkehrt und eine Determinante von $1$,
dass sie insgesamt in Originalgröße abbildet.  Dabei kann es aber sein, dass sie in manchen Richtungen verkleinert und dafür in anderen Richtungen vergrößert.

Die einfachste Abbildung ist $1\\!\\!1=$`Aff.id`, die Identitätsabbildung, die jeden Punkt auf sich selbst abbildet.  Die nächsteinfachen sind die Verschiebungen (engl. translations), für die man eine Verschiebung in $x$- und in $y$-Richtung angibt.  Etwas trickreicher sind die Drehungen `rotate(alpha)`, die alle Punkte um den Ursprung $(0, 0)$ (engl. `origin`) um einen Winkel $\alpha$ drehen.

Trickreicher sind auch die Spiegelungen `flipX` und `flipY`, die jeweils die $x$- oder die $y$-Achse umkehren, die jeweils andere Achse aber so belassen.  Die letzten beiden haben beide Determinante -1, alle anderen Determinante 1.

Es gibt jedoch auch noch andere affine Abbildungen, die aber hier nicht alle einzeln explizit konstruiert werden sollen.  Stattdessen gibt es eine Funktion `nextAff()` mit der man eine zufällige affine Transformation erzeugen kann.  Dazu braucht man lediglich einen Zufallsgenerator, den wir am Anfang der Klasse (`val random = Random(seed)`, engl. für Zufall) angelegt haben.  Da der Computer keinen echten Zufall erzeugen kann, gibt man einen Startwert (engl. `seed` Futter/Samen) vor und von dort an springt der Generator chaotisch durch die ganzen Zahlen.

Der Ausdruck `random.nextDouble()` liefert eine zufällige reelle Zahl zwishen 0 und 1.  Wenn man die mit 2.2 multipliziert und 1.1 abzieht, erhält man eine zufällige reelle Zahl zwischen -1.1 und 1.1.  Entsprechend funktioniert das mit `20*random.nextDouble()-10`.


# 2. Fraktale erzeugen
Wenn man statt einer affinen Abbildung gleich 2 oder 3 verwendet, dann kann man ein Fraktal erhalten.  Dazu muss aber jede Abbildung kontrahierend sein $-1<\mathrm{det}<1$.  Man kann das Fraktal rekonstruieren, indem man zufällig einige dieser Transformationen anwendet.  Um die Fläche des Fraktals gleichmäßig abzulaufen, sollte man die Transformation $A$ mit einer Wahrscheinlichkeit $p(A)\sim|\det A|$ wählen.  Insgesamt müssen sich alle Wahrscheinlichkeiten zu 1 addieren.  Das kann man so erreichen:

```kotlin
  class IFS :JComponent() {
    ...
    init {
      fs = (1..n).map {
        var f = random.nextAff()
        while (abs(f.det)>=1.0)
          f = random.nextAff()
        f
      }
      val ps = fs.map { f -> abs(f.det) }
      val f = 1.0/ps.sum()
      this.ps = ps.map { p -> p*f }.cumsum()
    }

    fun iterate(pt :Point2D) :Point2D {
      val p = random.nextDouble()
      val k = ps.firstOrNull { p1 -> p<=p1 } ?: fs.size-1
      return fs[k].map(pt)
    }
    ...
  }

  fun Iterable<Double>.cumSum() :List<Double> {
    var sum = 0.0
    return map { v -> sum+=v; sum }
  }
```

Wie gesagt, wir müssen sicherstellen, dass jede affine Abbildung $f$ eine Determinante vom Betrag (engl. absolute value) $< 1$ hat.  Dann weisen wir zunächst jeder Abbildung den Betrag der Determinante zu und schließlich teilen wir noch durch die Summe der Determinanten und bilden die kumulativen Summen, d.h. wenn die Determinanten $0.9$, $0.6$ und $-0.5$ sind, dann wird zunächst $[0.9, 0.6, 0.5]$ daraus, was sich zu $2.0$ summiert, also ist der Skalierungsfaktor $1/2.0=0.5$ und somit auf $[0.45, 0.3, 0.25]$ abgebildet und anschließend zu $[0.45, 0.75, 1.0]$ kumuliert.  Die Höhe der jeweiligen Stufen ist proportional zum Betrag der jeweiligen Determinante und die Gesamthöhe ist 1.  Offenbar ist das nicht Bestandteil der Kotlin Standard-Bibliothek.  Deshalb steht am Ende, wie man das für reelle Zahlen berechnet.

Die Iteration wählt nun eine zufällige affine Abbildung aus, indem zunächst eine Zahl zwischen 0 und 1 gewählt wird (gleichverteilt) und anschließend die entsprechende Stufe gefunden wird.  Als letzten Schritt muss man nur noch den Punkt `pt` mit dieser affinen Abbildung abbilden (engl. `map`).


# 3. Größe des Fraktals abschätzen
Das ist komplizierter als es erst aussieht:  Leider kann man aus den 2--3 Transformationen nicht einfach abschätzen, wie groß das Fraktal am Ende wird.  Stattdessen kann man jedoch einfach einige Punkte des Fraktals ausrechnen und schauen, bis wohin die wachsen.  Das kann man etwa so tun:

```kotlin
  ...
  fun bbox() :Rect {
    var x0 = p.x;  var x1 = p.x
    var y0 = p.y;  var y1 = p.y
    (1..10_000).forEach {
      p = iterate(p)
      if (p.x<x0)  x0 = p.x
      if (x1<p.x)  x1 = p.x
      if (p.y<y0)  y0 = p.y
      if (y1<p.y)  y1 = p.y
    }
    val dx = x1-x0;  val dy = y1-y0
    return Rect(x0-0.05*dx, y0-0.05*dy, 1.1*dx, 1.1*dy)
  }
```

Wir berechnen einfach 10'000 Punkte und schauen für jeden von diesen, ob die x-Koordinate kleiner als $x_0$ ist, dann wird $x_0$ aktualisiert, ob $x_1$ kleiner als die x-Koordinate ist, dann wird $x_1$ aktualisiert, dann nochmal das Entsprechende für die y-Koordinate und $y_0$ und $y_1$.  Das Ergebnins ist, dass $x_0$ die kleinstmögliche x-Koordinate der 10'000 Punkte ist, $x_1$ die größtmögliche, $y_0$ die kleinstmögliche y-Koordinate und $y_1$ die größtmögliche.  Damit ist auch klar, dass das umspannende Rechteck die Breite $\Delta x = x_1-x_0$ und die Höhe $\Delta y = y_1-y_0$ hat.  Tatsächlich sollten wir aber nicht zu knapp schätzen, da ein paar wenige Punkte später noch außerhalb dieser Grenzen liegen können.  Also geben wir lieber 10% in x- und in y-Richtung dazu ($1.1\Delta x$, $1.1\Delta y$).  Damit die Mitte in der Mitte bleibt, müssen wir den linken Punkt um $0.05\Delta x$ nach links und den unteren Punkt um $0.05\Delta y$ nach unten verschieben.

# 9. Selber Probieren
So wie das Programm geschrieben ist, erzeugt es jedes Mal ein zufälliges anderes Fraktal.  Wenn du immer das gleiche Fraktal erzeugen willst, musst du den Zufallsgenerator immer gleich initialisieren, z.B. so:
```kotlin
  val window = MyWindow(IFS(42L))
```

Was musst du am Programm ändern, wenn es 4 affine Transformationen statt 3 verwenden soll?  Wie kannst du die Farbe der Punkte von schwarz auf blau (engl. `blue`) ändern?

Wie kann man das Programm dynamisch machen?  Z.B. alle 1s (`Thread.sleep(1000)` wartet 1s) das Iterierte Funktionssystem neu initialisieren (mit einem anderen Wert) und dann das Fenster nochmal neu sichtbar machen.

Viel Spaß beim Probieren!
