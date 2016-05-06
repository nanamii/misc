---
documentclass: scrartcl
title: Softwaresysteme - Echtzeitsysteme (Prof. Dr. Klüver)
author: Susanne Kießling, \#948898
date: \today
toc: yes
---



# Aufgabenstellung

Im Folgenden werden als Leistungsnachweis für das Fach Softwaresysteme -
Echtzeit-Systeme, jeweils drei Fragen aus den Themengebieten der Vorlesung
(Definitionen und Grundbegriffe, Betriebssystem, Softwareentwicklung)
bearbeitet.



# Themengebiet: Definitionen und Grundbegriffe

**1.) Machen sie an geeigneten Beispielen den Unterschied zwischen "embedded
system" und "Echtzeit System" zu Standard-Rechnern klar.**

Bei Echtzeit-Systemen sind neben der Korrektheit der Ausgabewerte vorallem die
Zeiten, zu denen die Ausgabewerte verfügbar sind, von Bedeutung. Echtzeit-Systeme
müssen ein definiertes Zeitverhalten garantieren. Ein Beispiel wäre die
Motorsteuerung im Auto. Wird die Echtzeit in diesem Fall nicht erfüllt, kommt es
beispielsweise zu einem Stottern des Motors.

Embedded-Systeme sind häufig auch Echtzeit-Systeme, dies muss aber nicht zwingend
gegeben sein. In der Regel sind Embedded-Systeme für einen stabilen Dauerbetrieb
ausgelegt. Eingesetzt werden Embedded-Systeme beispielsweise in der Steuerung von
Haushaltsgeräten, wie z.B. Kühlschrank oder Waschmaschine.

Standard-Rechner sind eher für eine dynamische Anpassung bei wechselnder Last
ausgelegt. Es sollen verschiedene Problemstellungen effizient gelöst werden.
Für rechenintensive Aufgaben, wie beispielsweise Rendering, ist der
Standard-Rechner prädestiniert.


**2.) Welche Besonderheiten zeichnen ein Echtzeit-Rechner-System aus? Erläutern
Sie dabei die Begriffe "Rechtzeitigkeit", "Harte Echtzeit - Weiche Echtzeit"
genauer!**

Ein Echtzeit-Rechner-System muss ein definiertes Zeitverhalten garantieren, d.h.
es muss die Garantie vorliegen, dass die Antwort auf eine Anfrage in einer
definierten, vorher berechneten Zeit geliefert wird.

Folgende Begriffe sollen in diesem Zusammenhang näher erläutert werden:

**Rechtzeitigkeit**: Die Zeit zwischen zwei Ereignissen darf eine bestimmte
Maximalzeit (Deadline) nicht überschreiten, es muss *rechtzeitig* ausgeführt
werden. Grundsätzlich muss die Zeit zur Beantwortung einer Anfrage garantiert
und vorher berechenbar sein.

**Harte Echtzeit**: Wird eine definierte Maximalzeit (Deadline) überschritten,
gilt dies als Versagen des Systems. Als Beispiel soll die Steuerung des Airbags
im Auto genannt werden. Das Echtzeitsystem darf hier weder zu früh, noch zu spät
reagieren.

**Weiche Echtzeit**: Die Maximalzeit der harten Echtzeit kann überschritten
werden. Die Folgen darauß sind sozusagen verschmerzbar. Es kann allerdings sein,
dass es zu einer geringeren Qualität der Antwort kommt. Als Beispiel wird häufig
ein Aussetzen oder Ruckeln des Bildes bei der Videotelefonie genannt.


**3.) Nennen Sie drei Beispiele für Einsätze von Echtzeitrechnern und stellen Sie
jeweils die Unterschiede zu einem Standard-Rechner dar.**


**Airbag-Steuerung:** Das garantierte Zeitverhalten eines Echtzeit-Systems ist
hier von höchster Bedeutung. Es muss garantiert werden, dass der Airbag weder zu
früh, noch zu spät auslöst. Dieses Kriterium kann ein Standard-Rechner nicht
garantiert einhalten.


**Verkehrsampel-System:** Es muss gewährleistet werden, dass die Ampel
garantiert auf rot umschaltet, bevor die Ampeln der anderen Verkehrsteilnehmer
auf grün wechselt. Auch zu frühes Umschalten würde Unfälle provozieren. Bei
einem Standard-Rechner könnte es im Falle von hohen Lasten zu Verzögerungen
kommen.

**Motorsteuerung im Auto:** Erfolgt die Ansteuerung des Motors statt in Echtzeit
verzögert, kann es zu einem Stottern des Motors, oder sogar Stillstand des Autos
kommen. Ein Standard-Rechner kann die notwendige Echtzeit nicht gewährleisten.

\newpage

# Themengebiet: Betriebssystem

**1.) Welche Eigenschaften sind für ein Echtzeit-Betriebssystem wünschenswert?**


* **Multitasking:** Gleichzeitiges Ausführen mehrerer Prozesse. Die
  Systemaufrufe sollen reentrant sein, d.h. sie können erneut aufgerufen werden.
* **Echtzeit Scheduler/Dispatcher:** Ein höher priorisierter Task sollte
  eingeschoben werden können. *Preemption* beendet dazu aktive Tasks und schiebt
  den höher priorisierten Task ein, es bevorrechtigt sozusagen Tasks mit höherer
  Priorität. Ebenfalls müssen Abhängigkeiten von anderen Tasks berücksichtigt
  werden.
* **Echtzeit-Speicherverwaltung:** Der Zugriff auf den Speicher erfolgt in
  definierter Zeit. Dazu ist es beispielsweise hilfreich, Daten nicht
  auszulagern, oder den Caching-Mechanismus abzuschalten.
* **Echtzeit-Dateisystem:** Das Schreiben und Lesen von Dateien erfolgt in
  definierter Zeit. Dazu wird beispielsweise auf das Caching von Datei-Metadaten
  verzichtet.
* **IPC, ISR-fähig:** Daten werden aus ISR (Interrupt Service Routine) beispielsweise mittels
  Message Queues zurückgegeben.


**2.) Welche Rolle spielt UNIX im Prozeßrechnerbereich?**

Grundsätzlich gibt es nicht das klassische Echtzeit-Betriebssystem, das für
jeden Einsatzbereich geeignet ist. Die Anforderungen an das Betriebssystem
hängen vom Einsatzbereich ab.

Basierend auf UNIX gibt es das proprietäre Echtzeit-System *QNX*. Mit POSIX
(Portable Operating System Interface) gibt es einen an UNIX angelehnten
Standard für Schnittstellen zwischen Anwendungssoftware
und Betriebssystemen.

RT-Linux und LynxOS sind weitere Beispiele für Echtzeit-Systeme aus dem unixoiden
Bereich.



**3.) Was fehlt bei Standard-Linux zur Echtzeitfähigkeit? Was ist überflüssig?**

Standard-Linux kann nicht garantieren, dass ein Befehl in einer definierten Zeit
abgearbeitet ist.

Linux bietet keinen Scheduler, der Prozesse zu einem definierten
Zeitpunkt erneut ausführt. Es ist von vielen internen Faktoren abhängig, zu
welchem Zeitpunkt ein Prozess weiter ausgeführt wird. Außerdem ist es nicht garantiert, dass das System
unter Last Befehle entgegennehmen kann.

Durch die Nutzung eines präemtiven Kernels kann weiche Echtzeit ermöglicht
werden. Der Scheduler kann dann einen Task-Wechsel durchführen, sollte ein Task
mit höherer Priorität vorliegen.

Beim Standard-Linux ist beispielsweise die Multiuser--Fähigkeit überflüssig.
Außerdem alle rechenintensiven Bordmittel, die bei einem Echtzeit-System
nicht notwendig sind.

# Themengebiet:  Softwareentwicklung

**1.) Warum ist die Entwicklung von Echtzeitsoftware aufwendiger als die anderer
Software? Kann man Echtzeitsoftware nur mit Assembler realisieren?**

Es ist genaues Wissen über die verwendeten Bibliotheken notwendig. Der
Programmierer muss exakt wissen, wie die eingesetzten Bibliotheken intern
funktionieren, um Schwachstellen erkennen zu können. Es muss in Erfahrung
gebracht werden, wie einzelne Zeiten zustande kommen.

Der Entwickler muss Hardwarenahes Programmieren beherrschen. Außerdem sind
eventuell spezielle Hardware-Kenntnisse erforderlich.

Es muss Software geschrieben werden, deren Zeitverhalten von außen bestimmt ist.
Das erhöht den Testaufwand für Echtzeit-Software gegenüber anderer Software
enorm.

Echtzeit-Software muss nicht zwingend in Assembler geschrieben werden.
Programmiersprachen wie *C* oder *PEARL* sind ebenso möglich.


**2.) Wieso ist Pearl eine gute Programmiersprache für Echtzeitsysteme?**

PEARL bietet eine komfortable und sichere Entwicklung
von Multitasking- und Echtzeitanwendungen. Sie wurde seit 1977 in verschiedenen
Entwicklungsstufen genormt (letzte Normung DIN 66253-2 1998).

Bei der Entwicklung von PEARL wurde auf eine möglichst leichte Abbildbarkeit der
prozessrechentechnischen Probleme geachtet. PEARL wurde explizit für die
Programmierung von Echtzeit-Systemen entwickelt.




**3.) Wieso ist Ward-Mellor eine gute Entwurfmethode für Echtzeitsysteme?**

Die Ward-Mellor Methode ist sowohl für kleine, als auch für große Projekte geeignet.
Zudem ist sie leicht verständlich. Es gibt geeignete CASE-Tools, wie z.B.
EasyCase oder Select-Yourdon.

