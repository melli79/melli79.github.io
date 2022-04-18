Nachdem letzte Woche Augen gemalt haben, wollen wir heute moderne Kunst produzieren.

# Das Ziel
![Korbstuhl, mit freundlicher Werbung für rattani.de](/img/Korbstuhl.jpg "100x150")

# 1. Projekt aufsetzen
Das geht so ähnlich wie beim letzten Projekt:  Unter Datei > Neu > Projekt ... ein Kotlin-Projekt mit Namen "Modern Art" aufsetzen mit dem gleichen JDK wie beim letzten Mal.

Dann im Hauptprogramm "Main.kt":
```kotlin
  class MainWindow(contentPane :JComponent) :JFrame("Modern Art") {
    init {
      this.layout = BorderLayout()
      this.contentPane = contentPane

      this.setSize(500, 500)
      defaultCloseOperation = EXIT_ON_CLOSE
    }
  }

  fun main(args :Array<String>) {
    val window = MainWindow(Art())
    window.isVisible = true
  }
```
Diesmal brauchen wir keinen `MouseMotionListener`, d.h. es genügt, wenn die `contentPane` als `JComponent` gekennzeichnet wird.  Wenn du den Text hinein kopiert hast, musst du eventuell die Symbole `JComponent`, `JFrame`, `BorderLayout` importieren.

Die Klasse `Art` schreiben wir als nächstes.

# 2. Ein Korbmuster Malen

Das Grundelement ist folgendes:
```kotlin
  class Art :JComponent() {
    private val n = 10

    override fun paint(g :Graphics?) {
      if (g==null) {
        super.paint(null)
        return
      }
      val a = min(width-10, height-10)
      val x0 = (width-a)/2;  val y0 = (height-a)/2

      g.color = Color.black
      val dx = a/n
      for (k in 0 until n) {
        g.drawLine(x0 +k*dx, y0+a, x0+a, y0+a -k*dx)
        ...
      }

      ...
    }
  }
```
Du kannst das Programm (ohne die "...") in die Datei "Art.kt" abtippen/kopieren.  Am besten wenn du gleich mal die Tastenkürzel vom letzten Mal übst:  im Hauptprogramm den Textcursor auf `Art()` stellen und \<Alt\>+\<Enter\> drücken, dann "Klasse erstellen" und "in separater Datei".  Immer wenn du ein Symbol getippt hast, das rot erscheint, musst du es wahrscheinlich importieren, wieder \<Alt\>+\<Enter\> drücken und z.B. `JComponent` aus dem Packet "javax.swing" importieren.

Die `paint`-Methode erstellst du, indem du das Wort "overri" tippst und dann aus dem Menü `override fun paint(g :Graphics?) {}` auswählst.

## Was bedeutet das?

Die Erweiterung `: JComponent()` sollte dir vom letzten Mal bekannt vorkommen.  Das ist eine Grafikkomponente, die man in einem Fenster (`JFrame` im Hauptprogramm) anzeigen kann.

`override fun paint(g :Graphics?)` kennst du bestimmt auch vom letzten Mal.  Hier wird das Bild gemalt.

`if (g==null) { ... return }` bewirkt, dass die Malroutine abbricht, falls kein Graphics-Objekt zum Malen übergeben wird.

`val a = min(width-10, height-10)` ist die Kantenlänge des Quadrates (zunächst nur 1 Quadrant).  Das wird so groß wie möglich gewählt aber höchstens Breite (engl. width) minus 10 Pixel und Höhe (engl. height) minus 10 Pixel.

`val x0;  val y0` sind die Anfangskoordinaten der linken oberen Ecke vom Quadrat.  Die `-10` oben bewirken, dass wir immer einen Rand von mindestens 5 Pixeln auf jeder Seite lassen.

`g.color = Color.black` malt die folgenden Objekte in Schwarz (engl. black).

`val dx = a/n` bedeutet, dass wir jede weitere Linine im Abstand 1/n der Kantenlänge des Quadrats anfangen.

`for (k in 0 until n)` bedeutet, dass wir eine Schleife n-mal durchlaufen (weiter oben steht, dass `n=10`), wobei k die Werte 0, 1, 2, ... bis ausschließlich $n$ hat.

`g.drawLine(x0, y0, x1, y1)` malt eine Strecke von $(x_0, y_0)$ nach $(x_1, y_1)$, Punkte werden in $x$-$y$-Koordinaten angegeben.  $x$ ist der Abstand vom linken Rand des Fensters, $y$ ist der Abstand vom oberen Rand des Fensters. $ (x_0, y_0) \leftarrow (x_0+k*\Delta x, y_0+a) $ bedeutet, dass alle Linien in der gleichen Höhe (am unteren Ende vom Quadrat) anfangen, aber jeweils um $ \Delta x $ versetzt. $ (x_1,y_1) \leftarrow (x_0+a, y_0 +a -k*\Delta x) $ bedeutet, dass die Linien alle am rechten Rand des Quadrates aufhören, aber in verschiedenen Höhen, unten angefangen (für $k=0$ ist $y_1=y_0+a$, ...).

Probier's gleich mal aus (\<Strg\>+\<R\>).  Immerhin sollte er jetzt schon mal 10 Linien im Korbmuster malen.


## 4-mal versetzt

Wir können das noch verschönern.  Vielleicht können wir das gleiche Muster 4-mal malen, einmal in jeder Ecke, etwa so:
anstelle der ersten "..." schreiben wir:
```Kotlin
  g.drawLine(x0+a, y0+a -k*dx, x0 +a-k*dx, y0)
  g.drawLine(x0 +a-k*dx, y0, x0, y0 +a-k*dx)
  g.drawLine(x0, y0 +a-k*dx, x0 +k*dx, y0 +a-k*dx)
```

Offenbar werden jetzt 4-mal so viele Strecken gemalt.  Aber was sind das für Koordinaten (und wie kommt man darauf)?

Man erkennt, dass immer die letzten 2 Koordinaten gleich den ersten 2 Koordinaten der nächsten Zeile sind, d.h. es wird in jedem Schleifendurchlauf ein 4-Eck gemalt (ohne Absetzen).

Die zweite Linie beginnt also am rechten Rand und geht bis zum oberen Rand.  Die dritte Linie beginnt am oberen Rand und geht bis zum linken Rand.  Die vierte Linie schließlich beginnt am linken Rand und geht bis zum unteren Rand, dort wo die erste Linie anfing.


# 3. Die Mitte noch etwas auffüllen

Erinnerst du dich vom letzten Mal noch, wie wir gefüllte Ovale gemalt haben: `g.fillOval(x0, y0, d, d)`.  So ähnlich kann man auch Ellipsen (also die Ränder von Ovalen) malen:

```Kotlin
  g.color = Color.blue.darker()
  val d = (sqrt(0.5)*a).toInt()
  val vx = (a-d)/2
  g.drawOval(x0+vx, y0+vx, d, d)
```
Das fügst du einfach anstelle der zweiten "..." ein (hinter der for-Schleife).  Wenn du es startest, dann malt es einen, sehr dünnen, blauen Kreis.  Um den Kreis dicker zu malen, kann man wie folgt vorgehen.  Ersetze die 1 Zeile `g.drawOval(...)` durch folgende 4 Zeilen:
```Kotlin
g.drawOval(x0+vx-1, y0+vx-1, d, d)
g.drawOval(x0+vx+1, y0+vx-1, d, d)
g.drawOval(x0+vx-1, y0+vx+1, d, d)
g.drawOval(x0+vx+1, y0+vx+1, d, d)
```
Das malt eigentlich 4 Kreise, aber so dicht beieinander, dass die wie ein dick gemalter Kreis aussehen.  Jetzt sollte das Blau erkennbar sein.

## Den Kreis noch füllen
können wir mit folgenden Zeilen:
```Kotlin
  g.color = Color.green.darker()
  val c = sqrt(3.0)/2;  val s = 0.5
  val x = 0.5;  val y = 0.0
  for (k in 1..6) {
    g.drawLine(x0+vx+(d*(0.5-x)).toInt(), y0+vx+(d*(0.5-y)).toInt(),
      x0+vx+(d*(0.5+x)).toInt(), y0+vx+(d*(0.5+y)).toInt())
    val x1 = x
    x = c*x -s*y
    y = c*y +s*x1
  }
```
Das kommt hinter das Malen des Kreises.  Die Zeilen bedeuten das Folgende:

$(c, s)$ sind die Koordinaten eines Punktes im Winkel von 30º zur Waagerechten. $(x, y)$ sind jeweils die Punkte auf einem Kreis mit Radius $0.5*d$.

`g.drawLine(x0+vx+(d*(0.5-x)).toInt(), y0+vx+(d*(0.5-y)).toInt(),
 x0+vx+(d*(0.5+x)).toInt(), y0+vy+(d*(0.5+y)).toInt())` malt eine Strecke durch den Mittelpunkt des obigen Kreises beim ersten Durchlauf waagerecht ($x=0.5$, $y=0$) und dann das ganze noch 5 weitere Male mit veränderten Koordinaten.

 $$ {x \choose y} \leftarrow {c*x +s*y\choose c*y +s*x} $$ dreht die Punkte auf dem Kreis um 30º weiter, sodass man nach 6 Durchläufen bei ($x=-0.5$, $y=0$) ankommt.

 Wenn du das Programm fehlerfrei abgetippt hast, sollte es jetzt ausführbar sein.  Das ergibt ein Korbmuster außen im Quadrat und einen blauen Kreis mit einem grünen Stern in der Mitte.

 Funktioniert es alles bei dir?


 # 9. Selbst ausprobieren

 Kannst du rotierende Linien malen, die von $(0.0, 0.5)$ nach $(1.0, 0.0)$ und dann in Schritten von 30º gedreht verlaufen?

 Am besten legst du dir dazu 4 Variablen an:
 ```Kotlin
   var x1 = ...;  var y1 = ...
   var x2 = ...;  var y2 = ...
```
die du mit den Startwerten belegst.  Dann eine Schleife, die 12mal durchlaufen wird und innerhalb der Schleife malst du eine Strecke von $(x_0+a/2+x_1, y_0+a/2+y_1)$ nach $(x_0+a/2+x_2, y_0+a/2+y_2)$ und drehst die $(x_i, y_i)$ mit den $(c,s)$ von oben weiter.

Viel Spaß beim Probieren und sag Bescheid, wenn es Probleme/Fragen gibt.
