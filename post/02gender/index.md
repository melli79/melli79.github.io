Heute wollen wir uns fragen, wie der Computer am Namen erkennen kann, ob jemand Mann oder Frau ist.

# 1. Woran erkennt man, ob jemand weiblich ist?

Natürlich ist der Vorname nicht die einzige Quelle, anhand derer man das entscheiden kann, z.B. kann man oft auch am Surf- oder Kaufverhalten herausfinden, ob jemand Mann (Junge) oder Frau (Mädchen) ist.

Auch ist es nicht immer ganz eindeutig, ob jemand Mann oder Frau ist, z.B. gibt es Männer, die sich schminken oder Frauen, die ... .  Aber das soll jetzt erst einmal nicht stören.

Als Ausgangspunkt dienen uns diese 2 Listen von [männlichen](/maleNames.csv) und [weiblichen](/femaleNames.csv) Vornamen.  Das sind die häufigsten männlichen und weiblichen Vornamen in Deutschland (im Jahre 2021), sortiert nach Häufigkeit.

Sprachwissenschaftler haben herausgefunden, dass man in verschiedenen Sprachen männliche und weibliche Namen an verschiedenen Merkmalen erkennen kann.  Im Zweifelsfall kann man sich zu jeden Namen einzeln merken, ob er weiblich oder männlich ist, aber geht es auch einfacher?  Manche Sprachen, wie etwa das Deutsche haben typische Eigenschaften von weiblichen Namen und alle Namen, die nicht typisch weiblich sind, sind dann eben männlich.  Das machen wir uns zunutze und wollen eine Funktion schreiben, die entscheidet, ob ein Vorname weiblich ist, also etwa so:

```Kotlin
  fun isFemale(name :String) :Boolean {
    return false
  }
```

An die Bezeichnung `fun` erinnerst du dich vielleicht noch, sie bedeutet function und kennzeichnet einen Teil des Programmes.  Im Gegensatz zur Hauptfunktion `fun main()` nimmt unsere Funktion ein Argument entgegen, konkret den Namen als Text `(name :String)`.  Außerdem gibt die Funktion explizit etwas zurück, nämlich die Antwort, ob der Name weiblich ist (`true`, wahr) oder nicht (`false`, falsch).  Dieser Typ heißt `Boolean`.

Natürlich ist die Funktion noch nicht fertig, denn nicht alle Vornamen sind nicht weiblich, aber im Prinzip muss die Funktion so ähnlich aussehen.

# 2. Wie gut ist unsere Funktion?

Als nächstes wollen wir fragen, wie gut unsere Funktion ist.  Wir wissen schon, dass sie nicht immer richtig antwortet, aber vielleicht können wir wenigstens sagen, wie oft sie richtig antwortet.  Dazu schreiben wir ein Hauptprogramm, dass ein paar weibliche, ein paar männliche Namen einliest und dann schaut, ob die Funktion richtig entscheidet.

Mit Hilfe der for-Schleifen vom letzten Mal können wir das erreichen:

```Kotlin
  fun main() {
    ...
    for (n in 1..10) {
      print("Geben Sie einen weiblichen Vornamen ein (oder drücken Sie direkt Enter): ")
      val line = readLine() ?: break
      val name = line.trim()
      if (name.isEmpty())
        break
      isFemale(name)
    }
    for (n in 1..10) {
      print("Geben sie einen männlichen Vornamen ein (oder drücken Sie direkt Enter): ")
      val line = readLine() ?: break
      val name = line.trim()
      if (name.isEmpty())
        break
      isFemale(name)
    }
    ...
  }
```

`readLine()` liest eine Zeile von der Kommandozeile ein und `?: break` bricht die Schleife ab, falls die Eingabe abgebrochen wurde.  `if (name.isEmpty())` testet, ob der Name leer ist.  In diesem Falle wird mit `break` ebenfalls die Schleife abgebrochen.

Wie kommen wir jetzt zum Auszählen?  Wir müssten irgendwie abhängig vom Ergebnis `isFemale(name)` einen Zähler erhöhen.  Den Zähler können wir so anlegen: `var truePositive = 0`, d.h. wir legen eine Variable mit Namen `truePositive` (also die korrekt als weiblich erkannten) an und fangen bei 0 an. `var` im Gegensatz zu `val` bedeutet, dass sich der Wert im Laufe des Programmes ändern kann.

Entsprechend brauchen wir auch eine Variable `var falseNegative = 0` (fälschlicher Weise als nicht-weiblich erkannt), `var trueNegative = 0` (korrekt als nicht-weiblich erkannt) und `var falsePositive = 0` (fälschlicher Weise als weiblich erkannt).

Wenn wir wissen, dass der Name korrekt als weiblich erkannt wurde, dann können wir mit `truePositive++` den Zähler um 1 erhöhen, entsprechend mit `falseNegative++` die Anzahl der falsch zugeordneten weiblichen Namen um 1 erhöhen.

Wir dürfen aber immer nur 1 der beiden Zähler erhöhen, abhängig vom Ergebnis von `isFemale(name)`.  Das heißt, wenn `isFemale(name)`, dann `truePositive++`, ansonsten `falseNegative++`.  Jetzt müssen wir das nur noch dem Computer verständlich machen.  Die Sprache Kotlin ist am Englischen angelehnt, d.h. "Wenn" wird mit `if` übersetzt (nicht `when`, da ja nicht klar ist, ob die Bedingung jemals eintritt) und "ansonsten" heißt `else`.  Insgesamt sieht das dann in der ersten for-Schleife so aus:

```Kotlin
  if (isFemale(name))
    truePositive++
  else
    falseNegative++
```

In der zweiten for-Schleife haben wir nur männliche Namen, d.h. wenn ein männlicher Namen als nicht-weiblich erkannt wird, dann ist das ein korrekt negativer Fall, ansonsten ein falscher positiver Fall.

```Kotlin
  if (!isFemale(name))
    trueNegative++
  else
    falsePositive++
```
`!isFemale(name)` bedeutet nicht-weiblich, `!` negiert also wahr/falsch-Werte.

Wie können wir nun am Ende die Auszählung ausgeben?

Wir könnten einfach die 4 Zahlen ausgeben (`truePositive`, `falseNegative`, `trueNegative`, `falsePositive`), aber dann müsste man sich immer merken, welches welche Zahl ist.  Außerdem ergibt sich die Zahl der `falseNegative`, wenn man die Gesamtzahl der weiblichen Namen kennt.  Diese sollten wir auch mit angeben.  Entsprechend ergibt sich die Zahl der `falsePositive`, wenn man auch die Anzahl der männlichen Namen kennt.  Das kann man etwa so machen:

```Kotlin
  val positives = truePositive +falseNegative
  val negatives = trueNegative +falsePositive
  val numberOfErrors = falsePositive +falseNegative
  println("Korrekt weiblich erkannt: $truePositive/$positives;  korrekt männlich erkannt: $trueNegative/$negatives;  Gesamtzahl Fehler: $numberOfErrors.")
```
Vom letzten Mal kennst du dich sicherlich noch `println()`, das gibt eine neue Zeile aus.  Neu ist, dass man da auch einen Text angeben kann, z.B. `"Gesamtzahl Fehler: $numberOfErrrors."`.  Texte werden immer in doppelten Anführungszeichen geschrieben.  Interessant ist auch, dass der Computer nicht wörtlich den Text "\$numberOfErrors" ausgibt, sondern dort die Anzahl der Fehler einsetzt (also die Zahl ausgibt), dazu dient das `$`.


# 2. Was erzeugt das Programm?

Das bisher geschriebene Programm sieht etwa so aus:

```Kotlin
  fun isFemale(name :String) :Boolean {
    return false
  }

  fun main() {
    var truePositive = 0;  var falseNegative = 0
    for (n in 1..10) {
      print("Bitte geben Sie einen weiblichen Vornamen ein (oder drücken Enter zum Beenden): ")
      val line = readLine() ?: break
      val name = line.trim()
      if (isFemale(name))
        truePositive++
      else
        falseNegative++
    }

    var trueNegative = 0;  var falsePositive = 0
    for (n in 1..10) {
      print("Bitte geben Sie einen männlichen Vornamen ein (oder drücken Sie Enter zum Beenden): ")
      val line = readLine() ?: break
      if (!isFemale(name))
        trueNegative++
      else
        falsePositive++
    }

    val positives = truePositive +falseNegative
    val negatives = trueNegative +falsePositive
    println("Korrekt weiblich erkannt: $truePositive/$positives;  korrekt männlich erkannt: $trueNegative/$negatives;  Gesamtzahl Fehler: $numberOfErrors.")
  }
```

Wenn wir jetzt das Programm starten (auf den grünen Pfeil am linken Rand klicken), dann bleibt das Programm stehen mit dem Text "Bitte geben Sie einen weiblichen Vornamen ein (...): ".  Hier geben wir jetzt der Reihe nach weibliche Vornamen ein, z.B.
```log
  Julia<Enter>
  Barabara<Enter>
  Susanne<Enter>
  Ina<Enter>
  <Enter>
```

Danach fragt das Programm nach männlichen Vornamen.  Hier können wir eingeben:
```log
  Luca<Enter>
  Konrad<Enter>
  Melchior<Enter>
  Dieter<Enter>
  Wolfgang<Enter>
  <Enter>
```

Danach gibt das Programm die Auszählungen aus, etwa:
```log
  Korrekt weiblich erkannt: 0/4;  korrekt männlich erkannt: 5/5;  Gesamtzahl Fehler: 4.
```

Funktioniert das Programm?

Wenn du es fehlerfrei geschrieben hast, dann sollte es (bei obiger Eingabe) dieses Resultat ausgeben.  Was bedeutet das?  Das Programm hat alle männlichen Vornamen korrekt erkannt, aber alle weiblichen Vornamen falsch zugeordnet, also 4 Fehler gemacht.  Das ist ja auch kein Wunder, weil das Programm einfach immer behauptet hat, dass der Vorname nicht weiblich ist.  Das ist noch nicht gut.


# 3. Ein bisschen bessere Erkennung

Wie kann man die Erkennung weiblicher Vornamen besser machen?  Naja, wenn wir uns die weiblichen Vornamen anschauen, dann enden die meist auf "a" und "e", vielleicht auch manchmal auf "in".  Dann können wir das doch einbauen.  Wenn der Name auf "a" endet oder der Name auf "e" endet oder der Name auf "in" endet, dann ist er weiblich.  Am Ende bleiben nur noch die männlichen Namen übrig.

```Kotlin
  fun isFemale(name :String) :Boolean {
    if (name.endsWith("a")||name.endsWith("e")||name.endsWith("in"))
      return true
    return false
  }
```

Hier haben wir 3 neue Elemente:  `name.endsWith("a")` bedeutet, dass der Computer prüft, ob der Text im Namen auf "a" endet (engl. endsWith).  Hierzu ist es wichtig, dass der Name am Ende kein Leerzeichen oder Zeilenende enthält.  `||` bedeutet "oder", also `name.endsWith("a")||name.endsWith("e")` ob der Name auf "a" oder auf "e" endet.

Schließlich bedeutet `return true`, dass `true`, also wahr, zurückgegeben wird.  Das `return` bewirkt noch etwas anderes, nämlich, dass die Funkion hier endet.  Wenn also der Name auf "a", "e" oder "in" endet, dann wird `true` zurückgegeben und die Funktion beendet.  (Der letzte Befehl `return false` wird nicht ausgeführt).

Wenn die Bedingung falsch ist, also der Name weder auf "a" noch auf "e" noch auf "in" endet, dann wird `return true` nicht ausgeführt.  Stattdessen wird die nächste Zeile ausgeführt: `return false`, also ist der Name nicht weiblich.

Wenn wir jetzt das Programm wieder ausführen und die gleichen Namen wie vorher eingeben, dann ergibt sich:

```log
  Korrekt weiblich erkannt: 4/4;  korrekt männlich erkannt: 4/5;  Gesamtzahl Fehler: 1.
```

D.h. die weiblichen Namen wurden alle richtig erkannt, aber ein männlicher Name wurde falsch erkannt.  Kannst du erkennen, welcher Name falsch erkannt wurde?  Immerhin hat das Programm jetzt nur noch 1 Fehler gemacht, also schon besser.

Auflösung:  Der männliche Name "Luca" wird falsch erkannt, da wir gesagt haben, dass alle Vornamen auf "a" weiblich sind.  Was kann man da tun?  Wir können entweder sagen, dass 1 Fehler bei 9 Eingaben nicht so schlecht ist, oder wir müssen eine weitere Fallunterscheidung machen, z.B. so:

```Kotlin
fun isFemale(name :String) :Boolean {
  if (name.endsWith("uca")||name.endsWith("scha"))
    return false
  if (name.endsWith("a")||name.endsWith("e")||name.endsWith("in"))
    return true
  return false
}
```

Wir testen also zuerst, ob der Name auf "uca" oder "scha" endet.  Das ist bei "Luca", "Sascha" oder "Grischa" der Fall, alles männliche Namen.

Wenn wir jetzt das Programm erneut ausführen, dann kommen wir auf:

```log
  Korrekt weiblich erkannt: 4/4;  korrekt männlich erkannt: 5/5;  Gesamtzahl Fehler: 0.
```

Das sieht schon seht gut aus, aber stimmt es wirklich immer?


# 4. Wie gut ist die Erkennung für große Namenslisten?

Bisher mussten wir die weiblichen und männlichen Namen immer wieder eingeben.  Außerdem erlaubt uns das Programm nur maximal je 10 Namen einzugeben.  Wie können wir die gesamten Listen (populärer weiblicher und männlicher) Namen abarbeiten?

Dazu müssten wir die Namen nicht vom Benutzer, sondern aus den Dateien einlesen, etwa so:

```Kotlin
  val femalesFilename = "femaleNames.csv"
  var file = fopen(femalesFilename, "rt") ?: throw IllegalArgumentException("Kann die Datei $femalesFilename nicht lesen.")
  val bufferSize = 128
  val buffer = malloc(bufferSize.toULong()) as CValuesRef<ByteVar>? ?: throw IllegalStateException("Vasamm exhausted")
  for (n in 10..1000) {
    val line = fgets(buffer, bufferSize, file)?.toKString() ?: break
    val name = line.trim()
    ...
  }
  fclose(file)
```

Da sind jetzt einige neue Elemente enthalten.  Also der Reihe nach: `fopen(name, "rt")` öffnet (engl. open) eine Datei (english file) zum Lesen.  Das Ergebnis ist die geöffnete Datei (oder nichts). `?: throw IllegalArgumentException("...")` bedeutet, dass wir einen Ausnahmefall (engl. exception) feststellen wollen, wenn sich die Datei nicht öffnen lässt.  Ausnahmefall bedeutet so auch, dass wir dann nicht mehr weiterarbeiten wollen (das Programm sich beendet).  Ist das gut?  Naja, der Nutzer erhält noch die Textnachricht "...", bevor sich das Programm beendet.  Da steht drinnnen, dass die Datei nicht geöffnet werden kann.  Ohne die Datei mit den Namen kann das Programm nicht weiterarbeiten, deshalb habe ich beschlossen, hier einen Ausnahmefall zu erzeugen.

`val buffer = malloc(bufferSize)` legt einen Puffer (engl. buffer), also temporären Zwischenspeicher an.  Das ist notwendig, damit beim Einlesen der Namen aus der Datei diese zwischengespeichert werden können.  Leider muss man auch angeben, wieviel Platz man dafür einräumt, hier `val bufferSize = 128`, also 128 einfache Zeichen (für einen Namen).  Was bedeutet `as CValuesRef<ByteVar>?` und warum ist das gelb? `malloc` kommt vom Betriebssystem und dem ist es egal, welche Werte wir in dem Speicher speichern.  Kotlin aber nicht.  Mit `as CValuesRef<ByteVar>?` erklären wir Kotlin, dass der Speicher für Bytes von beliebiger Gesamtlänge verwendet werden soll.  Das ganze ist gelb, weil die Entwicklungsumgebung nicht recht glauben kann, dass der Speicher wirklich dafür geeignet ist.  Ich weiß aber, dass es funktioniert, probier das Programm am Ende aus.  Das `?: throw IllegalStateException("...")` kennen wir schon, es bedeutet wieder einen Ausnahmefall, wenn wir keinen Speicher bekommen haben (dann kann man auch nicht mehr weiterarbeiten).

`fgets(buffer, bufferSize, file)` liest eine Zeile ein, diesmal nicht von der Kommandozeile, sondern aus der Datei (`file`).  Die eingelesene Zeile wird in `buffer` gespeichert und darf maximal `bufferSize` Zeichen lang sein, danach wird angehalten.

Wenn wir bereits am Ende der Dateil sind, dann bewirkt `?: break`, dass die Schleife abbricht (engl. break heißt abbrechen, manchmal auch unterbrechen, z.B. lunch break). `for (n in 10..1000)` wird also nicht wirklich 991 mal ausgeführt, nur maximal 991 mal.  (In der Datei stehen etwa 365 Namen).

Schließlich noch `fclose(file)`, das ist das Gegenstück zu `fopen(...)`.  Damit wird die Datei wieder geschlossen und deren Arbeitsspeicher freigegeben.  Das ist wichtig, damit das Programm im Arbeitsspeicher nicht immer größer wird.

Der Rest in dieser for-Schleife ist gleich geblieben.

Die zweite for-Schleife für die männlichen Namen sieht ähnlich aus:

```Kotlin
  val malesFilename = "maleNames.csv"
  file = fopen(malesFilename, "rt") ?: throw IllegalArgumentException("Kann die Datei $malesFilename nicht lesen.")
  for (n in 10..1000) {
    val line = readLine(buffer, bufferSize, file) ?: break
    ...
  }
  fclose(file)
  free(buffer)
```

`free(buffer)` ist das Gegenstück zu `val buffer = malloc(...)`.  Es beudetet, dass wir den Pufferspeicher wieder freigeben (engl. free, frei oder befreien, freigeben).

Damit das ganze funktioniert, muss man beim Herunterladen die 2 Dateien unter diesen Namen abspeichern:  die [weiblichen Vornamen](/femaleNames.csv) unter "femaleNames.csv" und die [männlichen Vornamen](/maleNames.csv) unter "maleNames.csv".  Alles im Projekt-Verzeichnis.

Mit den obigen Veränderungen bekomme ich:

```log
  Korrekt weiblich erkannt: 280.0/365.0;  Korrekt männlich erkannt: 315.0/345.0, Anzahl Fehler: 115.
```

Sicherlich ist immer noch nicht alles perfekt, aber es ist auch nicht ganz klar, ob die 115 Fehler viel oder wenig sind.  Deshalb ist es vielleicht besser, wenn wir die Raten berechnen, d.h. 280/365 teilen, 315/345 teilen und statt 115 den Durchschnitt der Fehlerraten ausgeben, etwa so:

```Kotlin
  val tpr = round(truePositive/positives*100).toInt()
  val tnr = round(trueNegative/negatives*100).toInt()
  val correctness = (tpr +tnr)/2
  val mismatch = round((1 - correctness)*100).toInt()
  println("Rate korrekt weiblich erkannter: $tpr%;  Rate korrekt männlich erkannter: $tnr%;  Fehlerrate: $mismatch%.")
```

Hier bedeutet `round(...).toInt()` das auf ganze Zahlen gerundet wird. `...*100` bedeutet, dass wir die Rate in Prozent angeben (pro hundert), dementsprechend habe ich "$tpr%;" geschrieben.

Dann sieht das Ganze so aus:

```log
  Rate korrekt weiblich erkannter: 77%; Rate korrekt männlich erkannter: 91%, Fehlerrate: 16%.
```

Das heißt, dass etwa 77% der weiblichen Namen erkannt werden und 91% der männlichen Namen.  Im Durchschnitt 16% Fehler, das reicht immerhin schon mal für eine 2 (gut).

Man kann es noch etwas besser machen, wenn man noch ein paar weitere Endungen für weibliche Namen hinzufügt:

```Kotlin
  fun isFemale(name :String) :Boolean {
    if (name.endsWith("uca")||name.endsWith("scha"))
      return false
    if (name.endsWith("a")||name.endsWith("e")||name.endsWith("in")||name.endsWith("i")||name.endsWith("th"))
      return true
  return false
}
```

```log
  Rate korrekt weiblicher Namen: 79%; Rate korrekt männlicher Namen: 88%; Fehlerrate: 16%.
```

Es werden mehr weibliche Namen richtig erkannt, aber auch weniger männliche Namen.  Die Fehlerrate insgesamt ist gleich geblieben.

Wenn wir wissen wollen, wie gut die Erkennung für die Menschen in Deutschland ist, sollten wir vielleicht berücksichtigen, dass die Namen am Anfang der Listen häufiger sind.  Das kann man so machen:

```Kotlin
  var truePositive = 0.0;  var falseNegative = 0.0
  ...
    if (isFemale(name))
      truePositive += 1.0/n
    else
      falseNegative += 1.0/n
  ...
  var trueNegative = 0.0;  var falsePositive = 0.0
  ...
    if (!isFemale(name))
      trueNegative += 1.0/n
    else
      falsePositive += 1.0/n
  ...
```
`truePositive += 1.0/n` im Gegensatz zu `truePositive++` bedeutet, dass wir nicht konstant immer um 1 erhöhen, sondern um $1/n$.  Also beim ersten Durchlauf (`n=10`) um $1/10=0.10000$), beim zweiten Durchlauf (`n=11`) um $1/11=0.09090909...$, schon etwas weniger, und im letzten Durchlauf (`n=1000`) dann nur noch um $1/1000=0.0010000$.  Das sind zwar ziemlich kleine und krumme Zahlen, aber am Ende geben wir nur Verhältnisse aus und runden auf ganze Prozent:

```log
  Rate korrekt weiblicher Namen: 85%; Rate korrekt männlicher Namen: 91%;  Fehlerrate: 12%.
```
Also noch etwas besser.


Jetzt ist Zeit zum selber probieren:  Fällt dir noch eine Regel ein, an der man weibliche Vornamen erkennt?  Oder nicht-weibliche Vornamen?

Kannst du mit `val line = readLine() ?: return` das Programm so erweitern, dass es am Ende nach 1 Namen fragt und für diesen entscheidet (und ausgibt), ob er weiblich oder männlich ist?

[Lösung](/02solution.md)
