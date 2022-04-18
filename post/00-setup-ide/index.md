
# 1. Schritt: IntelliJ für Windows 64 herunterladen und installieren

Als Entwicklungsumgebung für Kotlin empfehle ich IntelliJ Idea Kommunity-Version.  Diese kann man von der Website von [JetBrains](https://www.jetbrains.com/idea/download/#section=windows) herunterladen, wichtig ist es, auf den schwarzen Knopf (Community Version) zu klicken.

# 2A. Schritt: Java 11 Development Kit installieren

Das sollte aus der IntelliJ IDE heraus gehen: Datei > Projektstruktur > Projekt > SDK > Download JDK

Im Dialog: Version 11, Oracle Open JDK auswählen.  Der Pfad sollte automatisch ausgefüllt sein, ansonsten kann man C:\Programme\Java angeben.

Falls der Punkt "Projektstruktur" im Datei-Menü nicht wählbar ist, muss man ein

# 2B. Neues Projekt anlegen

Datei > Neu ... > Projekt ...:

Name: Hello World

Location: -- kann so bleiben, besser merken, dort liegen dann der Quellcode und die temporären Projektdateien

Kotlin Projekt auswählen.

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

# 3. Programm von der Kommandozeile (oder aus dem Explorer) startbar machen

Dazu muss ein "fat jar", d.h. mit Bibliotheken erzeugt werden.  Wenn ihr im Internet nach "Gradle Kotlin create fat jar" sucht, dann findet ihr die aktuelle Anleitung, ansonsten hier die derzeitige Version (2022 mit Gradle 7.x, Kotlin 1.6.x und JVM 11+):

1. Die Datei "build.gradle.kts" öffnen,
2. unter `tasks.withType<KotlinCompile>() {}` das `jvmTarget` von "1.8" auf "11" ändern (geht auch für neuere JVM-Versionen noch);
3. am Ende der Datei die 2 Blöcke ergänzen:
```kotlin
  application {
    mainClassName = "MainKt"
  }

  tasks {
    val fatJar = register<Jar>("fatJar") {
      dependsOn.addAll(listOf("compileJava", "compileKotlin", "processResources"))
      archiveClassifier.set("full") // suffix for the naming the jar
      duplicatesStrategy = DuplicatesStrategy.EXCLUDE
      manifest { attributes(mapOf("Main-Class" to application.mainClass)) }
      val sourcesMain = sourceSets.main.get()
      val contents = configurations.runtimeClasspath.get()
          .map { if (it.isDirectory) it else zipTree(it) } +
              sourcesMain.output
      from(contents)
    }
    build {
      dependsOn(fatJar) // Trigger fat jar creation during build
    }
  }
```

4. Falls `application` rot dargestellt wird, müsst ihr noch das Plugin "application" aktivieren:  Dazu ganz oben im Block
```kotlin
  plugins {
    kotlin("jvm") version "1.6.20"
    application
  }
```
ergänzen und anschließend das Script neu importieren (auf den blauen Kreis im Editorfenster clicken).

Wenn sich das Programm aus der IDE starten lässt, dann könnt ihr mit
```bash
  ./gradlew fatJar
```

das Programm erzeugen: "build/libs/helloworld-0.1-SNAPSHOT-full.jar"  Das sollte mehr als 1MB groß sein.  Wenn eine JVM≥11 installiert ist, dann kann man das ausführen, z.B. aus dem Explorer/Finder oder auf der Kommandozeile:
```bash
  java -jar helloworld.jar
```
wenn ihr diese Datei vorher unter dem Namen "helloworld.jar" gespeichert habt (und java im Pfad ist).

Es kann sein, dass man beim Start im Explorer/Finder die Kommandozeilenausgabe nicht sieht, d.h. ihr müsstet entweder eine grafische App schreiben, z.B. [11 Augen malen](../11eyes/) oder doch auf der Kommandozeile starten.
