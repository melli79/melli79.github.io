Nachdem wir letztens Fraktale gemalt haben, wollen wir heute Animationen produzieren.

# Das Ziel
<video width="320" height="240" controls autoplay="true" loop>
  <source src="/movies/quadruple2.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

# 0. Programm aufsetzen
So wie bei den meisten Desktop Grafik-Apps, starten wir mit einem Hauptprogramm,
dass ein Fenster anlegt, in dem dann unsere Komponente dargestellt wird.

```kotlin
  class MyWindow(private val content :JComponent) :JFrame("Curls on the Computer"), KeyListener {
    init {
      layout = BorderLayout()
      contentPane = content
      setSize(800, 600)
      defaultCloseOperation = EXIT_ON_CLOSE
      addKeyListener(this)
      ...
    }

    ...

    override fun keyTyped(event :KeyEvent?) {
      content.reset()
    }

    override fun keyPressed(event :KeyEvent?) {}
    override fun keyReleased(event :KeyEvent?) {}
  }

  fun main() {
    val window = MyWindow(Curl())
    window.isVisible = true
  }
```

Unser Fenster soll auf Tastatureingaben reagieren, deshalb registrieren wir `MyWindow` als `KeyListener`.  Dazu müssen wir 3 Methoden implementieren `keyTyped`, `keyPressed`, `keyReleased`.  Für unsere Zwecke reicht es, in der Funktion `keyTyped` etwas auszuführen, die anderen beiden Funktionen können also leer bleiben.

## 0.1 Die Komponente zum Malen
Der eigentliche Inhalt wird in einer `JComponent` gemalt, so wie wir das bisher auch gemacht haben:
```kotlin
  class Curls() :JComponent() {
     val gamma = 1.0
     private val curls = listOf(Curl(1.0, 0.0, gama), Curl(0.0, 1.0, gamma),
        Curl(-1.0, 0.0, gamma), Curl(0.0, -1.0, gamma))
     val dt = 0.001
     val bbox = computeBoundingBox()
     private lateinit var scale :Rect
     ...

     override fun paint(g :Graphics) {
       scale = scaling(width, height)
       ...
       g.color = Color.BLACK
       curls.forEach { c -> g.drawCurl(c) }
     }

     fun Graphics.drawCurl(c :Curl) {
       val d = ...
       drawOval(scale.px(c.x) -d/2, scale.py(c.y) -d/2, d, d)
     }

     ...

     fun computeBoundingBox() :Rect {
       val c = curls.first()
       var x0 = c.x;  var x1 = c.x
       var y0 = c.yl  var y1 = c.y
       for (c in curls) {
         if (c.x<x0)  x0 = c.x
         if (x1<c.x)  x1 = c.x
         if (c.y<y0)  y0 = c.y
         if (y1<c.y)  y1 = c.y
       }
       val dx = x1-x0;  val dy = y1-y0
       return Rect(x0-dx/2, y0-dy/2, 2*dx, 2*dy)
     }

     fun scaling(width :Int, height :Int) :Rect {
       val dx = min(width/bbox.dx, height/bbox.dy)
       return Rect(bbox.x0 -(width/dx-bbox.dx)/2, bbox.y1 +(height/dx-bbox.dy), dx, -dx)
     }

     data class Rect(val x0 :Double, val y0 :Double, val dx :Double, val dy :Double) {
       val y1 = y0+dy

       fun px(x :Double) = ((x-x0)*dx).roundToInt()
       fun py(y :Double) = ((y-y0)*dy).roundToInt()
     }
  }
```

Die Wirbel (engl. curl, eddy oder vortex) werden in einer Liste gehalten.  Wir werden später die Klasse `Curl` definieren.  Im Moment soll nur gesagt werden, dass die im Laufe der Zeit über den Bildschirm wandern können.

Da nicht klar ist, wie weit die verschiedenen Wirbel auseinander liegen oder wie groß das Fenster auf dem Bildschirm ist, passen wir die Skalierung (engl. `scale` und `scaling`) dynamisch an.  Dazu müssen wir auch das umfassende Rechteck (engl. bounding box) um die Wirbel berechnen (`computeBoundingBox`).  Auch die Klasse Rechteck (engl. rectangle) sollte dir vom letzten Mal bekannt vorkommen.

Das Malen wird schließlich von der Methode `paint` ausgelöst.  Dort wird jedoch zunächst die Skala initialisiert.  Zum Malen eines Wirbels rufen wir die Methode `drawCurl` auf.  Diese erhält den `Graphics` context als Kontext und den zu malenden Wirbel als Parameter.

`drawCurl` bestimmt zunächst den optischen Durchmesser des Wirbels und rechnet anschließend die `x`- und die `y`-Koordinate des Wirbels in Bildschirmkoordinaten um.  Mit `g.fillOval(px, py, w, h)` kann man eine ausgefüllte Ellipse im Fenster malen.  Dabei ist `(px, py)` die linke obere Ecke der Ellipse und `w` die Breite (engl. width) und `h` die Höhe (engl. height).

# 1. Was ist ein Wirbel
![Wirbel, (c) Getty Images 2019](/img/vortex.jpg)
> [Wirbel, &copy; Getty Images 2019](https://www.gettyimages.de/fotos/vortex?assettype=image&page=4&phrase=vortex&sort=mostpopular&license=rf%2Crm)

Eigentlich sind Wirbel ausgedehnte 3-dimensionale Phänomene in der Athmosphäre.  Der Einfachheit halber gehen wir jedoch von geraden Wirbeln aus, die wir durch eine kleine Scheibe auf dem Bildschirm darstellen.  Interessant ist die Verwirbelung der Luft/Wolken/Objekte um den Wirbel.  Ein realer Wirbel besteht aus einem Zentrum das kompakt starr rotiert.  Im Außenbereich herrscht die typische Geschwindigkeitsverteilung für Wirbel vor:
  $$ v = \frac{\Gamma}{d} $$
Wobei $\Gamma$ (griechisch Gamma) die Stärke des Wirbels ist und $d$ der Abstand des Beobachtungspunktes vom Zentrum des Wirbels.  Das bedeutet, dass in großer Entfernung sich fast nichts bewegt, je dichter man an das Zentrum kommt, umso schneller drehen sich die Objekte um den Wirbel.

In 2 Dimensionen berechnen sich die Koordinaten der Geschwindigkeit wie folgt
  $$ \mathbf{v}=(v_x, v_y) = \frac{\Gamma}{\\|\mathbf{x}-\mathbf{x}_0\\|^2}(\mathbf{x}-\mathbf{x}_0)^\perp = \frac{\Gamma}{\Delta x^2 +\Delta y^2}(-\Delta y, \Delta x). $$

In diesem einfachen Modell wird ein Wirbel also durch folgende Daten beschrieben:  `x`- und `y`-Position und Wirbelstärke `gamma`.  Deshalb führen wir die Klasse `Curl` wie folgt ein:
```kotlin
  data class Curl(var x :Double, var y :Double, val gamma :Double) {
    fun v(p :Point2D) :Vector2D {
      val d = p.minus(Point2D(x, y))
      return d.perp()*(gamma/d.abs2())
    }

    ...
  }
```
Hier haben wir die internen Variablen `x` und `y` veränderbar gemacht, da wir die Wirbel über den Bildschirm wandern lassen wollen.

Jetzt müssen wir nur noch Punkte (engl. point) und Vektoren (so etwas wie die Geschwindigkeit) definieren:
```kotlin
  data class Point2D(val x :Double, val y :Double) {
    companion object {
      val origin = Point2D(0.0, 0.0)
    }

    operator fun minus(o :Point2D) = Vector2D(x-o.x, y-o.y)

    fun translate(v :Vector2D) = Point2D(x+v.x, y+v.y)
  }
```

```kotlin
  data class Vector2D(val x :Double, val y :Double) {
    companion object {
      val ZERO = Vector2D(0.0, 0.0)
    }

    fun abs2() = x*x +y*y
    fun perp() = Vector2D(-y, x)

    operator fun times(f :Double) = Vector2D(f*x, f*y)
  }
```
Was ist nun der Unterschied zwischen einem Vektor und einem Punkt?  Naja, der Punkt ist eine Position auf dem Bildschirm.  Es gibt einen speziellen Punkt, den Ursprung (engl. `origin`).
Ein Vektor ist so etwas wie die Differenz zwischen 2 Punkten, d.h. wenn man `p-o` berechnet, wobei `p` und `o` Punkte sind, dann ist das Ergebnis ein Vektor.  Umgekehrt kann man von einem Punkt ausgehen und diesem um einen Vektor verschieben (engl. `translate`).

Vektoren kann man auch skalieren, z.B. verdoppeln, halbieren, umkehren.  Punkte kann man nicht skalieren.  In 2D gibt es einen senkrechten (engl. perpendicular) Vector zu jedem Vektor, der zusätzlich noch die gleiche Länge hat und gegen den Uhrzeigersinn gedreht ist.  Außerdem ist der Betrag eines Vektors (engl. absolute value) als Wurzel aus $x^2 +y^2$ definiert.


# 2. Wie kommt jetzt Bewegung in das Bild?
Wenn wir das Programm aus den 5 Teilen zusammensetzen, die wir bisher geschrieben haben, dann sieht man lediglich 4 schwarze Scheiben im Fenster.

Physikalisch müssen wir die Wirbel in ihrem eigenen Feld bewegen:
```kotlin
  fun List<Curl>.evolve(dt :Double) {
    val vs = map { c ->
      val p = Point2D(c.x, c.y)
      filter { c0 -> c0!=c }
        .map { c0 -> c0.v(p) }
        .foldRight(ZERO) { s, v -> s+v }
    }
    forEachIndexed { i, c -> c.translate(vs[i]*dt) }
  }
```
D.h. dass wir zuerst die Gesamtgeschwindigkeit jedes Wirbels im Feld aller anderen Wirbel berechnen und dann jeden Wirbel mit seiner Geschwindigkeit um einen kleinen Anteil $\Delta t$ weiter bewegen.  Dazu müssen wir noch die Verschiebung des Wirbels definieren:
```Kotlin
  data class Curl(...) {
    ...

    fun translate(v :Vector2D) {
      x += v.x;  y += v.y
    }
  }
```
Damit das ganze ziemlich glatt aussieht, habe ich $\Delta t = 1$ms (also `dt=0.001`) oben festgelegt.  Allerdings reicht es nicht, wenn wir die Wirbel einmalig bewegen (engl. evolve), sondern wir müssen sie alle 1ms erneut bewegen.  Das kann man wie folgt erreichen:

```kotlin
  class Curls(...) :JComponent() {
    ...
    private var timer :Timer? =null
    ...
    override fun paint(g :Graphics) {
      ...
      if (timer==null)
        startTimer()
      ...
    }

    fun startTimer() {
      timer = Timer()
      timer.schedule(100, 10) {
        (1..10).forEach { curls.evolve(dt) }
        SwingUtilities.invokeLater {
          repaint()
        }
      }
    }
    ...
  }
```
Dabei musst du auch die Methode `schedule` importieren (aus "kotlin.coroutine"), da es sich nicht um die Java-Methode handelt.  Die 2 Zahlen bedeuten folgendes:  100ms warten wir bevor der Timer (engl. für Zeitgeber) das erste Mal zuschlägt.  Dann schlägt er alle 10ms erneut zu.  Immer wenn der Timer abgelaufen ist, dann wird der Codeschnipsel in geschweiften Klammern ausgeführt:  Wir bewegen die Wirbel 10mal (deshalb nur alle 10ms den Timer wiederholen) und anschließend geben wir den Auftrag, das Fenster zu aktualisieren.

Leider können wir das Aktualisieren (engl. repaint) nicht im Timer selbst machen.  Stattdessen geben wir den Auftrag, das im Swing-Framework zu machen.  Swing ist das Grafik-Framework, mit welchem unsere Fenster und deren Komponenten erzeugt werden.

# 3. Noch etwas Polieren
Damit der Fenstertitel den Inhalt beschreibt, habe ich die `setTitle`-Methode wie folgt abgeändert:
```kotlin
  class MyWindow(private val component :Curl) :JFrame() {
    init {
      ...
      setTitle(component.title)
    }

    override fun setTitle(detail :String) {
      super.setTitle("Curls on the Computer – $detail")
    }
  }
```

Außerdem müssen wir die Eigenschaft (engl. property) `title` noch der Komponente hinzufügen, sowie auch die Methode `reset` (engl. für Zurücksetzen).  Das kann man wie folgt machen:
```kotlin
  class Curls(private val scenario :Pair<String, List<Curl>>) :JComponent() {
    companion object {
      val gamma = 1.0
      val twoPairs = Pair("2 Pairs", listOf(Curl(...), ...))
      val quadruple = Pair("Quadruple", listOf(
        Curl(1.0, 0.0, gamma), Curl(0.0, 1.0, gamma),
        Curl(-1.0, 0.0, gamma), Curl(0.0, -1.0, gamma),
      ))
    }
    val dt = 0.001
    private var curls = scenario.second.clone()
    val title = scenario.first
    ...

    fun reset() {
      curls = scenario.second.clone()
    }
  }
```
Da wir die Wirbel im Laufe der Zeit verändern, müssen wir sie beim Zurücksetzen (engl. `reset`), klonen (engl. clone), d.h. kopieren.  Dazu müssen wir folgende Erweiterungsfunktion schreiben:
```kotlin
  fun List<Curl>.clone() = map { c -> Curl(c.x, c.y, c.gamma) }
```
Außerdem müssen wir jetzt die Anfangskonfiguration beim Erstellen der Malkomponente übergeben, etwa wie folgt:
```kotlin
  fun main() {
    val window = MyWindow(Curl(Curl.quadruple))
    ...
  }
```

Schließlich sollte ich dir noch verraten, wie man die sichtbare Größe des Wirbels auf dem Bildschirm bestimmt:  Eine einfache Lösung ist `val d=5`, d.h. unabhängig davon, wie stark der Wirbel ist, wird er immer durch eine Scheibe der Größe 5 (Pixel) dargestellt.  Alternative kann man die Größe direkt von der Stärke `gamma` abhängen lassen.  Dabei treten 2 Probleme auf:  Zum einen gibt es rechts-drehende und links-drehende Wirbel, aber der Durchmesser muss immer positiv sein, also verwenden wir `abs(gamma)` statt `gamma` direkt.  Zum anderen gibt es Wirbelstärken in ganz verschiedenen Größenordnungen, also von 0.001 über 1.0 bis 1000.0.  Deshalb sollten wir lieber den Logarithmus der absoluten Stärke verwenden `ln(abs(gamma))`.  Schließlich gibt es noch mitbewegte Beobachter, also Wirbel der Stärke 0.  Deshalb addieren wir sicherheitshalber noch eine Mindeststärke:
```kotlin
  fun Graphics.drawCurl(c :Curl) {
    val d = (ln(abs(c.gamma)+0.0001)*2+21).roundToInt()
    fillOval(...)
  }
```


# 9. Selbst Probieren
Wie sieht es aus?  Funktioniert das Programm bei dir?

Falls das Programm nicht auf Anhieb funktioniert, musst du dort schauen, wo die Fehlermeldung angezeigt wird.  Verstehst du, was der Code an dieser Stelle machen soll?  Worüber meckert der Compiler?  Wie kann man das beheben?

## 9.1 Ein weiteres Szenario
Statt 4 Wirbel, die sich im Kreis drehen, können wir auch 2 Paare von Wirbeln darstellen, die aufeinander zulaufen, etwa wie folgt:
 $$ +(-5.0, 1.0), -(-5.0, -1.0); -(5.0, 1.0), +(5.0, -1.0) $$
Die Zahlenpaare sind die Koordinaten der 4 Wirbel.  Die Vorzeichen bedeuten, dass die Wirbelstärke jeweils `±gamma` betragen soll.  Damit das Szenario angezeigt wird, musst du es unter `val twoPairs = listOf(...)` in der Klasse `Curls` eintragen und beim Zusammenbauen des Fensters übergeben `MyWindow(Curls(Curls.twoPairs))`.

## 9.2 Noch weitere Szenarien
Fällt dir noch eine andere Anordnung von Wirbeln ein, die man probieren kann?  Wie wäre es mit 8 Wirbeln auf einem Kreis angeordnet?

Viel Spaß beim Probieren.
