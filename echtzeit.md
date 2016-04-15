---
documentclass: scrartcl
title: Softwaresysteme - Echtzeitsysteme (Prof. Dr. Klüver)
author: Susanne Kießling, \#948898
date: \today
---


Im Folgenden werden als Leistungsnachweis für das Fach Softwaresysteme -
Echtzeitsysteme, jeweils drei Fragen aus den Themengebieten der Vorlesung
(Definitionen und Grundbegriffe, Betriebssystem, Softwareentwicklung)
bearbeitet.



# Definitionen und Grundbegriffe

`1.) Machen sie an geeigneten Beispielen den Unterschied zwischen "embedded
system" und "Echtzeit System" zu Standard-Rechnern klar.`

Bei Echtzeitsystemen sind neben der Korrektheit der Ausgabewerte vorallem die
Zeiten, zu denen die Ausgabewerte verfügbar sind, von Bedeutung. Echtzeitsysteme
müssen ein definiertes Zeitverhalten garantieren. Ein Beispiel wäre die
Motorsteuerung im Auto. Wird die Echtzeit in diesem Fall nicht erfüllt, kommt es
beispielsweise zu einem Stottern des Motors.

Embedded-Systeme sind häufig auch Echtzeitsysteme, dies muss aber nicht zwingend
gegeben sein. In der Regel sind Embedded-Systeme für einen stabilen Dauerbetrieb
ausgelegt. Eingesetzt werden Embedded-Systeme beispielsweise in der Steuerung von
Haushaltsgeräten, wie z.B. Kühlschrank oder Waschmaschine.

Standard-Rechner sind eher für eine dynamische Anpassung bei wechselnder Last
ausgelegt. Es sollen verschiedene Problemstellungen effizient gelöst werden.
Für Rechenintensive Aufgaben, wie beispielsweise Rendering, ist der
Standard-Rechner prädestiniert.


`2.) Welche Besonderheiten zeichnen ein Echtzeit-Rechner-System aus? Erläutern
Sie dabei die Begriffe "Rechtzeitigkeit", "Harte Echtzeit - Weiche Echtzeit"
genauer!`

Ein Echtzeit-Rechner-System muss ein definiertes Zeitverhalten garantieren, d.h.
es muss die Garantie vorliegen, dass die Antwort auf eine Anfrage in einer
definierten, vorher berechneten Zeit geliefert wird.

Folgende Begriffe sollen in diesem Zusammenhang näher erläutert werden:

*Rechtzeitigkeit*: Die Zeit zwischen zwei Ereignissen darf eine bestimmte
Maximalzeit (Deadline) nicht überschreiten, es muss *rechtzeitig* ausgeführt
werden. Grundsätzlich muss die Zeit zur Beantwortung einer Anfrage garantiert
und vorher berechenbar sein.

*Harte Echtzeit*: Wird eine definierte Maximalzeit (Deadline) überschritten,
gilt dies als Versagen des Systems. Als Beispiel soll die Steuerung des Airbags
im Auto genannt werden. Das Echtzeitsystem darf hier weder zu früh, noch zu spät
reagieren.

*Weiche Echtzeit*: Die Maximalzeit der harten Echtzeit kann überschritten
werden. Die Folgen darauß sind sozusagen verschmerzbar. Es kann allerdings sein,
dass es zu einer geringeren Qualität der Antwort kommt. Als Beispiel wird häufig
ein Aussetzen oder Ruckeln des Bildes bei der Videotelefonie genannt.


`3.) Nennen Sie drei Beispiele für Einsätze von Echtzeitrechnern und stellen Sie
jeweils die Unterschiede zu einem Standard-Rechner dar.`


**Airbag-Steuerung:** 

**Antiblockiersystem im Auto:**

**Verkehrsampel-System:** Es muss gewährleistet werden, dass die Ampel
garantiert auf rot umschaltet, bevor die Ampeln der  anderen Verkehrsteilnehmer
auf grün wechselt. Auch zu frühes Umschalten würde Unfälle provozieren.





# Betriebssystem

`1.) Welche Rolle spielt UNIX im Prozeßrechnerbereich?`

Die meisten gebräuchlichen UNIX Systeme sind nicht echtzeitfähig (Linux,
BSD--Derivate). Mit QNX gibt es allerdings ein kommerzielles Unix das selbst
harte Echtzeit beherrscht. Es wird wegen seiner hohen Ausfallsicherheit sogar in
Atomkraftwergen eingesetzt.

`2.) Welche Eigenschaften sind für ein Echtzeit-Betriebssystem wünschenswert?`


* *Multitasking:* Mehrere Prozess können gleichzeitig ausgeführt werden. Die
  Systemaufrufe sollten dazu reentrant sein um nötigenfalls neu ausgeführt
  werden zu können.
* *Echtzeit Scheduler/Dispatcher:* Ein Task sollte via Preemption unterbrochen
  werden können um höher priorisierten Aufgaben den Vortritt zu lassen.
* Echtzeit-Speicherverwaltung: Speicherzugriffe erfolgen in definierter Zeit,
  beispielsweise ist dazu nötigenfalls der Caching Mechanismus abschaltbar.
* Echtzeit-Dateisystem: Schreiben/Lesen von Daten erfolgt in definierter Zeit,
  beispielsweise wird hierbei kein Caching von Dateimetadaten und Dateien
  durchgeführt, wie es auf Desktop System (vernünftigerweise) üblich ist.
* IPC mit ISR Unterstütung: Unterstützung von Messagequeues um Daten aus ISR
  zurückzugeben.

``3.) Was fehlt bei Standard-Linux zur Echtzeitfähigkeit? Was ist überflüssig? ``

Linux bietet kein Scheduler (bzw. Dispatcher) der Prozesse zu einem definierten
Zeitpunkt wieder ausführt. Die Zeit nach der ein Prozess weiterarbeitet hängt
von vielen internen Faktoren ab. Zudem ist es nicht garantiert, dass das System
unter Last Ereignisse entgegen nehmen kann.

Um weiche Echtzeit zu ermöglichen ist es möglich einen Preemptive Kernel zu
nutzen, der es dem Scheduler ermöglicht einen Kontextwechsel auch dann
durchzuführen wenn ein Treiber ausgeführt wird falls ein höher priorisierter 
Task vorliegt. 

Überflüssig beim Linux Kernel ist beispielsweise die Multiuser--Fähigkeit,
bzw. alles was auf typischen Embeddedsystemen keine Anwendung findet, da dies
nur zusätzlicher Ballast darstellt und eventuell unnötige Kontextwechsel verursacht.

# Softwareentwicklung

`1.) Wieso ist Pearl eine gute Programmiersprache für Echstzeitsysteme?`

PEARL bietet als Echtzeithochsprache eine komfortable und sichere Entwicklung
von Multitasking- und Echtzeitanwendungen. Da sie vom Deutschen Institut für
Normung standardisiert wurde, eignet sie sich zudem für den Lehrbetrieb und hält
keine ,,Versions-Überaschungen'' bereit wie andere Sprachen.

Da die Sprache prozessorunabhängig und speziell für den Einsatz zur
Echtzeitprogrammierung konzipiert wurde stellt sie oft eine bessere Alternative
da als beispielsweise ,,Echtzeit-Java'' das eine Menge Insiderwissen benötigt
und noch zusätzlich viele Stolpersteine mitbringt und auch dann oft nur wenig
zufriedenstellend funktioniert.

`2.) Warum ist die Entwicklung von Echtzeitsoftware aufwendiger als die anderer
Software? Kann man Echtzeitsoftware nur mit Assembler realisieren?`

Herkömmliche Programme können auf bestimmte Ereignisse beliebig lange warten und
können für bestimmte Abschnitte prinzipiell genauso beliebig lange brauchen.

Zwar ist auch bei einem normalen Rechner die ,,Deadline'' erreicht wenn man er
nach einer eigentlich kleinen Berechnung eingefroren ist, doch diese Art von
,,Echtzeit'' wird bereits von den meisten Programmen automatisch garantiert.

Echtzeitsoftware hingegen darf nur eine bestimme Zeit auf ein Ergebnis warten
und im Fehlerfall selbst eine Fehlerbehandlung ausführen. Zudem müssen bestimmte
Berechnungen und Aufgaben in einer definierten Zeiten erledigt werden um auch
eine ausreichend kleine Antwortzeit bei hoher Last zu gewährleisten.

Dies erhöht den Testaufwand der Software enorm, zumal die Ausführung des
Programms abhängig von vielen Faktoren ist, die nicht alle in einer
Simulationsumgebung aufgefangen werden können. Die Zielplattform kann zudem noch
Differenzen zur Simulationsumgebung haben, was zu schwierig nachvollziehbaren
Fehlern führen kann.

Echtzeitsoftware muss nicht zwangsweise in Assembler geschrieben werden --- und
sollte des Aufwands wegen lieber in einer geeigneten Hochsprache verfasst
werden. Sprache wie PEARL oder C bieten sich hier an, da sie eine hohe
Hardwarenähe bieten und keine Hinderungsgründe wie Garbage--Collection haben.

`3.) Welche Vorteile kann eine gute Echtzeitsprache für einen Entwickler bieten?
(Was wünschen Sie sich?)` 

* Kein Garbage Collector (und damit keine unkalkulierbaren Unterbrechungen im
  Programmablauf).
* Kein eigener Userspace--Scheduler (wie bei Golang).
* Nach Möglichkeit direkter Hardwarezugriff.
* Komfortable ISR Benutzung.
* Hochsprache um Probleme aus der realen Welt gut abbilden zu können.
* Prozessorunabhänigige Entwicklung möglich (nicht wie Assembler).
