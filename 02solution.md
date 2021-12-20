
## Abfrage eines Namens und Auswertung, ob der weiblich ist.

```Kotlin
  fun main() {
    ...
    print("Geben Sie einen Vornamen ein: ")
    val line = readLine() ?: return
    val name = line.trim()
    if (name.isEmpty())
      return
    if (isFemale(name))
      println("$name ist weiblich.")
    else
      println("$name ist männlich.")
  }
```

Wie zwischendurch angedeutet, darf der Name keine Leerzeichen oder das Zeilenende-Zeichen enthalten.  Dazu entfernt `.trim()` alle Leerzeichen an Anfang und Ende.
