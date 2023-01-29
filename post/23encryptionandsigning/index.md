Nachdem wir letztes Mal den RSA Algorithmus kennen gelernt haben, wollen wir diesmal etwas allgemeiner über
Kryptographie sprechen.

Wer lieber ein (kurzes) Buch liest, dem sei [1] empfohlen.

[1] A. Beutelspacher: *Kryptologie, Eine Einführung in die Wissenschaft vom Verschlüsseln, Verbergen und Verheimlichen*, Taschenbuch, November **2014**.

# 1. Wenn Alice Bob eine geheime Nachricht schicken will...

Zunächst einmal, worum geht es eigentlich bei Kryptographie (als Teilgebiet der Mathematik/Informatik).  Wir haben 2 Subjekte, die geheime Informationen über einen öffentlichen Kanal austauschen wollen.  Traditionell heißen diese beiden Alice und Bob, also A und B.

Am sichersten wäre es, wenn Alice und Bob sich immer wieder in einem privaten Raum treffen könnten, in dem sie niemand belauschen kann und sie beliebig oft Nachrichten hin- und herreichen könnten.  Aber vielleicht wollen sie auch nicht, dass irgendjemand erfährt, dass sie miteinander kommunizieren, d.h. wir gehen davon aus, dass sie nur über Distanz und über öffentlich einsehbare Kanäle kommunizieren können.

Können sie das erreichen und wenn ja, wie?

Dazu wollen wir zunächst 4 Dinge unterteilen:

1. eine verschlüsselte Nachricht: ein Text, eine Zeichenfolge, Zahlen, ... an denen man die geheime Nachricht nicht unmittelbar ablesen kann.

2. eine geheime Nachricht: ein Text, den sich Alice ausgedacht hat und an Bob übermitteln will, ohne dass andere den mitbekommen.

3. Ein Algorithmus/Protokoll: ein Prinzip nach dem Alice eine geheime Nachricht verschlüsselt, dann öffentlich überträgt und anschließend (nur) Bob die geheime Nachricht wieder entschlüsseln kann.

4. Ein (öffentlicher/geheimer) Schlüssel:  ein zusätzliches Stück Information, das Alice und Bob einmalig ausgetauscht haben, das niemand anderes kennt, aber auf das sie zurückgreifen können, um den Algorithmus zu betreiben.


# 2. Cäsar-Chiffre
## 2.1 Erklärung

Die ersten Versuche, geheime Nachrichten zu übermitteln begannen damit, dass man einen geheimen Text durch geheime oder vertauschte Zeichen hinterlassen hat.  Der Herrscher Julius Cäsar war bekannt dafür, dass er geheime Nachrichten hinterließ, von denen nur eingeweite sie lesen konnten.  Was hat er dazu getan?

Er hatte 2 verschiebbare Reihen des (lateinischen) Alphabets, die er gegeneinander verschieben konnte, also etwa so:

| ABCDE|FGHIJ|KLMN|OPQR|STUV|WXYZ |
|------|-----|----|----|----|-----|
| XYZAB|CDEFG|HIJK|LMNO|PQRS|TUVW |

Und wenn er einen Nachricht, z.B. "TRIFF MICH HEUTE ABEND UM NEUN BEI DEN PROPYLAEN." verschlüsseln wollte, dann suchte er die Buchstaben der Reihe nach in der oberen Zeile und ersetzte sie durch die Buchstaben in der unteren Zeile.  Also das 'T' durch 'Q', das 'R' durch 'O' und so weiter, sodass sich die folgende verschlüsselte Nachricht ergibt:

> QOFC CJFZ EEBR QBXY BKAR JKBR KYBF ABKM OLMV IXBK

Auf einem wöchentlichen geheimen Treffen hat Caesar dann seinen Getreuen den Schlüssel verraten, hier "3 nach rechts" und jeder, der den (und den Algorithmus) kannte, konnte damit wieder die geheime Nachricht lesen, indem er die Buchstaben von unten nach oben ersetzte.

## 2.2 Als Programm
Mit einer kleinen Funktion sieht das etwa wie folgt aus:

```kotlin
  const val letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
  fun encrypt(shift :Byte, secretMessage :String) = secretMessage
    .filter { it in 'A'..'Z' }
    .map { c -> letters[(c-'A'+shift)%26] }
    .joinToString("")
```

Zunächst filtert die Funktion alle Zeichen raus, die keine Großbuchstaben sind und dann wird jedes Zeichen, z.B. 'T' mit 'A' verglichen, 'T' ist das 20. Zeichen, 'A'das erste, also `'T'-'A'` gibt 19.  Da wird der Shift hinzugefügt und dann im Wesentlichen der Buchstabe gewählt, der dieser Zahl entspricht.  Die Konvention ist, dass wenn man über die 26 hinaus kommt, man wieder vorne anfängt, das wird mit `()%26` bewirkt.  Schließlich hätte man eine Liste von Zeichen.  mit `.joinToStrin(...)` wird die wieder in einen String umgewandelt.  Das `""` bedeutet, dass zwischen 2 Zeichen ein leerer String (also nichts) eingefügt wird.

Aufrufen kann man diese Funktion etwa in folgendem Programm:
```kotlin
  fun main() {
    print("Caesar Chiffre\n  Bitte geben sie den Shift an (0-25, links positiv, rechts negativ): ")
    val shift = readlnOrNull()?.trim()?.toByteOrNull()
    if (shift==null) {
      println("Kein gültiger Shift.")
      return
    }
    while (true) {
      print("Geheime Nachricht: ")
      val message = readlnOrNull()?.trim()
      if (message.isNullOrEmpty())
        return
      val encrypted = encrypt((26 +shift.toInt()%26).toByte(), message)
    }
  }
```
Hier kann man jetzt das obige Ergebnis erzeugen, indem man als Shift "3" eingibt und dann die Original-Nachricht (Groß-/Kleinschreibung und Interpunktion wird entfernt).

Umgekehrt kann man mit diesem Programm wieder entschlüsseln, indem man den negativen Shift (hier also "-3") eingibt und die verschlüsselte Nachricht eintippt.

## 2.3 Wie sicher ist das?
Offenbar ist es wichtig, dass kein Spion den Shift erfährt, denn die Verschlüsselung ist höchstens so sicher, wie die Aufbewahrung dieses Schlüssels.

Aber ist das wirklich alles?

Ok, der Shift darf nicht 0 sein, denn dann sieht die verschlüsselte Nachricht genauso aus, wie die geheime Nachricht.

Aber auch dann ist die Nachricht noch nicht sicher:  Wenn die Nachricht einigermaßen lang ist, kann ein Angreifer einfach alle Schlüssel durchprobieren (es gibt nur 26) und schauen, ob er die Nachricht lesen kann.

In einem Programm sähe das etwa wie folgt aus:
```kotlin
  fun main() {
    print("Caesar Chiffre knacken.\n  Bitte geben sie die verschlüsselte Nachricht ein: ")
    val encrypted = readlnOrNull()?.trim()
    for (shift in 1..25) {
      val decrypt = encrypt(shift, encrypted)
      println("Shift $shift: $decrypt")
    }
  }
```

Jetzt müssen wir nur noch aufmerksam auf die 25 Zeilen Ausgabe schauen und können sowohl die geheime Nachricht als auch den Schlüssel finden.

## 2.4 Systematische Analyse
Im Zeitalter von Computern kann man natürlich einfach die gesamte Nachricht 25mal verschlüsseln und mit etwas Geduld die geheime Nachricht entschlüsseln.  Aber wenn man keinen Computer hat und stattdessen etwas Geduld, kann man auch folgendes tun:

Die häufigsten Buchstaben im Deutschen sind: E, N, I, R, T, S, L

Umgekehrt kann man (besonders bei einer längeren Nachricht) einfach durchzählen, welcher verschlüsselnde Buchstabe wie oft auftaucht und dann versuchen, diese mit den obigen Buchstaben abzupassen.  Diese Idee stammt von [al Kindī](https://de.wikipedia.org/wiki/Al-Kind%C4%AB).

In einem Programm sieht das etwa wie folgt aus:
```kotlin
  fun main() {
    print("Analyse einer Substitutionschiffre\n  Bitte geben Sie die verschlüsselte Nachricht ein: ")
    val encrypted = readlnOrNull()?.trim()?.uppercase(Locale.getDefault())
    if (encrypted.isNullOrEmpty())
      return
    val statistics = encrypted
      .filter { it in 'A'..'Z' }
      .groupBy { it }
      .map { Pair(it.key, it.value.size) }
      .sortedByDescending { it.second }
    println("Die häufigsten Buchstaben sind: ${statistics.take(7)}")
    val shift = (26+('E' -statistics.first().first))%26
    val decrypted = encrypt(shift.toByte(), encrypted)
    println("Der Shift ist wahrscheinlich: $shift,  die entschlüsselte Nachricht ist: $decrypted")
  }
```

Was bedeutet dieses Programm?

Nach der Begrüßung wird eine Zeile Text eingelesen `readlnOrNull()`.  Falls der Nutzer nur \<Enter\> drückt,  ist das Ergebnis `null`.  Anschließend werden noch die Leerzeichen am Anfang und am Ende entfernt `.trim()`.  Das `?.` bedeutet, dass das nur gemacht wird, falls nicht vorher schon `null` herauskam. `.uppercase(...)` bedeutet, dass alles in Großbuchstaben verwandelt wird.  Das hängt leider vom verwendeten Alphabet ab, weshalb man als Parameter den Standard (engl. `Locale`) übergeben muss.  `Locale.getDefault()` liest diesen Standard vom Betriebssystem aus, d.h. wenn dein Rechner auf Deutsch eingestellt ist, dann übergibt das `Locale.GERMAN` (engl. German heißt Deutsch).

Das Auszählen der Statistik funktioniert wie folgt: `encrypted.filter { it in 'A'..'Z' }` behält nur die Zeichen, die Großbuchstaben 'A' bis 'Z' sind. `.groupBy { ... }` gruppiert diese Zeichen nach dem Zeichen.  Das Ergebnis ist ein Wörterbuch/ eine Map, die aus allen vorgekommenen Zeichen besteht und hinter jedem eine Liste der Zeichen.  Das scheint zwar etwas komisch, liegt aber daran, dass `.groupBy` nicht nur zum Auszählen von Buchstaben verwendet werden soll.  Jedenfalls bilden wir dann noch die Einträge als `.map{ Pair(it.key, it.value.size) }` ab, d.h. aus einem Eintrag wie ['B':['B','B','B','B','B','B','B']] wird dann `Pair('B',7)`.  Anschließend sortieren wir die Liste der Häufigkeiten absteigend (engl. descending) nach der Häufigkeit.

Für die Ausgabe wollen wir nicht alle 24 (oder so) vorkommenden Buchstaben ausgeben, sondern nur die 7 häufigsten.  Das passiert mit `statistics.take(7)`.

Dann habe ich noch die Entschlüsselung automatisiert, d.h. ich gehe einfach davon aus, dass der häufigste Buchstabe das 'E' verschlüsselt und der Shift ist dann offenbar 'E' minus diesen Buchstaben, im Beispiel `'E'-'B'` also 3.  Ein kleines Problem am obigen `encrypt(...)` ist, dass es nur mit positivem Shift arbeitet, aber das kann man dadurch korrigieren, dass man ggf negative Shifts um 26 verschiebt, deshalb `shift = 26+('E'-'B')`.


# 3. Substitutions-Chiffre
## 3.1 Eine Einfache Idee
Die Frage ist, wie man diese Verschlüsselung sicherer machen kann.  Die erste Idee ist es vielleicht, nicht nur um eine Konstante zu verschieben (engl. shift), sondern die Buchstaben beliebig durcheinander einzusetzen.  Eine entsprechende Verschlüsselungstabelle könnte etwa wie folgt aussehen:

| ABCDE|FGHIJ|KLMN|OPQR|STUV|WXYZ |
|------|-----|----|----|----|-----|
| XZACE|GIKMO|QSUW|YBDF|HJLN|PRTV |

Ein Effekt ist, dass der Schlüssel jetzt größer ist, d.h. es reicht nicht, einen konstanten Shift zu übergeben, sondern man muss jeden einzelnen Buchstaben übersetzen, z.B. indem man die 2. Zeile aus obiger Tabelle übergibt.

## 3.2 Ist das nun besser?

Schauen wir uns den ersten Teil obiger Analyse an. Der verschlüsselte Text ist nun:

> JFMG GUMA KKEL JEXZ EWCL UWEL WZEM CEWB FYBT SXEW

und die Häufigkeitsanalyse ist:

| E | W | M | L | J | F | ... |
|---|---|---|---|---|---|-----|
| 7 | 5 | 3 | 3 | 2 | 2 | ... |

Wenn wir mit der obigen Cäsar-Chiffre anfangen, sehen wir, dass der Text offenbar so nicht verschlüsselt wurde.  Stattdessen modifizieren wir das Analyseprogramm wie folgt:
```kotlin
  fun main() {
    print("Analyse einer Substitutions-Chiffre\n  Geben Sie die verschlüsselte Nachricht ein: ")
    val encrypted = readlnOrNull()?.trim()?.uppercase(Locale.getDefault())
    if (encrypted.isNullOrEmpty())
      return
    val sanitized = encrypted.filter { it in 'A'..'Z' }
    val statistics = computeStatistics(sanitized)
    println("Die Häufigkeiten der Buchstaben sind: "+ statistics.joinToString{ it.first+"%.1f%%".format(it.second*100)})
    val decoder = mapOf(Pair(statistics[0].first, 'E'), Pair(statistics[1].first, 'N')).toMutableMap()
    while (true) {
      val decrypted = sanitized
          .map { decoder[it] ?: '.' }
          .joinToString("")
      println("Encrypted message:          $sanitized")
      println("Partially decryped message: $decrypted")
      print("Guessed some letter [encrypted decrypted]: ")
      val pair = readlnOrNull()?.trim()?.split("\\s+".toRegex())
      if (pair==null||pair.size!=2)
          return
      decoder[pair[0][0]] = pair[1][0]
    }
  }

  fun statistics(encrypted :String) = encrypted
    .filter { it in 'A'..'Z'}
    .groupBy { it }
    .map { Pair(it.key, it.value.size) }
    .normalize()
    .take(7)

  fun <E> List<Pair<E, Int>>.normalize() :List<Pair<E, Double>> {
    val total = sumOf { it.second }
    val f = if (total>0) 1.0/total  else 1.0
    return map { Pair(it.first, f*it.second) }
  }
```

Analog zum ersten Fall raten wir:

| ....E...I....N............ |
|----------------------------|
| ....E...M....W............ |

Dann sieht die teilweise entschlüsselte Nachricht wie folgt aus:

```log
  ..I...I...E..E..EN...NE.N.EI.EN.......EN
```

Wird es nun besser, wenn wir auch `'L' -> 'R'` ersetzen?

```log
  ..I...I...ER.E..EN.R.NERN.EI.EN.......EN
```

Hmm, es ist nicht sicher falsch, aber ob wirklich "NERN" in der geheimen Nachricht vorkommt, kann man nicht sagen.

Insgesamt können wir sagen, dass für kurze Texte die Analyse wahrscheinlich nicht den ganzen Code knackt.

Wenn die Texte aber länger werden, sieht es schon anders aus.

Wenn wir etwa die esten Abschnitte aus obigem Blog verwenden, dann ergibt die Statistik:

| E     | W     | M    | X    | F    | H    | K    |
|-------|-------|------|------|------|------|------|
| 17.5% | 11.4% | 8.8% | 6.4% | 6.4% | 5.9% | 5.7% |
|  E    |  N    |  I   |  ?   | R/T? |  ?   |  ?   |

und damit bekommt man schon mal einen Eindruck vom Text:

```log
WXAKCEUPMFSEJVJEHUXSCEWFHXXSIYFMJKULHQEW  N....E..I..E...E.....EN........I......EN
WEWIESEFWJKXZEWPYSSEWPMFCMEHUXSEJPXHXSSI  NEN.E.E.N....EN....EN.I..IE....E........
EUEMWEFZEFQFTBJYIFXBKMEHBFEAKEWPEFSMEZEF  E.EINE..E............IE...E..EN.E..IE.E.
EMWQLFVEHZLAKSMEHJCEUHEMEUBGYKSEWXZELJES  EIN....E......IE...E..EIE......EN..E..E.
HBXAKEFQFTBJYSYIMEEMWEEMWGKFLWIMWCMEPMHH  .....E..........IEEINEEIN....N.IN.IE.I..
EWHAKXGJNYUNEFHAKSHHESWNEFZEFIEWLWCNEFKE  EN..........E.......E.N.E..E..EN.N..E..E
MUSMAKEWJXHAKEWZLAKWYNEUZEFPEWWXSMAEZYZE  I..I..EN.....EN....N..E..E..ENN..I.E...E
MWEIEKEMUEWXAKFMAKJHAKMAQEWPMSSVLWAKHJEM  INE.E.EI.EN....I......I..EN.I....N....EI
WUXSPYFLUIEKJEHEMIEWJSMAKZEMQFTBJYIFXBKM  N.........E..E.EI.EN..I...EI...........I
EXSHJEMSIEZMEJCEFUXJKEUXJMQMWGYFUXJMQPMF  E....EI..E.IE..E.....E...I.IN......I..I.
KXZEWHLZOEQJECMEIEKEMUEMWGYFUXJMYWEWZEFE  ...EN....E..E.IE.E.EI.EIN......I.NEN.E.E
MWEWGGEWJSMAKEWQXWXSXLHJXLHAKEWPYSSEWJFX  INEN..EN..I..EN..N...........EN....EN...
CMJMYWESSKEMHHEWCMEHEZEMCEWXSMAELWCZYZXS  .I.I.NE...EI..EN.IE.E.EI.EN..I.E.N......
HYXLWCZXUHMAKEFHJEWPFEEHPEWWXSMAELWCZYZH  ....N.....I..E...EN..EE..ENN..I.E.N.....
MAKMUUEFPMECEFMWEMWEUBFMNXJEWFXLUJFEGGEW  I..I..E..IE.E.INEINE...I...EN......E..EN
QWWJEWMWCEUHMEWMEUXWCZESXLHAKEWQXWWLWCHM  .NN.ENIN.E..IENIE..N..E......EN..NN.N..I
EZESMEZMIYGJWXAKFMAKJEWKMWLWCKEFFEMAKEWQ  E.E.IE.I....N....I...EN.IN.N..E..EI..EN.
WWJEWXZEFNMESSEMAKJPYSSEWHMEXLAKWMAKJCXH  NN.EN..E..IE..EI.......EN.IE....NI......
HMFIEWCOEUXWCEFGKFJCXHHHMEUMJEMWXWCEFQYU  .I..EN..E..N.E..........IE.I.EIN.N.E....
ULWMVMEFEWCKPMFIEKEWCXNYWXLHCXHHHMEWLFZE  ..NI.IE.EN...I..E.EN....N........IEN...E
FCMHJXWVLWCZEFGGEWJSMAKEMWHEKZXFEQXWSEQY  ..I...N..N..E...EN..I..EIN.E....E..N.E..
UULWMVMEFEWQWWEWQWWEWHMECXHEFFEMAKEWLWCP  ...NI.IE.EN.NNEN.NNEN.IE...E..EI..EN.N..
EWWOXPMECXVLPYSSEWPMFVLWAKHJCMWIELWJEFJE  ENN...IE........EN.I...N.....IN.E.N.E..E
MSEWEMWENEFHAKSHHESJEWXAKFMAKJEMWJERJEMW  I.ENEINE.E.......E..EN....I...EIN.E..EIN
```

## 3.3 Angriff mit Klartext

Wenn du bemerkt hast, dass ich viele Beiträge mit den Worten "Nachdem wir letztes Mal ... haben, wollen wir heute ..." anfange, kannst du in obigem Text weitere Buchstaben raten, z.B. dass `'X' -> 'A'`, `'A' -> 'C'`, `'K' -> 'H'`, `'C' -> 'D'`, `'U' -> 'M'`, ...

```log
WXAKCEUPMFSEJVJEHUXSCEWFHXXSIYFMJKULHQEW  NACHDEM.I..E...E.MA.DEN..AA....I.HM...EN
WEWIESEFWJKXZEWPYSSEWPMFCMEHUXSEJPXHXSSI  NEN.E.E.N.HA.EN....EN.I.DIE.MA.E..A.A...
EUEMWEFZEFQFTBJYIFXBKMEHBFEAKEWPEFSMEZEF  EMEINE..E.........A.HIE...ECHEN.E..IE.E.
EMWQLFVEHZLAKSMEHJCEUHEMEUBGYKSEWXZELJES  EIN....E...CH.IE..DEM.EIEM...H.ENA.E..E.
HBXAKEFQFTBJYSYIMEEMWEEMWGKFLWIMWCMEPMHH  ..ACHE..........IEEINEEIN.H..N.INDIE.I..
EWHAKXGJNYUNEFHAKSHHESWNEFZEFIEWLWCNEFKE  EN.CHA....M.E..CH...E.N.E..E..EN.ND.E.HE
MUSMAKEWJXHAKEWZLAKWYNEUZEFPEWWXSMAEZYZE  IM.ICHEN.A.CHEN..CHN..EM.E..ENNA.ICE...E
MWEIEKEMUEWXAKFMAKJHAKMAQEWPMSSVLWAKHJEM  INE.EHEIMENACH.ICH..CHIC.EN.I....NCH..EI
WUXSPYFLUIEKJEHEMIEWJSMAKZEMQFTBJYIFXBKM  NMA.....M.EH.E.EI.EN..ICH.EI........A.HI
EXSHJEMSIEZMEJCEFUXJKEUXJMQMWGYFUXJMQPMF  EA...EI..E.IE.DE.MA.HEMA.I.IN...MA.I..I.
KXZEWHLZOEQJECMEIEKEMUEMWGYFUXJMYWEWZEFE  HA.EN....E..EDIE.EHEIMEIN...MA.I.NEN.E.E
MWEWGGEWJSMAKEWQXWXSXLHJXLHAKEWPYSSEWJFX  INEN..EN..ICHEN.ANA.A...A..CHEN....EN..A
CMJMYWESSKEMHHEWCMEHEZEMCEWXSMAELWCZYZXS  DI.I.NE..HEI..ENDIE.E.EIDENA.ICE.ND...A.
HYXLWCZXUHMAKEFHJEWPFEEHPEWWXSMAELWCZYZH  ..A.ND.AM.ICHE...EN..EE..ENNA.ICE.ND....
MAKMUUEFPMECEFMWEMWEUBFMNXJEWFXLUJFEGGEW  ICHIMME..IEDE.INEINEM..I.A.EN.A.M..E..EN
QWWJEWMWCEUHMEWMEUXWCZESXLHAKEWQXWWLWCHM  .NN.ENINDEM.IENIEMAND.E.A..CHEN.ANN.ND.I
EZESMEZMIYGJWXAKFMAKJEWKMWLWCKEFFEMAKEWQ  E.E.IE.I....NACH.ICH.ENHIN.NDHE..EICHEN.
WWJEWXZEFNMESSEMAKJPYSSEWHMEXLAKWMAKJCXH  NN.ENA.E..IE..EICH.....EN.IEA.CHNICH.DA.
HMFIEWCOEUXWCEFGKFJCXHHHMEUMJEMWXWCEFQYU  .I..END.EMANDE..H..DA...IEMI.EINANDE...M
ULWMVMEFEWCKPMFIEKEWCXNYWXLHCXHHHMEWLFZE  M.NI.IE.ENDH.I..EHENDA..NA..DA...IEN...E
FCMHJXWVLWCZEFGGEWJSMAKEMWHEKZXFEQXWSEQY  .DI..AN..ND.E...EN..ICHEIN.EH.A.E.AN.E..
UULWMVMEFEWQWWEWQWWEWHMECXHEFFEMAKEWLWCP  MM.NI.IE.EN.NNEN.NNEN.IEDA.E..EICHEN.ND.
EWWOXPMECXVLPYSSEWPMFVLWAKHJCMWIELWJEFJE  ENN.A.IEDA......EN.I...NCH..DIN.E.N.E..E
MSEWEMWENEFHAKSHHESJEWXAKFMAKJEMWJERJEMW  I.ENEINE.E..CH...E..ENACH.ICH.EIN.E..EIN
```

Jetzt kommt man dem Originaltext schon näher.


# 4. Polychiffre und wie die Enigma geknackt wurde
## 4.1 Was sind Polychiffre?
Wir haben gesehen, dass eine Cäsar-Chiffre recht leicht geknackt werden kann, wenn man nur wenigstens einen (durchschnittlichen) verschlüsselten Satz hat. Die Substitutionschiffre ist bei längeren Texten auch nicht wirklich sicher.  Da hatten wir mit statistischen Untersuchungen der Zeichenhäufigkeiten die wichtigsten Zeichen erraten können.  Außerdem kann man auch recht schnell ans Ziel gelangen, wenn man einen Teil der geheimen Nachricht kennt, z.B. eine einführende Parole.

Beides kann man umgehen, wenn man nicht im ganzen Text nur eine Chiffre verwendet, sondern die Chiffre häufig ändert.  Das kann man etwa erreichen, wenn man einen zweiten, kanonischen Text verwendet und dessen Buchstaben als den Shift der Cäsar-Chiffre intepretiert.  Der Unterschied zur Substitutionschiffre ist nun, dass der häufigste Buchstabe 'E' nicht immer durch das gleiche Zeichen ersetzt wird, sondern durch ein häufig wechselndes Zeichen.  Damit ist diese Kodierung gegen statistische Untersuchungen gefeit.

## 4.2 Wie sieht ein solcher Algorithmus aus?

Wir können das in einem Kotlin-Programm wie folgt implementieren:
```kotlin
  private val letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

  fun polycrypt(message :String, nonce :String) :String = message
    .uppercase(Locale.getDefault())
    .filter { it in 'A'..'Z' }
    .mapIndexed { i, c -> letters[(c-'A'+(nonce[i%nonce.length]-'A'))%26]  }
    .joinToString("")
```

Wie bisher auch verwandeln wir den zu verschlüsselnden Text zunächst in Großbuchstaben und filtern alle nicht-Buchstaben heraus.  Dann ersetzen wird das Zeichen `c` durch `letters[(c-'A'+shift)%26]` wobei der `shift` sich als `nonce[i%nonce.length]-'A'` ergibt.  Wenn der Hilfttext (engl. nonce) mindestens so lang wie die zu verschlüsselnde Nachricht ist, dann hat das `%nonce.length` keine Wirkung.  Falls der kanonische Text aber kürzer ist, wiederholen wir ihn einfach so oft, bis die ganze Nachricht damit abgedeckt ist.  Um es nochmal explizit zu sagen `.mapIndexed { i, c -> ... }` ersetzt die einzelnen Zeichen, wobei `c` das zu ersetzende Zeichen ist und `i` die Position dieses Zeichens, beginnend bei 0.  Das `.joinToString("")` solltest du schon kennen, es fügt die Liste von Zeichen wieder zu einem String zusammen.

Die Funktion kann man in einem Programm wie folgt aufrufen:
```kotlin
  private val nonce = """Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
  | incididunt ut labore et dolore magna aliqua. Mi bibendum neque egestas congue quisque egestas. Potenti
  | nullam ac tortor vitae purus faucibus ornare suspendisse sed. Morbi tristique senectus et netus et. Eu
  | tincidunt tortor aliquam nulla facilisi cras. Sapien pellentesque habitant morbi tristique senectus et.
  | Feugiat in fermentum posuere urna nec tincidunt praesent semper feugiat. Praesent tristique magna sit
  | amet. In iaculis nunc sed augue lacus viverra vitae congue eu. Consectetur adipiscing elit duis
  | tristique. Arcu bibendum at varius vel. Vestibulum rhoncus est pellentesque elit ullamcorper dignissim.
  | Molestie nunc non blandit massa enim nec dui nunc mattis. Sociis natoque penatibus et magnis. Mauris a
  | diam maecenas sed enim. Orci a scelerisque purus semper. Et odio pellentesque diam volutpat commodo sed
  | egestas egestas fringilla. In iaculis nunc sed augue lacus viverra vitae. Etiam non quam lacus
  | suspendisse faucibus interdum posuere lorem. Eget mauris pharetra et ultrices neque ornare aenean
  | euismod.""".trimMargin()
    .uppercase(Locale.ITALIAN).filter { it in 'A'..'Z' }

  fun main() {
    print("Polychiffe\n  Bitte geben Sie den zu verschlüsselnden Text ein: ")
    val message = readlnOrNull()?.trim()
    val encrypted = polycrypt(message, nonce)
    println("Die verschlüsselte Nachricht lautet: $encrypted")
  }
```
Der eigentliche Aufruf ist recht einfach.  Die spannende Frage bleibt, woher man den Hilfstext bekommt.  Ich habe hier einen traditionellen Text aus dem Schriftsatz genommen, er heißt "Lorem Ipsum", weil er mit diesen Worten anfängt.  Es soll Latein sein, deshalb habe ich den Standard (`Locale`) italienisch verwendet, was dem Lateinischen vielleicht am nächsten kommt.  Das besondere an diesem Text ist, dass alle (lateinischen) Buchstaben darin vorkommen und auch die meisten Ligaturen (also Doppelbuchstaben, die in einem guten Schriftsatz verschmolzen werden).  Du solltest natürlich für deine Verschlüsselung einen eigenen Text wählen, z.B. den Anfang von deinem Lieblingsbuch oder auch einen selbstgeschriebenen Text.

Zum Entschlüsseln braucht man nur das "Vorzeichen" vom `shift` umdrehen, etwa so
```kotlin
  fun polydecrypt(encrypted :String, nonce :String) = encrypted
    .uppercase(Locale.getDefault())
    .filter { it in 'A'..'Z' }
    .mapIndexed { i, c -> letters[(c-'A'+26-(nonce[i%nonce.length]-'A'))%26] }
    .joinToString("")
```

## 3.3 Wie sicher ist das?

Nun das hängt vor allem davon ab, wie geheim du den Hilfstext halten kannst.  Solange der Hilfstext noch nicht erraten wurde, kann man aus einer unbekannten abgefangenen Nachricht nicht durch statistische Analyse die gemeine Original-Nachricht erschließen.

Schauen wir uns das am anfänglichen Geheimtext an.  Dieser lautet verschlüsselt:

> EFZJ RUXU BTHI ESRT MGDG QGGI ATIK WIGJ IOSG AIWP

Wenn man jetzt die statistische Analyse darüber laufen lässt, ergibt sich:
```log
  I: 15,0%, G: 15,0%, T: 7,5%, E: 5,0%, J: 5,0%, R: 5,0%, U: 5,0%
  Partially decryped message: .........I.E...I.N.N.NNE.IE..EN.E..N.E..
```

Offenbar gibt es noch keine Gleichverteilung der verschlüsselnden Buchstaben, aber wenn man glaubt, dass das häufigste Zeichen für 'E' und das zweithäufigste für 'N' steht, dann kommt man zu keinem guten Ergebnis.

Ein weiterer Vorteil ist auch, dass selbst, wenn die Nachricht immer mit der gleichen Parole anfängt, dann kann man im Klartext-Angriff nur die ersten Buchstaben aus dem Hilfstext erraten.  Wenn es ein einfacher Text ist, kann man diesen vielleicht an den Anfangsworten erraten, z.B. falls man erkennt, dass er mit "Lorem ipsum" losgeht. Wenn wir wieder mit den Paragrafen vom Anfang starten, dann ergibt sich der verschlüsselte Text:

> YOTLPMBOCDOSENKWAFAXHXPF
> FSENZSKCKHPCHSWPVRTKPTXJ

Wenn man jetzt mit dem Klartext "Nachdem wir letztes Mal" vergleicht,

```kotlin
  fun main() {
    print("Polychiffre analysieren\n  Bitten geben Sie den verschlüsselten Text ein: ")
    val encrypted = readlnOrNull()?.trim()
    if (encrypted.isNullOrEmpty())
      return
    print("Bitte geben Sie den Anfang vom vermuteten Klartext ein: ")
    val fragment = readlnOrNull()?.trim()
    if (fragment.isNullOrEmpty())
      return
    val nonce = polyAnalyze(encrypted, fragment)
    print("Der Anfang vom Hilfstext lautet: $nonce")
  }

  fun polyAnalyze(encrypted :String, fragment :String) = encrypted
    .uppercase(Locale.getDefault())
    .filter { it in 'A'..'Z' }
    .zip(fragment.uppercase(Locale.getDefault()).filter { it in 'A'..'Z' })
    .map { (e, f) -> letters[(e-'A'+26-(f-'A'))%26] }
    .joinToString("")
```

erhält man einen Hilfstext von:
```log
  LOREMIPSUMDOLORSITAM
```

Wenn man jetzt im Internet sucht, kann man den Text "Lorem Ipsum" finden und damit einen guten Teil des Hilfstextes raten.

Die Sicherheit ist also nur so gut, wie man einen geheimen Hilfstext findet.


## 3.4 Was war die Enigma?

Ein Problem an den Polychiffren ist offenbar, dass man einen einigermaßen langen aber geheimen Text finden muss.  Eine Alternative ist es, statt eines lesbaren Textes die Shifts von einem Algorithmus ausrechnen zu lassen.  Dieser Algorithmus wird dann ab und zu mit einem neuen Schlüssel versorgt.  Genau das hatten die Hersteller der Enigma getan.

Die Enigma war eine Chiffriermaschine für die geheime Kommunikation der Wehrmacht vor und im 2. Weltkrieg.  Es gab auf jedem Schlachtkreuzer und in jedem U-Boot eine, sowie natürlich in der OHL.  Der elektromechanische Mechanismus zur Chiffrierung bestand aus zunächst 2, später 3 Chiffrierzahnrädern, die jeweils eine Cäsar-Chiffre ausführten, nach einem festen Algorithmus den Shift aber bei jeder Übersetzung änderten.  An der Maschine wurde täglich ein neuer Startcode eingestellt.

## 3.5 Wie konnte die Enigma geknackt werden?

Die Briten (Polen und Amerikaner) waren natürlich an der Verschlüsselung und dem Knacken des Codes interessiert.  Sie hatten Glück und bereits am Anfang des Krieges eine Enigma erbeutet.  Aber der Mechanismus und auch das Knacken der Codes waren nicht einfach.  Zum Glück hatten sie ein junges Talent namens [Alan Turing](https://de.wikipedia.org/wiki/Alan_Turing), der viel von Mathematik und Logik verstand, sowie ein passionierter Kryptoanalytiker war.  Er entwarf zusammen mit anderen auf den Ideen der Polen und seinen eigenen Ideen von Berechenbarkeit einen Automaten ([the Bomb](https://de.wikipedia.org/wiki/Turing-Bombe)), mit dem man im Prinzip die Verschlüsselung der Enigma knacken konnte.  Es gab nur 2 Probleme:  Zum einen musste man hinreichend viele Funksprüche abfangen und zum anderen dauerten die Berechnungen einigermaßen lange.  Am Anfang reichte die Rechenzeit nicht aus, um innerhalb eines Tages den Code zu knacken, und die Nazis wechselten den Code jeden morgen.  Dann aber entdeckten die Briten einen Trick.  Die gründlichen Deutschen starteten jeden verschlüsselten Funkspruch mit der Enigma mit einer Standardparole, der Uhrzeit und dem Wetterbericht. Diese Daten waren mehr-oder-weniger frei verfügbar.  Somit hatte man einen Klartext, um die Analyse zu beschleunigen.  Damit gelang es, den Code schnell genug zu knacken.

Leider war der Krieg damit noch nicht gewonnen und das größte Risiko für die britische Spionage war, dass die Nazis entdeckten, dass ihre Enigma geknackt wurde und sie auf stärkere Verschlüsselung setzen mussten.  Das Ergebnis war eine ethisch grenzwertige Sache, dass nämlich nicht gegen alle Operationen der Nazis unmittelbar vorgegangen wurde.  So konnten die Nazis ein amerikanisches Passagierschiff versenken, obwohl die Briten den Funkspruch zum Angriff empfangen und entschlüsselt hatten.  Stattdessen wurden die entschlüsselten Funksprüche genutzt, um genügend viele kriegsentscheidende Schlachten zu gewinnen.  Aus Sicht der Nazis verloren sie zwar häufiger, aber schlossen noch nicht, dass das am Entschlüsseln ihrer geheimen Funksprüche lag.  Wer die genaueren Hergänge und das geheime Programm der Briten zur Entschlüsselung der Enigma wissen will, sollte die Filme [Enigma -- Das Geheimnis (2001)](https://de.wikipedia.org/wiki/Enigma_%E2%80%93_Das_Geheimnis) und [The Imitation Game (2014)](https://de.wikipedia.org/wiki/The_Imitation_Game_%E2%80%93_Ein_streng_geheimes_Leben) schauen oder die Bücher dazu lesen.


# 4. Warum sollte RSA sicherer sein?

Wenn du dich an den RSA-Algorithmus vom letzten Mal erinnerst, dann stellt sich die Frage, ob der wirklich sicherer als etwa die Enigma ist.  Dazu sollte man folgende Verbesserungen erkennen:

1. Für den RSA-Algorithmus ist der Schlüssel typischer Weise 512--4096 bit lang -- Das ist mehr als die Codes, die in der Enigma konfiguriert wurden.

2. Der RSA-Algorithmus verschlüsselt Bytegruppen, nicht nur einzelne Buchstaben -- Was ich beim letzten Mal nicht geschrieben habe ist, dass man mit dem PGP-Programm nicht einfach jeden Buchstaben einzeln verschlüsselt, sondern immer so viel Bytes auf einmal zusammen, wie der Modulus hergibt.  Daraus ergibt sich, dass man nicht einfach nur 26 Ersetzungen erraten muss, sondern z.B. $2^{4096}$ -- das sind zu viele, um sie in einer einfachen Tabelle zu speichern.

3. Der RSA-Algorithmus verschlüsselt asymmetrisch -- Im Gegensatz zur Cäsar-Chiffre und auch zur Poly-Chiffre kann man die Entschlüsselung nicht einfach aus der Verschlüsselung gewinnen.  Das liegt daran, dass in der Faktorisierung $e\cdot f \equiv 1 \pmod{\phi(n)}$ die Exponenten $e$ und $f$ zwar gleichwertig eingehen, man aber bei Unkenntnis von $\phi(n)$ nicht aus dem einen auf den anderen schließen kann.  Ein Risiko gibt es aber bei der üblichen Verwendung:  Da der Exponent der öffentlichen Verschlüsselung immer 65537 ist, kann man aus dem privaten Schlüssel (der aus $f$ und aus $n$ besteht), den öffentlichen Schlüssel leicht rekonstruieren.


# 5. Digitale Signaturen

Kann man außer der Übermittlung geheimer Nachrichten auch noch andere Dinge tun, z.B. mit RSA?

Ja, zum Beispiel eMail digital signieren.  Während man auf Papier eine eigene Unterschrift so unter einen Brief schreiben kann, dass sie nur schwer zu fälschen ist, ist das bei digitalen Nachrichten schon schwieriger.  Wenn der Angreifer einmal eine Datei mit deiner gescannten Unterschrift erhalten hat, dann kann er die natürlich an jede Nachricht, die er für authentisch erklären will, anfügen.

Die Lösung ist eine digitale Unterschrift.  Aber lass uns zunächst klären, wie das Prozedere dazu aussieht:

1. Man entwirft eine Nachticht (digital), die authentisch sein soll.
2. Man produziert eine kaum fälschbare Unterschrift, in der auf die Nachricht Bezug genommen wird.
3. Man publiziert die Nachricht mit der Unterschrift.
4. Jeder kann die Nachricht lesen, aber durch Entschlüsseln der Unterschrift kann man sich überzeugen, dass nur die Original-Nachricht unterschrieben wurde.

Dabei sind offenbar 2 Dinge wichtig: zum einen, dass jeder die Unterschrift überprüfen kann und zum anderen, dass niemand die Unterschrift (mit ihrem Bezug) zur Nachricht fälschen kann.

Wie kann man das mit RSA tun?

Ganz einfach, man kann z.B. die gesamte Nachricht mit dem privaten Schlüssel verschlüsseln und dann kann jeder die verschlüsselte Nachricht mit dem öffentlichen Schlüssel entschlüsseln, aber niemand (außer dir selbst) eine andere Nachricht verschlüsseln (denn keiner hat den privaten Schlüssel).  Der Grund, dass es egal ist, ob man erst mit dem privaten Schlüssel verschlüsselt und dann mit dem öffentlichen Schlüssel entschlüsselt oder erst mit dem öffentlichen Schlüssel verschlüsselt und dann mit dem privaten Schlüssel entschlüsselt, ist dass $(b^e)^f\equiv b \equiv (b^f)^e \pmod n$.  Das ist eine der Eigenschaften dieser asymmetrischen Verschlüsselung.  Mit einem symmetrischen Schlüssel kann man das nicht hinbekommen.

## 5.2 Anwendung: Einzellizenz für Software

Man kann dieses Prinzip auch anwenden, um Software für einzelne Kunden zu lizensieren.  Dazu muss man zunächst sicherstellen, dass man den/die Rechner des Kunden sicher identifizieren kann.  Ein Beispiel in Assembler/C ist, die ID der CPU auszulesen.  Diese ist für jede hergestellte CPU verschieden und kann, wenn das Programm wirklich auf der CPU ausgeführt wird, nicht gefälscht werden.  Die Nachricht, die signiert wird, ist also die ID des Rechners, für den das Programm bezahlt wurde.  Entsprechend braucht man einen geheimen (und einen öffentlichen) Schlüssel.

Der 2., ebenfalls trickreiche Teil ist, dass das Programm zur Ausführungszeit diese Signatur überprüfen muss und entsprechend die Ausführung verweigern muss, falls der Schlüssel nicht zum Rechner passt. In einem C oder Rust-Programm kann man das ganz gut sicherstellen, besonders, wenn man das Debuggen und Disassemblieren/Decompilieren verhindert.  Leider kann man das bei JVM- oder JS-Programmen nicht so sicher verhindern, da sich deren ausführbarer Code im Prinzip wieder rückübersetzen und auch debuggen lässt.  Mit mehr Aufwand kann man das erschweren.

Im Prinzip ist es eine Abwägung zwischen der Sicherheit der Lizenz und dem Aufwand, den man zur Sicherung betreiben will.  Zusätzlich absichern kann man das ganze, indem man die Lizenz nur für einen begrenzten Versionsbereich der Software erteilt (was dann in der entschlüsselten Nachricht ebenfalls 'drinsteht).  Wenn man oft genug Updates herausbringt, dann müssen die Nutzer regelmäßig neue Schlüssel beantragen und ein eventuelles Fälschen eines alten Schlüssels bringt nicht lange genug einen Vorteil.

Die Alternative ist, dass man die Benutzung der Software/Bibliothek kontrolliert, z.B. wenn diese per Internet sich beim Hersteller meldet (zusammen mit dem registrierten Nutzer und der ID des Rechners).  Falls dann zu viele Instanzen den gleichen registrierten Nutzer oder Lizenz melden, kann das Programm die Ausführung verweigern oder man kann entsprechend Abmahnungen an die Nutzer schreiben.  Im Firmenbreich oder für qualitativ hochwertige Software ist das sicherlich sinnvoll, besonders, wenn die Softwarefirma viel Zeit in die Entwicklung und das Design der Software gesteckt hat.

Umgekehrt ist es jedoch nicht akzeptabel, wenn öffentliche Nachrichten, z.B. eine eMail einer Behörde an Nutzer, nur mit proprietären Programmen (also für die man Geld bezahlen muss), lesbar und nutzbar sind.  Dagegen gibt es zum einen offene Standards, etwa die meisten Programmiersprachen, Markdown, HTML 5, DOCX, PDF und noch einige andere.  Und zum anderen gibt es freie Implementierungen von Standard Office-Software, z.B. Libre Office.  Natürlich kann man argumentieren, dass Briefe in Microsoft Word immer etwas schöner aussehen als im Libre Office, aber die Frage ist halt auch, ob ein privater Nutzer wirklich Microsoft bezahlen muss, um eine behördliche eMail zu lesen oder ein Formular auszufüllen.

# Weiterführende Literatur

[2] [Spektrum der Wissenschaft, Januar '23](https://www.spektrum.de/kolumne/kryptografie-der-algorithmus-der-die-nsa-in-den-wahnsinn-trieb/2101827)
