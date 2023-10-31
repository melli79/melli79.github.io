Nachdem wir letztens Wirbel im Computer animiert haben, wollen wir heute ein kleines animiertes Spiel produzieren.

# Das Ziel
<center>
<video width="320" height="240" controls autoplay="true" loop>
  <source src="/movies/spaceShooter.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
</center>

# 0. Programm aufsetzen
So wie bei den meisten Desktop Grafik-Apps, starten wir mit einem Hauptprogramm,
dass ein Fenster anlegt, in dem dann unsere Komponente dargestellt wird.

```kotlin
  abstract class GameComponent :JComponent() {
    abstract fun accelerateLeft()
    abstract fun accelerateRight()
    abstract fun shoot()
    abstract fun reset()
  }

  class MyWindow(private val content :JComponent) :JFrame("Space Shooter"), KeyListener {
    init {
      layout = BorderLayout()
      contentPane = content
      preferredSize = Dimension(600, 400)
      setSize(600, 400)
      defaultCloseOperation = EXIT_ON_CLOSE
      addKeyListener(this)
    }

    override fun keyPressed(event :KeyEvent) = when (event.code) {
      KeyEvent.VK_LEFT -> content.accelerateLeft()
      KeyEvent.VK_RIGHT -> content.accelerateRight()
      KeyEvent.VK_SPACE -> content.shoot()
      KeyEvent.VK_BACKSPACE -> content.reset()
      KeyEvent.VK_ESCAPE -> exitProcess(0)
      else -> {}
    }
    override fun keyReleased(event :KeyEvent) {}
    override fun keyTyped(event :KeyEvent) {}
  }

  fun main() {
    val window = MyWindow(SpaceShooter())
    window.isVisible = true
  }
```

Unser Fenster soll auf Tastatureingaben reagieren, deshalb registrieren wir `MyWindow` als `KeyListener`.  Dazu müssen wir 3 Methoden implementieren `keyTyped`, `keyPressed`, `keyReleased`.  Für unsere Zwecke reicht es, in der Funktion `keyPressed` etwas auszuführen, die anderen beiden Funktionen können also leer bleiben.

Damit wir nicht die `Shooter`-Komponente hart verdrahten, habe ich eine abstrakte Klasse `GameComponent` eingeführt.  Diese erfült 2 Zwecke.  Zum einen muss sie von `JComponent` erben, kann also kein reines `interface` sein.  Zum anderen definiert sie eine handvoll Methoden, die auf Tastaturereignisse reagieren.  Diese werden in der Methode `keyPressed` aufgerufen.  Zusätzlich haben wir da noch implementiert, dass man das Spiel mit \<ESC\> schnell beenden kann.

## 1 Der Shooter
Der eigentliche Dynamik geschieht in einer `GameComponent`, so wie wir das bisher mit einer `JComponent` gemacht haben:
```kotlin
  class SpaceShooter() :GameComponent() {
     val range = Rect.of(-1.0, 0.0, 1.0, 1.0)
     private lateinit var scale :Rect

     private val background = mutableListOf<Point>()
     private val shots = mutableListOf<Shot>()
     ...
     private var state = State.CONTINUE

     private lateinit var shot :Image
     private lateinit var bomb :Image
     private lateinit var hero :Image
     private lateinit var rock :Image
     private lateinit var boom :Image

     init {
       loadSprites()
       reset()
     }

     override fun reset() {
       state = State.CONTINUE
     }

     ...

     override fun paint(g :Graphics) {
       scale = computeScale(width, height)
       background.forEach {
         g.drawImage(rock, scale.px(it.x), scale.py(it.y), this)
       }
       for (shot in shots.toList()) {
         g.drawImage(this.shot, scale.px(shot.x), scale.py(shot.y), this)
         ...
       }
       ...
     }

     fun computeScale(width :Int, height :Int) :Rect {
       val dx = width/range.dx
       val dy = height/range.dy
       return Rect(range.x0 +10/dx, range.y1 +10/dy, dx, -dy)
     }

     data class Rect(val x0 :Double, val y0 :Double, val dx :Double, val dy :Double) {
       companion object {
         fun of(x0 :Double, y0 :Double, x1 :Double, y1 :Double) =
            Rect(x0, y0, x1-x0, y1-y0)
       }
       val y1 :Double
         get() = y0+dy

       fun px(x :Double) = ((x-x0)*dx).roundToInt()
       fun py(y :Double) = ((y-y0)*dy).roundToInt()
     }
  }
```

# 3. Dynamische Komponenten

## 3.1 Felsen

Der Hintergrund besteht aus einer handvoll Felsen, die durch Punkte gespeichert werden und sich nicht bewegen sollen.  Allerdings können die Felsen zerstört werden, sodass wir ihre Liste dynamisch machen `background = mutableListOf<Point>()`.  Woraus besteht nun ein Punkt?  Klar, x- und y-Koordinate, also

```kotlin
  data class Point(val x :Double, val y :Double) {}
```

Außerdem müssen wir am Anfang eine Hand voll Felsen anlegen, etwa so:

```kotlin
  fun fillBackground() {
    background.clear()
    (-9..9).forEach { background.add(Point(0.1*it, 0.9)) }
    (-5..< 5).forEach { background.add(Point(0.2*it+0.1, 0.8)) }
    (-2..2).forEach { background.add(Point(0.4*it, 0.7)) }
  }

  private fun loadSprites() {
    loadRock()
    loadBomb()
    loadShot()
    loadHero()
  }

  private fun loadRock() {
    rock = BufferedImage(20, 20, TYPE_4BYTE_ABGR)
    val g = rock.getGraphicsWithFont(Font.Bold, 64.0f)
    g.drawString("*", -2, 48)
    g.dispose()
  }

  private fun Image.getGraphicsWithFont(style :Int =Font.NORMAL, size :Float =18.0) {
    val g = graphics
    g.font = g.font.derive(size, style)
    g.color = Color.BLACK
    return g
  }
```

## 3.2 Bewegliches Flugzeug

Offenbar brauchen wir Koordinaten für das Flugzeug, also

```kotlin
  private var posX = 0.0
  private val posY = 0.0

  override fun paint(g :Graphics) {
    ...
    g.drawImage(hero, scale.px(posX), scale.py(posY), this)
  }
```

Außerdem wollen wir das Flugzeug bewegen, wennimmer wir die Tasten \<\<--\> oder \<--\>\> drücken, also

```kotlin
  private var vx = 0.0

  override fun accelerateLeft() {
     vx -= 0.01
     posX += vx
     if (posX>=1.0) {
       posX = 1.0
       vx = 0.0
     }
     if (posX<=-1.0) {
       posX = -1.0
       vx = 0.0
     }
     repaint()
  }

  override fun accelerateRight() {
     vx += 0.01
     posX += vx
     if (posX>=1.0) {
       posX = 1.0
       vx = 0.0
     }
     if (posX<=-1.0) {
       posX = -1.0
       vx = 0.0
     }
     repaint()
  }
```

Außerdem müssen wir die Form des Flugzeugs festlegen.

```kotlin
  private fun loadHero() {
    hero = BufferedImage(20, 20, TYPE_4BYTE_ABGR)
    val g = hero.getGraphicsWithFont()
    g.drawString("A", 5,2)
    g.dispose();
  }
```

Wenn du jetzt die fehlenden Methoden leer ergänzt -- `override fun shoot() {}`, `private fun loadBomb() {}`, `private fun loadShot() {}` und `data class Shot() {}` -- dann kannst du das Programm schon mal testen.  Es sollte ein paar Felsen am oberen Fensterrand malen und ein "Flugzeug" (also eigentlich ein 'A') am unteren Rand.  Wenn du die Tasten \<\<--\> und \<--\>\> drückst, bewegt sich das Flugzeug nach links oder rechts, umso schneller je öfter du drückst.


## 3.3 Schießen

Im Prinzip können wir Schüsse (engl. `shot`) einfach mit Folgendem erzeugen:

```kotlin
  override fun shoot() {
    ...
    shots.add(Shot(posX, posY))
    repaint()
  }
```

Allerdings wollen wir, dass die Schüsse durch's Fenster fliegen.  Dazu brauchen wir 2 zusätzliche Dinge:

```kotlin
import kotlin.concurrent.timer

class SpaceShooter() :GameComponent() {
  ...
  private var timer :Timer? =null
  ...

  override fun paint(g :Graphics) {
    if (timer==null)
      this.timer = timer(daemon= false, initialDelay= 100, period= 100) { evolve() }
    ...
  }

  private fun evolve() {
    for (shot in shots) {
      if (!shot.fly())
        shots.remove(shot)
    }
    ...
    repaint()
  }

  private fun loadShot() {
    shot = BufferedImage(10, 10, TYPE_4BYTE_ABGR)
    val g = shot.getGraphicsWithFont()
    g.drawString("^", 0, 5)
    g.dispose()
  }
```

Damit ist klar, dass ein Schuss folgendes können muss:
```kotlin
  data class Shot(var x :Double, var y :Double, val vy :Double =0.1) {
    fun fly() :Bool {
      y += vy
      return y in -1.0..1.0
    }

    val isUp = vy>0

    ...
  }
```

Das Verhalten des Flugzeugs ist etwas komisch:  Es bewegt sich immer genau dann, wenn man eine der Tasten nach links oder nach rechts drückt.  Stattdessen brauchen wir eher folgendes Verhalten:

```kotlin
  private fun evolve() {
    ...
    posX += vx
    if (posX<=-1.0) {
      posX = -1.0
      vx = 0.0
    } else if (posX>=1.0) {
      posX = 1.0
      vx = 0.0
    }
    repaint()
  }

  override fun accelerateLeft() {
    vs -= 0.01
  }

  override fun accelerateRight() {
    vs += 0.01
  }
```

## 3.4 Explosionen

Momentan gehen die Schüsse einfach durch die Felsen und verschwinden dann.  Stattdessen wollen wir aber, dass sie explodieren.  Das kann man etwa wie folgt erreichen:

```kotlin
  override fun paint(g :Graphics) {
    ...
    val pos = Point(posX, posY)
    for (shot in shots.toList()) {
      val obs = background.find { shot.hits(it) }
      if (obs!=null) {
        g.drawImage(boom, scale.px(shot.x), scale.py(shot.y), this)
        background.remove(obs)
        shots.remove(shot)
      } else if (shot.hits(pos)) {
        g.drawImage(boom, scale.px(shot.x), scale.py(shot.y), this)
        shots.remove(shot)
        state = State.EXPLODING
      } else if (shot.isUp)
        g.drawImage(this.shot, scale.px(shot.x), scale.py(shot.y), this)
      else
        g.drawImage(this.bomb, scale.px(shot.x), scale.py(short.y), this)
    }

    when (state) {
      State.CONTINUE -> g.drawImage(hero, scale.px(posX), scale.py(posY), this)
      State.EXPLODING -> if (remaining>0.0)
          remaining -= 0.1
        else
          state = State.GAME_OVER
      GAME_OVER -> {
        g.color = Color.red.lighter()
        g.font = g.font.derive(Font.BOLD, 48.0f)
        g.drawString("Game Over!", width/2 -120, height/2)
      }
      State.WINNING -> {
        g.drawImage(hero, scale.px(posX), scale.py(posY), this)
        g.color = Color.green.darker()
        g.font = g.font.derive(Font.BOLD, 48.0f)
        g.drawString("You Win!!", width/2 -120, height/2)
      }
    }
  }

  data class Shot(...) {
    ...

    fun hits(obs :Point) = abs(obs.x -z)+abs(obs.y -y) <= 0.09
  }
```

## 3.5 Dropping Bombs

Damit es nicht zu einfach wird zu gewinnen, wollen wir noch ab und zu Bomben auf das Flugzeug fallen lassen.  Dazu nehmen wir an, dass sich in den Felsen feindliche Bombenwerfer verstecken und diese zufällig eine Bombe fallen lassen.  Das kann man etwa so erreichen:

```kotlin
val random = Random(System.currentMillis())

class SpaceShooter() :GameComponent() {
  ...

  private fun evolve() {
    attack()
    ...
  }

  private fun attack() {
    if (state==GAME_OVER || state==State.WINNING)
      return
    if (random.flipCoin(0.1) && background.isNotEmpty()) {
      val enemy = background.random()
      shots.add(Shot(enemy.x, 0.7, -0.02))
    }
    if (background.isEmpty())
      state = State.WINNING
  }

  private fun loadBomb() {
    bomb = BufferedImage(16, 16, TYPE_4BYTE_ABGR)
    val g = bomb.getGraphicWithFont()
    g.drawString("v", 4, 15)
    g.dispose()
  }
  ...
}

fun Random.flipCoin(bias :Double =0.5) = nextDouble() < bias
```

Eine Münze zu werfen heißt im Englischen `flipCoin()`.  Für eine faire Münze ist die Wahrscheinlichkeit 50% (also 0.5).  Das tatsächliche Ergebnis hängt aber vom Zufall ab, d.h. wir brauchen eine Zufallsquelle (engl. `random`).  Diese müssen wir bei Programmstart initialisieren.  Das kann man am bequemsten tun, indem man einen Zufallsgenerator anlegt (`val random = Random(seed)`), allerdings brauchen wir noch einen Startwert (engl. seed), weil im Computer nichts wirklich zufällig abläuft.  Als recht zufälligen Startwert kann man die aktuelle Systemzeit in Millisekunden verwenden.  Selbst falls jemand das Spiel immer zur gleichen Uhrzeit spielt, wird er es kaum auf die Millisekunde genau zur gleichen Zeit starten.

Offenbar ist `fun Random.flipCoin(...)` eine Erweiterungsfunktion, d.h. sie ist eigentlich nicht in der Klasse (oder im Interface) von `Random` enthalten, aber wir können sie so bequem wie eine Methode aufrufen.  Entsprechend gibt es für Kollektionen (z.B. `List` oder `Set`) eine Erweiterungsfunktion (engl. extension function), die ein zufälliges Element auswählt.  Allerdings sollte man sicherstellen, dass die Kollektion nicht leer ist.


# 4. Noch etwas Polieren

Ok, im Prinzip funktioniert das Spiel jetzt, aber einige unserer Methoden sind ziemlich lang, z.B. `fun paint(g :Grapgics)` hat ca. 30 Zeilen.  Das können wir reduzieren, indem wir die in kleinere Teile aufteilen, die wir dann aus der `paint`-Methode heraus aufrufen, etwa so:

```kotlin
  private var points = 0

  fun paint(g :Graphics) {
    scale = computeScale(width, height)
    if (timer==null)
      timer = timer(daemon= true, initialDelay= 100, period= 100) { evolve() }
    paintRocks(g)
    paintShots(g)
    g.drawString("Points: ${points*100}", 0, 16)
    paintForState(g)
  }


  private fun paintRocks(g :Graphics) {
    for (rock in background) {
      g.drawImage(this.rock, scale.px(rock.x), scale.py(rock.y), this)
    }
  }

  private fun paintShots(g :Graphics) {
    for (shot in shots) {
      g.drawImage(if (shot.isUp) this.shot else this.drop, scale.px(shot.x), scale.py(shot.y), this)
      val obs = background.find { shot.isHitting(it) }
      if (obs != null) {
        g.drawImage(boom, scale.px(obs.x), scale.py(obs.y), this)
        points++
        background.remove(obs)
      }
    }
  }

  private fun paintForState(g :Graphics) {
    when (state) {
      State.GAME_OVER -> paintGameOver(g)
      State.WINNING -> {
        paintWon(g)
        paintHero(g)
      }

      else -> paintHero(g)
    }
  }

  private fun paintWon(g :Graphics) {
    g.color = Color.GREEN.darker()
    g.font = g.font.deriveFont(Font.BOLD, 48.0f)
    g.drawString("You Win!", width/2 -120, height/2)
  }

  private fun paintGameOver(g :Graphics) {
    g.color = Color.RED.brighter()
    g.font = g.font.deriveFont(Font.BOLD, 48.0f)
    g.drawString("Game Over!", width/2 -120, height/2)
  }

  private fun paintHero(g :Graphics) {
    if (state==State.EXPLODING)
      g.drawImage(boom, scale.px(posX), scale.py(posY), this)
    else
      g.drawImage(hero, scale.px(posX), scale.py(posY), this)
  }
```

Eine andere recht lange Methode is `fun evolve()`.  Auch die können wir in 5 Teile zerlegen:

```kotlin
  private fun evolve() {
    attack()
    replicateRocks()
    propagateShots()
    evolveHero()
    repaint()
  }

  // method attack() as before

  private fun propagateShots() {
    for (shot in shots.toList()) {
      if (!shot.fly())
        shots.remove(shot)
    }
  }

  private fun evolveHero() {
    moveHero()
    checkCollision()
  }

  private fun checkCollision() {
    if (state == State.EXPLODING) {
      if (remaining > 0)
        remaining -= 0.1
      else
        state = State.GAME_OVER
    } else {
      val hero = Point(posX, posY)
      if (shots.any { it.isHitting(hero) })
        state = State.EXPLODING
    }
  }

  private fun moveHero() {
    posX += vx
    if (posX >= 1.0) {
      posX = 1.0
      vx = 0.0
    } else if (posX < -1.0) {
      posX = -1.0
      vx = 0.0
    }
  }
```

## 4.1 Felsen Replizieren

Im Prinzip sollte das Program noch so, wie vorher funktionieren.  Allerdings ist mir noch eine Idee gekommen:  Wenn der Held zu lange wartet, dann replizieren sich die Felsen, etwa so:

```kotlin
  private fun replicateRocks() {
    if (background.size<32 && random.flipCoin(0.05)) {
      val candidates = generateBackground().filter { it !in background }
      background.add(candidates.random())
    }
  }
```

Dazu müssen wir aber das Erzeugen den Hintergrundes vom Füllen des Hintergrundes trennen, vielleicht so:
```kotlin
  private fun fillBackground() {
    background.clear()
    background.addAll(generateBackground())
  }

  private fun generateBackground() =
    (-9..9).map { Point(it*0.1, 0.9) } +
        (-5..< 5).map { Point(0.2*it +0.1, 0.8) } +
        (-2..2).map { Point(0.4*it, 0.7) }
```

So, jetzt kann man das Programm auch in 6 Monaten noch gut verstehen und man kann ganz gut damit spielen.


# 9. Selbst Probieren
Wie sieht es aus?  Funktioniert das Spiel bei dir?

Falls das Programm nicht auf Anhieb funktioniert, musst du dort schauen, wo die Fehlermeldung angezeigt wird.  Verstehst du, was der Code an dieser Stelle machen soll?  Worüber meckert der Compiler?  Wie kann man das beheben?

## 9.1 Ein weiteres Level

Wenn man alle Felsen entfernt hat, dann steht da "You Win!!", aber es passiert nichts weiteres.  Vielleicht fällt dir etwas ein, was man im nächsten Level schwieriger machen kann.  Auch wäre es sicherlich nett, wenn man das Level anzeigen würde.  Dazu solltest du eine Eigenschaft (engl. property) in der Klasse einführen (z.B. `private var level = 1`) und die entsprechend in der `paint()`-Methode mit ausgeben (z.B. hinter den Punkten).

Viel Spaß beim Probieren und Tunen.
