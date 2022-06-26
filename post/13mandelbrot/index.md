Nachdem wir letzte Woche Korbmuster gemalt haben, wollen wir heute weitere mathematische Kunst produzieren.

# Das Ziel
<iframe width="560" height="315" src="https://www.youtube.com/embed/b005iHf8Z3g" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# 0. Programm Aufsetzen

Das kennst du vielleicht schon:  Du legst ein neues Kotlin/JVM-Projekt an und schreibst das Hauptprogramm etwa so:
```kotlin
  class MainWindow(contentPane :JComponent) :JFrame("Mandelbrot") {
    init {
      this.layout = BorderLayout()
      this.contentPane = contentPane

      this.setSize(500, 500)
      defaultCloseOperation = EXIT_ON_CLOSE
    }
  }

  fun main(args :Array<String>) {
    val window = MainWindow(Mandelbrot())
    window.isVisible = true
  }
```
Danach musst du wieder die zahlreichen Symbole importieren: `JComponent`, `JFrame`, `BorderLayout`, ...; jeweils mit \<Alt\>+\<Enter\> und dort import aus einem Unterpaket von `javax.swing` wählen.

# 1. Mandelbrot definieren
Wie malt man nun ein Mandelbrot?

Der Kern ist folgende Iteration:

$$ z_{n+1} = z_n^2+c $$

D.h. man wählt einen komplexen Parameter $c=(c_x, c_y)$ und einen Startpunkt $z_0=(0,0)$ und berechnet dann in komplexen Koordinaten das Quadrat vom letzten $z_n$ und addiert $c=(c_x,c_y)$.

Wenn du Glück hast, erzählt dir der Mathelehrer im Abitur, was komplexe Zahlen sind.  Für unsere Zwecke reicht es zu wisssen, dass die komplexe Zahl $z_n=(x,y)$ aus 2 reellen (lies gewöhnlichen) Zahlen besteht.  Die addiert man dann komponentenweise, also so
 $$ z+c = (x+c_x, y+c_y). $$

Multiplizieren ist etwas komplizierter, denn es soll ja $zw$ nur 0 sein, wenn $z$ oder $w$ null ist.  Dazu merkt man sich die Regel $\mathsf{i}^2=-1$ und $1=(1,0)$ und $\mathsf{i}=(0,1)$, also
 $$ zw = xw_x + \mathsf{i}(xw_y+yw_x) +\mathsf{i}^2yw_y = (xw_x-yw_y, xw_y+yw_x). $$

Für das Quadrat bedeutet das
 $$ z^2 = (x^2-y^2, 2xy). $$

Bei der Implementierung müssen wir noch aufpassen, dass wir für $z_{n+1}= z_n^2+c$ an allen Stellen die alten Werte für $z_n=(x,y)$ einsetzen, etwa so
```kotlin
  var x,y, cx,cy = ...
  val x1 = x*x-y*y + cx
  y = 2*x*y + cy
  x = x1
```

## 1.2 Was bedeutet eine Iteration?
Die obigen Formeln sagen, dass man $z_1= z_0^2+c$ ausrechnen kann.  Dann $z_2=z_1^2+c$, $z_3=z_2^2+c$ und so weiter.  Das heißt, dass man eine ganze Folge von (komplexen) Zahlen bekommt: $z_0$, $z_1$, $z_2$, $z_3$, ...

Leider kann man nicht unendlich viele Zahlen malen (oder auch nur ausrechnen).  Stattdessen hat Bernoît Mandelbrot herausgefunden, dass man ein interessantes Muster bekommt, wenn man zu _jeder_ Zahl $c=(c_x,c_y)$ einträgt, ob die Folge beschränkt bleibt.

Im 2-dimensionalen ($z=(x,y)$) sind die Schranken Kreise um den Ursprung, also $r=\sqrt{x^2+y^2}$ und die Erfahrung lehrt, dass man etwa bis $r\le2$ gehen muss.  Um die Berechnung der Quadratwurzel zu sparen, bildet man einfach $r^2=x^2+y^2$ und vergleicht $r^2\le4$.


# 2. Wie malt man das?
## 2.1 Wie entscheidet man “beschränkt bleibt”?

Wie gesagt, die Erfahrung lehrt, dass man auf $r^2\le4$ testen muss.  Aber man kann auch nicht unendlich viele Folgenelemente ausrechnen.  Stattdessen entscheidet man sich für eine Grenze `MAX_COLORS=256` und rechnet maximal 256 Folgenelemente aus und schaut, dass jeweils $r^2=x^2+y^2\le4$ bleibt.  Wenn das bis `MAX_COLORS` so bleibt, dann gehört der Punkt $c=(c_x, c_y)$ zum Mandelbrot und wird schwarz (engl. `BLACK`) gezeichnet.

## 2.2 Wie bekommt man Farben?

In der Beispielanimation wurde nicht nur schwarz-weiß gemalt, sondern man hat die Punkte am Rand auch farbig gemalt.  Die Farben kann man etwa so bestimmen:  Wenn bereits der erste Punkt ($z_1=c=(c_x, c_y)$) außerhalb von $r\le2$ liegt, dann wird er mit Farbe 1 gemalt (z.B. blau), wenn der erste Punkt drinne, aber der zweite Punkt $z_2=(x,y)$ außerhalb liegt, dann wird $(c_x, c_y)$ mit Farbe 2 gemalt (z.B. grün), und so weiter.

Was wir also brauchen, ist folgende Funktion:
```kotlin
  val MAX_COLORS = 256

  fun iterate(x0 :Double, y0 :Double, cx :Double, cy :Double) :UInt {
    var x=x0;  var y=y0
    var x2 = x*x;  var y2 = y*y
    for (color in 1..MAX_COLORS) {
      if (x2+y2>4)
        return color.toUInt()
      y = 2*x*y +cy
      x = x2-y2 +cx
      x2 = x*x;  y2 = y*y
    }
    return 0u
  }
```

### Das sind doch aber Zahlen, keine Farben!

Na dann muss man halt beim Malen noch die Zahlen in Farben umwandeln, z.B. so

```kotlin
  fun colorize(color :UInt) :Color = when (color) {
    0u -> Color.BLACK
    else -> Color(color%256u, 128, 255 -color%256u)
  }
```

Dabei bedeutet `0u -> Color.BLACK`, dass 0 in die Farbe Schwarz (engl. black) umgewandelt wird.
`color%256u` bedeutet, dass der Farbwert modulo 256 berechnet wird, also $1\mapsto1$, $2\mapsto2$, ..., $256\mapsto0$, $257\mapsto1$, ...  `Color(v,128,255-v)` bedeutet, dass die Farben von $(1,128,254)$ (hellblau) über $(128,128,128)$ (grau) bis $(255,128,0)$ (hellrot) verlaufen.  Das liegt daran, dass man dem Computer eine beliebige Farbe im Rot-Grün-Blau-Anteilen gibt, also `Color(255,0,0)` ist Rot, `Color(0,255,0)` ist Grün, `Color(128,128,128)` ist Grau, `Color(255,0,255)` ist Lila, `Color(255,255,0)` ist Gelb, `Color(255,255,255)` ist Weiß und so weiter.


## 2.3 Und wie bekomme ich jetzt ein ganzes Bild?

Dazu bedienen wir uns eines Tricks aus dem Fernsehen: statt alle Punkte zugleich auszurechnen, laufen wir durch das Bild von links-oben bis rechts-unten zeilenweise und berechnen für jeden Punkt die Farbe und zeichnen sie ein, etwa so:

```kotlin
  val base = Rect.of(-2.0, -4.0/3.0, 0.5, 4.0/3.0)
  private lateinit var scale :Rect

  fun paint(g :Graphics?) {
    g ?: return
    scale = scaling(width, height)
    var cy = scale.toY(0)
    for (py in 0 until height) {
      var cx = scale.toX(0)
      for (px in 0 until width) {
        val c = iterate(0.0, 0.0, cx, cy)
        g.setColor(colorize(c))
        g.fillRect(px, py, px+1, py+1)
        cx += scale.dx
      }
      cy += scale.dy
    }
  }
```
`for (py in 0 until height)` ist wieder eine for-Schleife für die Variable `py`.  Die Schleife beginnt bei 0, dann 1, ..., geht aber bis (engl. until) ausschließlich `height`, d.h. der letzte Wert ist `height-1`.  Das liegt daran, dass das Fenster `height` Punkte hoch ist, die aber von 0 bis `height-1` reichen.

Entsprechend ist das bei `px`.

Am Ende jeder Schleife steht noch `cx += scale.dx` (oder für `cy` entsprechend).  Das brauchen wir, um auch $c=(c_x, c_y)$ auf den nächsten Punkt zu setzen.  Vor jeder Schleife definieren wir `var cx = scale.toX(0)`, d.h. wir rechnen den linken (oberen) Rand in die entsprechende Koordinate um.


## 2.4 Was ist ein `Rect`?

Das kommt vom englischen Wort rectangle, also Rechteck.  Ein Rechteck besteht aus 4 Punkten und wenn es parallel zu den Achsen ist, kann man es mit 1 Punkt `(x0, y0)` und Breite `dx` und Höhe `dy` ausdrücken.  In Kotlin etwa so:

```kotlin
  data class Rect(val x0 :Double, val y0 :Double, val dx :Double, val dy :Double) {
    companion object {
      fun of(val x0 :Double, val y0 :Double, val x1 :Double, val y1 :Double) = Rect(x0, y0, x1-x0, y1-y0)
    }
    ...
  }
```


### a) Wie rechnet man die Bildschirm-Koordinaten in komplexe Zahlen um?

Das hängt von der aktuellen Bildschirmskalierung (engl. `scale`) ab.  Im Prinzip lauten die Formeln:
```kotlin
  data class Rect(...) {
    ...
    fun toX(px :Int) = x0 +px*dx
    fun toY(py :Int) = y0 +py*dy
  }
```

Jetzt muss man nur noch die aktuelle Skala ausrechnen.  Dazu können wir die aktuelle Fenstergröße abfragen `(width, height)`.  Wichtig ist jetzt, dass das Mandelbrot immer im gleichen Seitenverhältnis abgebildet wird, d.h. wir müssen am Ende $|dx|=|dy|$ wählen.  Die Basisgröße (engl. `base`) des Mandelbrots ist $(-2, -1.2)$--$(0.5, 1.2)$.  Prinzipiell wollen wir die Skalen `dx/width` und `dy/height` verwenden.  Wenn die aber verschieden sind, dann müssen wir für beide Achsen die größere Skala (sic!) verwenden, also
```kotlin
  val dx = max(base.dx/width, base.dy/height)
  ...
  return Rect(x0, y0, dx, -dx)
```

Tatsächlich müssen wir die y-Skala umdrehen, weil auf dem Bildschirm die Achse nach unten zeigt, aber im mathematischen Koordinatensystem die y-Achse nach oben zeigen soll, also `-dx`.


### b) Wie kommen wir an die Randpunkte `(x0, y0)` heran?

Im Prinzip wollen wir `base.x0` und `base.y0` verwenden, aber dabei gibt es 2 Probleme:

1. Wenn `base.dy/height > base.dx/width` ist, dann müssen wir links (und rechts) noch etwas auffüllen, also `val x0 = base.x0 -(dx*width-base.dx)/2`

2. Da die y-Achse eine negative Skala hat (`dy < 0`), muss man hier `y1 = base.y0 +base.dy` verwenden und entsprechend das Offset addieren, also `val y0 = base.y1 +(dx*height-base.dy)/2`

Die Formeln sind nicht ganz einfach und am besten überprüft man sie noch einmal, z.B. indem man schaut, ob das richtige Mandelbrot auf dem Bildschirm ist (oder etwa nur eine blaue Fläche).

Insgesamt sieht die Funktion so aus:
```kotlin
  fun scaling(width :Int, height :Int) :Rect {
    val dx = max(base.dx/width, base.dy/height)
    val x0 = base.x0 -(dx*width-base.dx)/2
    val y0 = base.y1 +(dx*height-base.dy)/2
    return Rect(x0, y0, dx, -dx)
  }

  data class Rect(...) {
    ...
    val y1 = y0+dy
  }
```

# 9. Selber Probieren
Jetzt bist du dran.  Wenn du das ganze Programm fehlerfrei abgetippt hast, dann sollte es starten und ein Mandelbrot (der Anfang von obigem Video) malen.

## 9.0 Funktioniert das Programm bei dir?

## 9.1 Kannst du den Farbverlauf ändern?

Aktuell wird in Lilatönen gemalt.  Was musst du (wie) ändern, damit die Farben von Rot über Gelb bis Grün verlaufen?

Hinweis:  Du musst die Funktion `colorize(color :UInt)` ändern.


## 9.2 Wie kann man einen Ausschnitt vom Mandelbrot malen?

Hinweis: `val base = Rect.of(...)` bestimmt diesen Ausschnitt.

Wie kann man auf die Kugel links am Mandelbrot zoomen?

Viel Spaß beim Probieren.  Im Zweifelsfall einfach die alten Werte wieder herstellen, dann sollte das original Mandelbrot (in Lilatönen) erscheinen.
