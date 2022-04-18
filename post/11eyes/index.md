Nachdem ihr eine [Entwicklungsumgebung aufgesetzt](../00-setup-ide/) habt, wollen wir heute etwas malen.

# Das Ziel
![(.)(.)](/img/eyes.png "100x50")

# 1. Projekt aufsetzen

So, wie in der Anleitung im Schritt 2B angegeben, erstellt ihr eine Datei > Neu > Projekt ... vom Typ Kotlin, Console Application.  Ihr könnt dem Projekt einen ordentlichen Namen geben, z.B. Eyes (engl. für Augen), achtet darauf, dass das Build-System Gradle-Kotlin ist, das Projekt open JDK 11 (oder was ihr damals als Java-Umgebung installiert habt) verwendet.  Der Rest kann bei den Standardwerten bleiben.

Nach einer Weile öffnet die IDE (Entwicklungsumgebung) die Datei "Main.kt" mit einem minimalen Hauptprogramm ```fun main(args :Array<String) {...}```.  Dieses ändert ihr jetzt wie folgt ab:

```Kotlin
class MainWindow(contentPane :Eyes) :JFrame("Eyes") {
    init {
        layout = BorderLayout()
        this.contentPane = contentPane

        setSize(600, 400)
        defaultCloseOperation = EXIT_ON_CLOSE
        // hier kommt noch etwas
    }
}

fun main(args :Array<String>) {
    val window = MainWindow(Eyes())
    window.isVisible = true
}
```

Wenn ihr das eingefügt/eingegeben habt, sollten `:Eyes`, `BorderLayout` und `JFrame` rot markiert sein.  Das liegt daran, dass diese noch importiert/implementiert werden müssen.  Stellt den Cursor auf `JFrame` und drückt \<Alt\>+\<Enter\>:  Es sollte die Zeile
```Kotlin
  import javax.swing.JFrame
```
ganz oben eingefügt werden.  (Falls keine Zeile eingefügt werden kann, habt ihr vielleicht nicht "Konsolen-Applikation" ausgewählt oder noch kein JDK, dann müsstet ihr das Projekt nochmal aufsetzen/anpassen.)

Entsprechend stellt ihr euch auf `BorderLayout` und drückt nochmal \<Alt\>+\<Enter\>.  Dabei sollte die Zeile
```Kotlin
  import java.awt.BorderLayout
```
eingefügt werden (über der Zeile mit `JFrame`).

## Bedeutung

Was bedeuten diese Zeilen?  Es wird ein Fenster (in der Größe 600x400 Pixel) erzeugt und ein Objekt vom Typ `Eyes` als Inhalt (engl. content pane) gesetzt.  Leider funktioniert das Programm noch nicht, weil die Klasse `Eyes` noch nicht implementiert ist. (Notfalls könnt ihr `contentPane :Eyes` und `this.contentPane = ...` entfernen, dann sollte das Programm kompilieren und starten mit \<Strg\>+\<R\>.)


# 2. Augen Malen

Wie malt man nun etwas in das Fenster?

Dazu muss man eine eigene Klasse schreiben, die die Klasse `JComponent` erweitert.

Das erreicht ihr, indem ihr im Hauprprogramm auf `Eyes` zeigt und \<Alt\>+\<Enter\> drückt.  Dabei sollte ein Menü erscheinen, in dem ihr "Klasse erstellen" und dann "in eigener Datei" auswählt.  Danach sollte ein weiteres Editor-Tab aufgehen mit Namen "Eyes.kt" und Inhalt:
```Kotlin
  class Eyes {

  }
```
Dort ergänzt ihr jetzt zu
```Kotlin
  class Eyes :JComponent() {

  }
```
zeigt auf `JComponent` und drückt erneut \<Alt\>+\<Enter\>.  Dann sollte die Zeile
```Kotlin
  import javax.swing.JComponent
```
oben eingefügt werden.

Jetzt könnt ihr das Programm compilieren und ausführen (auch wenn ihr vorher nicht `contentPane :Eyes` entfernt habt).  Leider sieht man noch immer keine Augen.

Das bringen wir jetzt dem Computer bei:  In den Leer-Raum zwischen `{` und `}` stellt ihr den Cursor und tippt `overri`.  Dabei sollte ein Kontextmenü aufgehen, in dem ihr `override fun paint(g :Graphics?)` sucht und \<Enter\> drückt.  Das ist die Methode, die das Bild malen soll.  Das Grundgerüst (welches eingefügt wird) sieht etwa so aus:
```Kotlin
  override fun paint(g :Graphics?) {
    super.paint(g)
  }
```
Falls `Graphics` rot dargestellt ist, müsstet ihr darauf zeigen und \<Alt\>+\<Enter\> drücken.  Spätestens dann wird die Zeile
```Kotlin
  import java.awt.Graphics
```
oben eingefügt.  (Falls es mehrere Optionen zur Wahl gibt, müsstet ihr `Graphics` im Paket "java.awt" wählen).

Jetzt ändert ihr die Methode wie folgt ab:
```Kotlin
  override fun paint(g :Graphics?) {
    if (g==null) {
      super.paint(null)
      return
    }
    val w = width;  val h = height
    g.color = Color.white
    g.fillOval(0, 0, w/2-5, h)
    g.fillOval(w/2+5, 0, w/2-5, h)

    g.color = Color.black
    val d = min((0.275*w).toInt(), (0.35*h).toInt())
    g.fillOval((0.25*w -d/2).toInt(), (0.5*h -d/2).toInt(), d, d)
    g.fillOval((0.75*w -d/2).toInt(), (0.5*h -d/2).toInt(), d, d)
  }
```
Wenn ihr alles richtig abgetippt/kopiert habt, dann könnt ihr jetzt das Programm kompilieren und ausführen durch \<Strg\>+\<R\>.

## Was bedeutet das?

1. `g.color = Color.black` und `g.color = Color.white` bedeutet, dass die nächsten Objekte in Schwarz (engl. black) oder Weiß (white) gemalt werden.

2. `width`, `height` ist die Breite und Höhe des Fensters.  Die braucht man, damit sich das Gemälde der Größe des Fensters anpassen kann.

3. `g.fillOval(x,y, w,h)` malt eine Ellipse die bei `(x,y)` (links oben) anfängt und $w\times h$ groß ist. Wenn $w=h$ ist, dann ist es eine runde Scheibe;

4. `d` ist der Durchmesser der Pupillen;

5. `0.25*w` ist links, `0.75*w` ist rechts,

6. `-d/2` heißt, dass die Pupille um den halben Durchmesser nach links/nach oben versetzt anfängt

7. `, d, d` heißt, dass jede Pupille Durchmesser `d` hat (und rund ist, nicht oval);

8. `(...).toInt()` ist eine Anpassung aus folgendem Grund:  Wenn man `0.25*w` ausrechnet, kommt eine gebrochene Zahl heraus, z.B. 10.25.  Die Position und Größe der Ellipse muss aber in ganzen Zahlen angegeben werden, also verwandeln wir einfach die gebrochene Zahl in die nächste ganze Zahl, z.B. 10.25 in 10.

Wenn du alles fehlerfrei abgetippt hast, sollte das Programm jetzt wieder kompilieren und nach dem Start dich 2 schwarze Pupillen anstarren.


## Mein Programm kompiliert nicht mehr

Wenn du eine Fehlermeldung wie "')' expected" erhälst und sich das Fenster mit den schwarzen Pupillen nicht öffnet, dann hast du leider einen Tippfehler.  Schau mal in die Zeile mit der Fehlermeldung und zähle die Anzahl öffnender '(' und schließender ')' Klammern, irgendeine fehlt da.

Ein paar Tipps:

* alle Klammern müssen rund '(' und ')' sein;
* der Dezimalpunkt muss ein Punkt sein, also `0.25` nicht `0,25`;
* jeder Aufruf von `g.fillOval()` braucht 4 Parameter, also 3 Kommas und zwischen den Kommas müssen alle geöffneten Klammern auch wieder geschlossen werden;
* das zweite `g.color = Color.black` nicht vergessen, sonst hast du weiße Pupillen in weißen Augenrändern (unsichtbar);
* falls `Color` und `min(` noch rot dargestellt werden, musst du die importieren, also mit dem Cursor darauf zeigen und \<Alt\>+\<Enter\> drücken.  Bei `min(` gibt es vielleicht mehrere Auswahlmöglichkeiten, dort eine aus dem Paket "kotlin.math" wählen (den korrekten Typ passt er beim Kompilieren an).


## Das Weiße der Augen ist Rechteckig

Das liegt daran, dass der Hintergrund auch weiß ist.  Du kannst das korrigieren, indem du die folgende Methode einfügst:
```Kotlin
  class Eyes :JComponent() {
    init {
      background = Color(255, 231, 214) // skin color
    }

    ...
  }
```


# 3. Augen bewegen

Der eigentliche Trick an Augen ist es, dass sie sich bewegen, z.B. einem Objekt folgen.  Das kann man wie folgt erreichen:
```Kotlin
  class Eyes :JComponent() {
    private var dxL = 0.0
    private var dxR = 0.0
    private var dy = 0.0

    init {...}

    override fun paint(g :Graphics?) {
      ...
      g.color = Color.black
      val d = ...
      g.fillOval((0.25*w +0.125*w*dxL -d/2).toInt(), (0.5*h +0.5*h*dy -d/2).toInt(), d, d)
      g.fillOval((0.75*w +0.125*w*dxR -d/2).toInt(), (0.5*h +0.5*h*dy -d/2).toInt(), d, d)
    }
  }
```
Dabei bedeutet $\Delta x_L$ (also `dxL`) die waagerechte Verschiebung des linken Auges, $\Delta x_R$ die waagerechte Verschiebung des rechten Auges und $\Delta y$ die senkrechte Verschiebung der Augen.

## Wie rechnet man `dxL`, `dxR` und `dy` aus?

Dazu muss man die Mausposition abfragen.  Das kann man aber nicht nur einmalig beim (ersten) Malen machen, sondern muss man immer tun, wenn sich die Maus bewegt.  Dazu ergänzen wir die Augen um einen `MouseMotionListener` wie folgt:
```Kotlin
  class Eyes :JComponent(), MouseMotionListener {
    ...
  }
```
Dann musst du wieder \<Alt\>+\<Enter\> auf dem `MouseMotionListener` drücken und das Symbol aus dem Paket "java.awt.event" importieren.

Jetzt wird `class Eyes` rot unterstrichen, weil 2 Methoden fehlen.  Dazu stellst du den Cursor auf `class Eyes` und drückst wieder \<Alt\>+\<Enter\>.  Im Menü sollte es den Punkt "Methoden Implementieren" geben, den du auswählst.  Aus dem folgenden Dialog wählst du alle Methoden aus, also `mouseMoved(MouseEvent?)` und `mouseDragged(MouseEvent?)` und drückst "Ok".

Danach sollte der folgende Quelltext entstanden sein:
```Kotlin
  class Eyes :... {
    private var ...

    override fun mouseDragged(e :MouseEvent?) {
      super.mouseDragged(e)
    }

    override fun mouseMoved(e :MouseEvent?) {
      super.mouseMoved(e)
    }

    ...
  }
```
Diese neuen Methoden änderst du nun wie folgt:
```Kotlin
  override fun mouseDragged(event :MouseEvent?) {
    if (event!=null)
      mouseMoved(event)
  }

  override fun mouseMoved(event :MouseEvent?) {
    if (event==null) {
      super.mouseMoved(null)
      return
    }

    val xL = event.x -width/4;  val xR = event.x -3*width/4
    val y = event.y -height/2
    val r2L = sqr(xL) +sqr(y) +sqr(width/4) +sqr(height/2)
    val r2R = sqr(xR) +sqr(y) +sqr(width/4) +sqr(height/2)
    val fL = 1/sqrt(r2L.toDouble());  val fR = 1/sqrt(r2R.toDouble())
    dxL = xL*fL;  dy = y*(fL+fR)/2
    dxR = xR*fR
    repaint()
  }

}

fun sqr(x :Int) = x*x
fun sqr(x :Double) = x*x
```
Dabei musst du noch die Funktion `sqrt` importieren, also \<Alt\>+\<Enter\> und irgendeine aus dem Paket "kotlin.math" auswählen.


## Was bedeutet das?

1. $x_L = \mathrm{event}.x - \mathrm{width}/4$ ist der x-Abstand der Mausposition (`event.x, event.y`) von der Mitte des linken Auges -- also $x_L<0$ wenn die Maus links von (der Mitte des) linken Auges ist und $x_L>0$ wenn die Maus rechts von (der Mitte des linken) Auges ist;

2. $x_R = \mathrm{event}.x - {}^3/_4*\mathrm{width}$ ist der x-Abstand der Mausposition von der Mitte des rechten Auges;

3. $y = \mathrm{event}.y -\mathrm{height}/2$ ist der y-Abstand der Mausposition von der Mitte der Augen;

4. `fun sqr(x :Int) = x*x` ist das Quadrat (engl. square) und $r^2_L$ das Quadrat des Abstands der Mausposition von der Mitte des linken Auges (s.a. Satz des Pythagoras);

5. `+10` ist ein Sicherheitsabstand, d.h. dass der Abstand mindestens 10 ist (weil wir gleich durch diesen Abstand dividieren);

6. $ \Delta x_L = x_L\*f_L$; $ \Delta x_R = x_R\*f_R$ bedeutet, dass sich das Auge etwas in Richtung des Mauszeigers bewegt, aber nur um einen kleinen Faktor;

7. $f_L = 1/\sqrt{r^2_L}$ und $f_R = 1/\sqrt{r^2_R}$ ist dieser kleine Faktor, `sqrt` bedeutet Quadratwurzel (engl. square root), das hängt mit dem Satz des Pythagoras zusammen (wir haben oben ja $r^2_L$ und $r^2_R$ ausgerechnet);

8. $\Delta y = y*(f_L+f_R)/2$ für die Höhenverschiebung der Augen verwenden wir den durchschnittlichen Faktor, insbesondere schauen beide Augen immer gleich hoch (sonst würden sie ja schielen);

9. `repaint()` bedeutet, dass wir dem Computer sagen, dass er die Augen neu malen muss.  Wir können nicht einfach `paint(g :Graphics?)` aufrufen, weil der Computer vielleicht gar nicht bereit ist zum Malen (wir kennen `g` noch nicht);


## Eine Sache fehlt noch

Wir müssen dem Fenster noch sagen, dass wir benachrichtigt werden wollen, wenn sich die Maus bewegt.  Das geht so:  In der Datei "Main.kt" ersetzen wir die Zeile
```Kotlin
  // hier kommt noch etwas
```
durch folgendes:
```Kotlin
  addMouseMotionListener(contentPane)
```


Das war jetzt viel Theorie, probier es doch einfach mal aus (mit \<Strg\>+\<R\> starten).

Wenn alles stimmt, dann sollten die 2 Augen zum Mauszeiger schauen und sich mitbewegen.  Es kann aber sein, dass du erst einmal auf das Fenster mit den Augen klicken musst (oder zumindest der Mauszeiger in diesem Fenster sein muss).


# 9. Mehr selbst probieren

## 9.1 Bunte Augen

Versuch doch mal, grüne Augen zu malen!

Dazu gibt es 2.5 Möglichkeiten:

1. Du ersetzt die Zeile
  ```Kotlin
    g.color = Color.white
  ```
  durch
  ```Kotlin
    g.color = Color.green
  ```

2. oder du ersetzt die Zeile
  ```Kotlin
    g.color = Color.black
  ```
  durch
  ```Kotlin
    g.color = Color.green.darker()
  ```

3. Du malst zwischen Augenhintergrund (die weißen Ovale) und Pupillen (die schwarzen Ovale) noch eine grüne Iris, z.B. so
  ```Kotlin
    g.color = Color.green.darker()
    val d = min((0.275*w).toInt(), (0.35*h).toInt())
    g.fillOval((0.25*w +0.125*w*dxL -d/2).toInt(), (0.5*h +0.5*h*dy -d/2).toInt(), d, d)
    g.fillOval((0.75*w +0.125*w*dxR -d/2).toInt(), (0.5*h +0.5*h*dy -d/2).toInt(), d, d)

    g.color = Color.black
    g.fillOval((0.25*w +0.125*w*dxL -d/4).toInt(), (0.5*h +0.5*h*dy -d/4).toInt(), d/2, d/2)
    g.fillOval((0.75*w +0.125*w*dxR -d/4).toInt(), (0.5*h +0.5*h*dy -d/4).toInt(), d/2, d/2)
  ```

Verstehst du den Unterschied zwischen 1, 2 und 3?

Was must du tun, wenn die rechte Iris blau sein soll?


## 9.2 Schielende Augen

Da gibt es einige Möglichkeiten.  Am Einfachsten ist es wohl, wenn du ein Auge etwas höher blicken lässt, z.B. so:
```Kotlin
  g.fillOval((0.75*w +0.125*w*dxR -d/2).toInt(), (0.4*h +0.5*h*dy -d/2).toInt(), d, d)
```

Die y-Koordinate zählt von oben, also $y=0$ ist am oberen Rand des Fensters, $y=0.5\*h$ in der Mitte und $y=h$ am unteren Rand.  Wenn du nun statt $y=0.5\*h$ einfach $y=0.4\*h$ wählst, dann ist das etwas höher als die Mitte.  Und wenn du das nur bei einem Auge machst, dann blickt das höher als das andere.


## 9.3 Schneemann malen

Kannst du die Malroutine (`fun paint(g :Graphics?)`) so abändern, dass anstatt von 2 Augen 3 Kugeln übereinander gemalt werden?

Hinweis:  Jede Kugel sollte den Durchmesser `val d = h/3` haben und die erste Kugel beginnt oben ($y=0$), die zweite nach 1/3 ($y=h/3$) und die dritte nach 2/3 ($y=2*h/3$).

Viel Spaß beim Ausprobieren und bitte sag Bescheid, wenn es Probleme gibt.
