
# 1. Schritt: IntelliJ für Windows 64 herunterladen und installieren

Als Entwicklungsumgebung für Kotlin empfehle ich IntelliJ Idea Kommunity-Version.  Diese kann man von der Website von [JetBrains](https://www.jetbrains.com/idea/download/#section=windows) herunterladen, wichtig ist es, auf den schwarzen Knopf (Community Version) zu klicken.

# 2B. Schritt: Java 11 Development Kit installieren

Das sollte aus der IntelliJ IDE heraus gehen: Datei > Projektstruktur > Projekt > SDK > Download JDK

Im Dialog: Version 11, Oracle Open JDK auswählen.  Der Pfad sollte automatisch ausgefüllt sein, ansonsten kann man C:\Programme\Java angeben.

Falls der Punkt "Projektstruktur" im Datei-Menü nicht wählbar ist, muss man ein

# 2B. Neues Projekt anlegen

Datei > New ... > Projekt ...:

Name: Hello World

Location: -- kann so bleiben, besser merken, dort liegen dann der Quellcode und die temporären Projektdateien

links Kotlin auswählen.

Project template: Console Application

Build system: Gradle Kotlin

Projekt JDK:  hier die gleichen Einstellungen wie oben, d.h. Download JDK ...

Wenn ihr ein 2. Projekt anlegt, könnt ihr natürlich das bereits installierte Open JDK 11 verwenden.

Artifact Coordinates:

group id: org.grutzmann

artifact ID: helloworld

Version: 0.1-SNAPSHOT

Dann auf weiter klicken.

Target JVM version: 11

test framework: JUnit 5

und dann Fertig wählen.

Das bewirkt 2 Dinge: 1. Dass ein Java Development Kit heruntergeladen wird und 2. dass ein leeres Kotlin Projekt angelegt wird.

In diesem Kotlin-Projekt könnt ihr jetzt das Beispiel [01 for Schleifen](../01loops/) ausprobieren (oder ein anderes).
