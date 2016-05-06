---
documentclass: scrartcl
title: Softwaresysteme - Parallele Softwareentwicklung (Prof. Dr. Märtin)
author: Susanne Kießling, \#948898
date: \today
toc: yes
---



# Allgmeines

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

OpenMP definiert sich durch folgende Haupteigenschaften:

* **Parallelisierung geschieht auf der Ebene von Schleifen**
* **Direktiven weisen Compiler an, Abarbeitung einer Schleife auf Threads bzw.
  Prozessoren zu verteilen**

Erste Tests wurden mit folgendem Code (basierend auf omp_hello.c von Blaise
Barney) ausgeführt:

```C
    #include <omp.h>
    #include <stdio.h>
    #include <stdlib.h>

    int main (int argc, char *argv[]) 
    {
    int nthreads, tid;


    /* Fork a team of threads giving them their own copies of variables */
    #pragma omp parallel private(nthreads, tid)
      {

        /* Obtain thread number */
        omp_set_num_threads(8);
        tid = omp_get_thread_num();
        printf("Hello World from thread = %d\n", tid);

        /* Only master thread does this */
        if (tid == 0) 
            {
            nthreads = omp_get_num_threads();
            printf("Number of threads = %d\n", nthreads);
            }

      }  /* All threads join master thread and disband */
    }

```



\newpage

# 2. pthread 

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
