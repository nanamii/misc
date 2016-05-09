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
    Bsp: `omp_set_num_threads(n)`, legt die Anzahl der Threads fest.
* **Umgebungsvariablen (Environment variables): **Setzen der Parameter der
  Laufzeitumgebung zu Beginn der Programmausführung.
    Bsp: `OMP_NUM_THREADS=2`

\newpage

Erste Tests wurden mit folgendem Code (basierend auf omp_hello.c von Blaise
Barney [^hello]) ausgeführt:

```C
    #include <omp.h>
    #include <stdio.h>
    #include <stdlib.h>

    int main (int argc, char *argv[]) 
    {
    int nthreads, tid;
    /* Anzahl der Threads explizit auf 3 gesetzt.*/
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


Beispiel:

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

 - **LinuxThreads:** Die originale pthreads-Implementierung. Wird seit glibc
   Version 2.4 nicht mehr unterstützt.
 - **NPTL (Native POSIX Threads Library):** Im Vergleich zu LinuxThreads erfüllt
   es eher die Anforderungen von POSIX.1. Außerdem bietet es eine verbesserte
   Performance bei einer großen Anzahl an Threads.

Auf dem verfügbaren Testsystem läuft NPTL in der Version 2.22.


[^posix]: http://www.opengroup.org/austin/papers/posix_faq.html

# 2.1 Haupteigenschaften

* **Erzeugen eines neuen Threads:**
```C
	include <pthread.h>
	int pthread_create( pthread_t *thread,
                        const pthread_attr_t *attribute,
                        void *(*funktion)(void *), void *argument );
```    
    Jeder Thread kann eindeutig identifiziert werden. Diese
    Identifikationsnummer vom Datentyp `pthread_t` wird in der Adresse unter
    `*thread` abgelegt. Wird der Parameter `const pthread_attr_t *attribute` mit
    `NULL` belegt, werden die Standard-Attribute gesetzt. Mit `void
    *(*funktion)(void*)` wird eine *Thread-Funktion* bereitgestellt. Mit dem
    letzten Parameter `void *argument` wird der *Thread-Funktion* der Wert
    übergeben, mit dem sie aufgerufen werden soll.

* **Beenden eines Threads:**
    Ein Thread kann, wie bei Funktionen unter *C* üblich, mit `return` oder mit
    der Funktion `pthread_exit` beendet werden. Der Rückgabewert muss in beiden
    Fällen vom Typ `void*` sein.

    Mit `pthread_cleanup_push()` und `pthread_cleanup_pop()` werden nach Beenden
    eines Threads Ressourcen freigegeben.

* **Warten auf Threads:**
   Die Funktion `pthread_join()` ermöglicht dem Haupt-Thread auf das Ende und
   den Rückgabewert einzelner Threads zu warten.

* **Mutex (Wechselseitiger Ausschluss):**   
    Um zu vermeiden, dass Threads gleichzeitig auf gemeinsam genutzte Daten
    zugreifen, verwendet pthread das *Mutex-Verfahren* (*mutual exclusion*) in einer
    speziellen Form des *Edsger Dijkstra Semaphor*[^edsger].

[^edsger]: Butenhof, David R. Programming with POSIX threads. Addison-Wesley
Professional, S. 47, 1997.
    

# 2.2 Serielle vs. parallele/synchronisierte Version

Da bereits in Kapitel 1 (OpenMP) die numerische Lösung des Laplace'schen
Differentialgleichungssystems betrachtet wurde, soll für pthread ein anderes
Beispiel herangezogen werden. Basierend auf dem Code von Blaise Barney zur Berechnung eines Skalarprodukts soll das Laufzeitverhalten der seriellen 
Version[^dot2] mit dem der parallelen/synchronisierten[^dot]
 Version verglichen werden.

Im Code-Beispiel liegt die folgende Datenstruktur vor, auf die alle Threads
gemeinsam zugreifen:

```C
    typedef struct 
    {
        double      *a;
        double      *b;
        double     sum; 
        int     veclen; 
    } DOTDATA;
```


Die Funktion `void *dotprod(void *arg)` berechnet das Skalarprodukt. Darin enthalten ist die folgende for-Schleife:

```C
	mysum = 0;
	for (i=start; i<end ; i++) 
    {
      mysum += (x[i] * y[i]);
    }
```

Die parallele Version dieser Funktion hat zusätzlich folgenden Code, um zu vermeiden, dass Threads gleichzeitig auf die `struct DOTDATA` zugreifen:

```C
   pthread_mutex_lock (&mutexsum);
   printf("Thread %ld adding partial sum of %f to global sum of %f\n",
          arg, mysum, dotstr.sum);
   dotstr.sum += mysum;
   pthread_mutex_unlock (&mutexsum);

   pthread_exit((void*) 0);
```

In der parallelen Version *joinen* die Threads nach Berechnung des Skalarprodukts in den Hauptthread.

Die Laufzeitunterschiede zwischen paralleler und synchroner Version
sind in diesem Beispiel eher gering bzw. weniger von Bedeutung. Es ist vielmehr
ein gutes Beispiel, um die Eigenschaften von pthread (Erstellen eines Threads,
Mutex, joinen, etc.) testen zu können.




[^dot]: https://computing.llnl.gov/tutorials/pthreads/samples/mpithreads_threads.c
[^dot2]: https://computing.llnl.gov/tutorials/pthreads/samples/mpithreads_serial.c



# 3. Intel's VTune Amplifier

Intel's VTune Amplifier ist eine kommerzielle Anwendung für die Performanceanalyse
von Software. Zwei der zahlreichen Funktionalitäten sind beispielsweise das
Erkennen von problematischen Funktionen im Source-Code und die Betrachtung, wie
effizent die Hardware von der Software ausgenutzt wird.

Zu Testzwecken wurde die 30-Tage-Testversion eingesetzt: Intel VTune Amplifier
XE 2016 Update3[^download].

**Installation unter Linux:**

 - Entpacken: `tar -xzf vtune_amplifier_xe_2016_update3.tar.gz`
 - Installation über GUI: `./install_GUI.sh`
 - VTune-Amplifier-Umgebung erstellen: `source <install-dir>/amplxe-vars.sh`

**Starten:**

 - VTune-Amplifier-GUI: `amplxe-gui`
 - VTune-Amplifier-CL: `amplxe-cl`


Getestet wurden anschließend verschiedene Funktionalitäten, die Intel's VTune
Amplifier anbietet. An dieser Stelle soll beispielhaft der Ablauf einer
*Hotspot-Analyse* gezeigt werden. VTune liefert mit der Installation Test-Code,
der zur Analyse verwendet werden kann. Die *Hotspot-Analyse* wird für die
Software `tachyon` ausgeführt. Diese Software rendert ein Bild und zeigt die
dabei verbrauchte Zeit an.

**Der grobe Ablauf einer `Hotspot-Analyse`:**

 - Projekt Anlegen, Projektname vergeben
 - Zu testende Software festlegen
 - Zu verwendente Parameter angeben (im Beispiel `balls.dat`, Bild das
   gerendert werden soll)
 - Analyse-Typ auswählen (im Beispiel *Basic Hotspots*)
 - Analyse starten

\newpage

**VTune liefert folgende Ergebnisse (siehe Abbildung 1):**

 - **Elapsed Time**: Insgesamt hat die Ausführung des Programms 27,826 Sekunden
   gedauert, davon reine CPU-Rechenzeit 17,780 Sekunden.
 - **Top Hotspots:** Hier sind die Funktionen aufgelistet, die am meisten Rechenzeit
   in Anspruch genommen haben.
 - **CPU Usage Histogram:** Zeigt an, wie sich der tatsächlichen CPU-Verbrauch zur
   *wall time* verhält.

Zusätzlich liefert VTune noch Informationen zur Testplattform. In weiteren
Reitern können die als *Hotspot* erkannten Funktionen näher analysiert werden.

**Weiteres Vorgehen:** Der Source-Code muss nun analysiert werden. Dazu bietet
VTune eine visuelle Hilfe an und markiert kritische Code-Stellen. Mit
Doppelklick auf den jeweiligen Source-Code öffnet sich der voreingestellte
Texteditor und der Code kann bearbeitet werden. Sind die gewünschten Änderungen
eingepflegt, führt man erneut die *Hotspot-Analyse* durch. Dieses Vorgehen wird
solange durchgeführt, bis die gewünschte Performance erreicht ist.


![Ergebnis der Hotspot-Analyse](/home/sue/STUDIUM/Softwaresysteme/vtune.png)


[^download]: https//software.intel.com/en-us/intel-vtune-amplifier-xe/try-buy
