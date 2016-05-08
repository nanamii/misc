---
documentclass: scrartcl
title: Softwaresysteme - Parallele Softwareentwicklung (Prof. Dr. Märtin)
author: Susanne Kießling, \#948898
date: \today
toc: yes
---


\newpage

# Allgemeines

Als Leistungsnachweis für das Fach Softwaresysteme -
Parallele Softwareentwicklung, werden aus fünf möglichen Aufgaben die folgenden drei Aufgaben bearbeitet:

* OpenMP
* pthread
* Intel's Vtune Amplifier

Tests wurden auf folgendem System ausgeführt:

 - CPU: Intel Core i7, 4x2,67GHz
 - OS: Fedora 23


# 1. OpenMP

OpenMP steht für *Open Specification for Multi Processing* und ist eine
Programmierschnittstelle für Shared-Memory-Programmierung auf
Multiprozessor-Rechnern. Wahlweise erfolgt die Programmierung in C, C++ oder
Fortran. Die erste Spezifikation von OpenMP ist 1997 entwickelt worden und
wurde vortan erweitert.


# 1.1 Haupteigenschaften und erste Tests

OpenMP definiert sich durch folgende Haupteigenschaften:

* **Parallelisierung** geschieht auf der Ebene von Schleifen (Haupteinsatzgebiet).
  Weiterhin wird Parallelität durch *Parallele Regionen*,
  *Worksharing-Konstrukte* und *Tasks* erreicht.
* **Direktiven** weisen Compiler an, Abarbeitung einer Schleife auf Threads bzw.
  Prozessoren zu verteilen.
* **Fork/Join Modell:** 
    OpenMP Programme starten mit einem Thread (Master-Thread),
    Team aus parallelen Threads (Worker-Threads) wird erstellt, Anweisungen im
    Parallel-Block werden von allen Threads bearbeitet, am Ende synchronisieren
    sich alle Threads und *joinen* in den Master-Thread. 
* **Laufzeit-Funktionen (Runtime subroutines/functions):** Werden hauptsächlich
  verwendet, um Parameter der Laufzeitumgebung von OpenMP abzufragen oder zu
  setzen.
    Bsp: `omp_set_num_threads(n)`, legt die Anzahl der OpenMP Threads fest
* **Umgebungsvariablen (Environment variables): **Setzen der Parameter der
  Laufzeitumgebung zu Beginn der Programmausführung.
    Bsp: `OMP_NUM_THREADS=2`
    

Erste Tests wurden mit folgendem Code (basierend auf omp_hello.c von Blaise
Barney [^hello]) ausgeführt:

```C
    #include <omp.h>
    #include <stdio.h>
    #include <stdlib.h>

    int main (int argc, char *argv[]) 
    {
    int nthreads, tid;
    /* Anzahl der Threads explizit auf 3 gesetzt.
    omp_set_dynamic(0)
    omp_set_num_threads(3);

    /* Ein Team von Threads *forken* und eine Kopie von Variablen übergeben */
    #pragma omp parallel private(nthreads, tid)
      {

        /* Thread-Nummer abfragen */
        tid = omp_get_thread_num();
        printf("Hello World from thread = %d\n", tid);

        /* Wird nur von Master-Thread ausgeführt, dieser ist standardmäßig 0 */
        if (tid == 0) 
            {
            nthreads = omp_get_num_threads();
            printf("Number of threads = %d\n", nthreads);
            }

      }  /* Alle Threads *joinen* den Master-Thread*/
    }

```

Ausgabe:

```bash
   >> Hello World from thread = 0
   >> Number of threads = 3
   >> Hello World from thread = 2
   >> Hello World from thread = 1
```

Das Code-Beispiel zeigt die parallele Ausführung der `print`-Anweisung. Innerhalb
des Codes wird der aktuell ausführende Thread mit `omp_get_thread_num()`
und die Anzahl der Threads mit `omp_get_num_threads()` abgefragt.
Die Reihenfolge in der die Threads die `print`-Anweisung abarbeiten, variiert je
Programmausführung.


[^hello]: https://computing.llnl.gov/tutorials/openMP/samples/C/omp_hello.c

# 1.2 Arten der Parallelverarbeitung

**Parallelisierung von Schleifen**

```C
    #include <stdio.h>
    #include <omp.h>

    int main(int argc, char* argv[])
    {

        int T[4];
        T[0] = 0;
        T[1] = 0;
        T[2] = 0;
        T[3] = 0;

        #pragma omp parallel
        {
            #pragma omp for
            for(int i=0; i<100; i++)
            {
                T[omp_get_thread_num()]++;
            }
        }
    printf("Thread 1: %d Durchläufe\n", T[0]);
    printf("Thread 2: %d Durchläufe\n", T[1]);
    printf("Thread 3: %d Durchläufe\n", T[2]);
    printf("Thread 4: %d Durchläufe\n", T[3]);
    return 0;
    }
```

Ausgabe:

```bash
    >> Thread 1: 25 Durchläufe
    >> Thread 2: 25 Durchläufe
    >> Thread 3: 25 Durchläufe
    >> Thread 4: 25 Durchläufe
```

Durch den Ausdruck `#pragma omp for` wird OpenMP angewiesen, die for-Schleife
parallel auszuführen. Wie in der Ausgabe zu sehen ist, hat bei vier verfügbaren Threads und insgesamt 100 Schleifen-Durchläufen jeder Thread 25 Durchläufe übernommen.

\newpage

**Parallele Regionen:** Parallele Ausführung eines beliebigen Code-Abschnitts.
  Die Direktive sieht in *C* wie folgt aus:

```C
    #pragma omp parallel [clause [clause ...]] 
    {
        ...
    }
```

Für *clause* kann eine der folgenden Optionen gewählt werden:
    `private(list)`,
    `shared(list)`,
    `default(private|shared|none)`,
    `firstprivate(list)`,
    `reduction(operator:list)`,
    `if(expr)`



```C
    #pragma omp parallel
    {
     printf(“Hello World!\n”);
    }
```

Ausgabe:

```bash
    >> Hello World!
    >> Hello World!
    >> Hello World!
    >> Hello World!
```

**Work-Sharing-Konstrukte:** Innerhalb einer parallelen Region kann die Arbeit
auf Threads verteilt werden.

+ Bei **manueller Aufteilung** der Arbeit muss der Zugriff auf die Aufgabenliste
  synchronisiert werden, um zu vermeiden, dass eine Aufgabe mehrfach ausgeführt
  wird.
+ Erfolgt die **Aufteilung der Arbeit nach Thread-Nummer**, kann die Anzahl der
  Threads im Team bestimmt werden. Die Arbeit lässt sich in Blöcke zerlegen und
  jedem Thread zuweisen.
+ **Work-Sharing-Konstrukte** ermöglichen eine automatische Verteilung der Arbeit.
    - Unterschiedliche Code-Abschnitte, die nicht voneinander abhängen und
      parallel zueinander abgearbeitet werden können: `sections`-Direktive
    - Code-Abschnitte, die nur von einem Thread bearbeitet werden dürfen.
      Hierdurch wird erreicht, dass eine bestimmte Anweisung
      sequenziell ausgeführt wird, ohne den parallelen Abschnitt verlassen zu
      müssen: `single`-Direktive
      

**Tasks:** Parallelisierung von Operationen auf komplexe Datenstrukturen, wie
Listen oder Bäume.


# 1.3 Laplace DGS

Bezüglich der numerischen Lösung des *Laplace'schen
Differentialgleichungssystems* wurde mit einer Implementierung von John Burkardt
experimentiert[^DGS].

[^DGS]: https://people.sc.fsu.edu/~jburkardt/c_src/heated_plate_openmp/heated_plate_openmp.c

Der Source-Code nähert sich der Lösung der iterativen Temperaturausbreitung in
einem zweidimensionalen Raum durch wiederholte Ausführung folgenden Codes:

```C
    W[Central] = (1/4) * ( W[North] + W[South] + W[East] + W[West] )
```

*W* ist ein Array, das die Temperaturen im Rechteck erfasst. *Central* ist der
aktuell betrachtete Punkt im Raum. *North*, *South*, *East* und *West* sind die
Punkte um den betrachteten Punkt herum. Durch die Multiplikation mit 1/4 wird
der Durchschnitt dieser benachbarten Punkte gebildet.

Durch ausreichend häufige Ausführung dieses Vorgehens geht die Differenz der
sukzessiven Schätzungen des Problems gegen Null. Hierzu kann eine Toleranzgrenze
gesetzt werden.

Der Source-Code erzeugt folgende Ausgabe:

```bash
    
C/OpenMP version
  A program to solve for the steady state temperature distribution
  over a rectangular plate.

  Spatial grid of 500 by 500 points.
  The iteration will be repeated until the change is <= 1.000000e-03
  Number of processors available = 4
  Number of threads =              4

  MEAN = 74.949900

 Iteration  Change

         1  18.737475
         2  9.368737
         4  4.098823
         8  2.289577
        16  1.136604
        32  0.568201
        64  0.282805
       128  0.141777
       256  0.070808
       512  0.035427
      1024  0.017707
      2048  0.008856
      4096  0.004428
      8192  0.002210
     16384  0.001043

     16955  0.001000

  Error tolerance achieved.
  Wallclock time = 59.955202

HEATED_PLATE_OPENMP:
  Normal end of execution
```

Die **parallele Implementierung** hat bei einem Rechteck mit 500x500 Punkten
**59.955202 Sekunden** gebraucht. Die Toleranzgrenze lag bei 0.001, dafür wurden
16955 Iterationen benötigt. Zur Zeitmessung wurde die Funktion `omp_get_wtime()`
der OpenMP-Laufzeitbibliothek verwendet.

Die **serielle Version** hat bei sonst gleichen Parametern **81.080792
Sekunden** benötigt. Zur Zeitmessung wurde die C-Bibliotheksfunktion `clock()`
der `<time.h>` C-Bibliothek verwendet. 

\newpage

# 2. POSIX Threads (pthread)

POSIX Threads, oder auch *pthreads* genannt, spezifizieren Schnittstellen
(Funktionen, Header-Dateien) für die parallele Programmierung. POSIX steht für
*Portable Operating System Interface* und ist eine von IEEE und der Open Group
entwickelte API [^posix].

Unter Linux gibt es zwei Implementierungen von pthreads, die von der GNU C
Bibliothek unterstützt werden:

    - LinuxThreads: Die originale pthreads-Implementierung. Wird seit glibc
      Version 2.4 nicht mehr unterstützt.
    - NPTL (Native POSIX Threads Library): Im Vergleich zu LinuxThreads erfüllt
      es eher die Anforderungen von POSIX.1. Außerdem bietet es eine verbesserte
      Performance bei einer großen Anzahl an Threads.

Auf dem verfügbaren Testsystem läuft NPTL in der Version 2.22.


[^posix]: http://www.opengroup.org/austin/papers/posix_faq.html

# 2.1 Haupteigenschaften

* Erzeugen eines pthreads


# 2.2 Arten der Parallelverarbeitung

# 2.3 Laplace DGS

**1.) Welche Eigenschaften sind für ein Echtzeit-Betriebssystem wünschenswert?**


* **Multitasking:** Gleichzeitiges Ausführen mehrerer Prozesse. Die
  Systemaufrufe sollen reentrant sein, d.h. sie können erneut aufgerufen werden.
* **Echtzeit Scheduler/Dispatcher:** Ein höher priorisierter Task sollte
  eingeschoben werden können. *Preemption* beendet dazu aktive Tasks und schiebt
  den höher priorisierten Task ein, es bevorrechtigt sozusagen Tasks mit höherer
  Priorität. Ebenfalls müssen Abhängigkeiten von anderen Tasks berücksichtigt
  werden.


**3.) Was fehlt bei Standard-Linux zur Echtzeitfähigkeit? Was ist überflüssig?**

# 3. Intel's Vtune Amplifier
