--- 
title: 'Go-Funktionen in Python aufrufen'
description: 'Wie kann man Go-Funktionen in Python aufrufen?'
pubDate: 'Jul 08 2022'
heroImage: '/picography-laptop-notepad-desk-1.jpg'
---

# Ziel
Wozu der ganze Kram eigentlich?

Python ist einfach sehr sehr langsam.
Nun könnte man natürlich auch einfach benötigte Funktion mit möglichst kurzer Ausführungszeit in C oder C++ schreiben und anschließend mit `ctypes` in Python importieren ... 
oder man macht das gleiche mit Go-Funktionen.

Go ist schnell geschrieben und einfach zu lesen - die steile Lernkurve erlaubt es außerdem die Grundkonzepte dieser Sprache an einem Nachmittag zu erfassen.
Gerade vor diesem Hintergrund ist Go als kompilierte Sprache verhältnismäßig schnell.
Der mitgelieferte Compiler ist schnell und einfach zu bedienen, außerdem ermöglicht die Nutzung von Goroutines ein sehr komfortables parallelisieren von Tasks - was man von Pythons Multithreading nicht behaupten kann.

Aus diesen Gründen ist es durchaus interessant in Anwendungsfällen, in dem bestimmte Teile eines vorhandenen Python-Skripts, welches die Vorzüge beispielsweise einer Library oder eines Frameworks nutzt, etwas beschleunigt werden müssen.

# Vorgehen
## Was brauchst du?

- Python 3 (besser 3.10+ für den folgenden Code)
- Go 1.18+


Lege dir zunächst eine Python- und eine Go-File an.
```bash
touch func.go && touch main.py 
```


## Funktion mit primitiven Datentypen

Zunächst schauen wir uns Funktionen mit primitiven Datentypen an.
Die Anwendung dieser ist etwas einfacher.

### Go-Funktion

Schreibe nun eine simple Funktion in deiner Go-File:

```go
package main

import "C"

//export add
func add(x int, y int) int {
	return x + y
}

func main() {}
```

Wichtig ist dabei, dass du mit `import "C"` die C-Funktionalitäten von Go einbindest.
Dies ermöglicht dir mit dem Kommentar ```//export <name>```, als eine Art Dekorator über deiner Funktion, 
dem Compiler die Anweisung zu geben, eben diese Funktion zu exportieren.

### Kompilieren

Um die Go-Funktionen nutzen zu können, müssen sie mit dem Go-Compiler zu einer Shared Library kompiliert werden. 
Mit der der Flag `-buildmode=c-shared` nutzt der Compiler die Dekoratoren in den Kommentaren über der Funktionsdefinition, 
um die Funktion in kompilierter Form in der Library und einer Header-Datei zu speichern.

Diese wollen wir dann mit Hilfe des `c-types`-Moduls in Python nutzen.

```bash
go build -o go_func.so -buildmode=c-shared ./func.go
```

Nun sollten in dem Arbeitsordner zwei neue Dateien liegen: 
1. Die Header-Datei `go_func.h`
2. Die Shared-Library `go_func.so`

## Python Script
Nun schreiben wir ein Python-Skript zur Nutzung dieser Shared Library.

```python
from ctypes import *
go = cdll.LoadLibrary("./go_func.so")

# call go function
r=go.add(10,20)

print(r)
```

Wichtig ist zunächst, dass wir die `ctypes` importieren.
Mit der Methode `LoadLibrary` auf `cdll` können wir nun beliebige Shared Libraries - und somit auch unsere eben erstellte - laden.

In diesem Fall habe ich die Library in die Variable `go` geschrieben.
Auf dieser können wir nun alle Funktionen der Library aufrufen - auch unsere `add`-Funktion.

Primitive Datentypen, wie Integer, Floats und Booleans können dabei nativ übertragen werden. `ctypes` kümmert sich selbstständig um die Konvertierung in C-Datentypen.

# Performance

Nun stellt sich natürlich die Frage: Wie sieht es mit der Performance aus?
Um diese Frage zu beantworten, machen wir einen kleinen Benchmark.

_Das ganze ist natürlich nicht repräsentativ und soll lediglich zur Einordnung dienen. Dass die Verwendung von Integern und Rekursion vorteilig auf die Berechnungszeit in Go und nachteilig auf die von Python wirkt, ist klar - aber es kommt in der Realität und vorallem in der wissenschaftlichen Datenverarbeitung häufiger zu solchen oder ähnlichen Begebenheiten_

## Aufbau

Ich führe einfach eine rekursive Berechnung der Fibonacci-Folge aus.
Schwischenergebnisse werden nicht gespeichert und wir nutzen auch keine Caching-Methoden in Python.

Zunächst die Go-File, die wir mit dem oben genannten Compiler-Befehl kompilieren:
```go
package main

import "C"

//export fibonacci
func fibonacci(n int) int {
	if n <= 1 {
		return n
	}
	return fibonacci(n-1) + fibonacci(n-2)
}

func main() {}
```

... und die Funktion in Python:
```python
def fibonacci(n:int) -> int:
    if n<=1:
        return n
    else:
        return fibonacci(n-1)+fibonacci(n-2)
```

Nun schreiben wir ein einfaches Skript in Python, um die beiden Funktionen miteinander zu vergleichen:
```python
from ctypes import *
from timeit import default_timer as timer

go = cdll.LoadLibrary("./go_func.so")

for limit in range(5,50):
	print(f"limit: {limit}")

	start1 = timer()
	res1 = fibonacci(limit)
	end1 = timer()
	print(f"Python: {end1-start1}")

	start2 = timer()
	res2 = go.fibonacci(40)
	end2 = timer()
	print(f"Go: {end2-start2}")
```

Wir führen das ganze nun unter Verwendung verschiedener Limits aus.

### Ergebnis

![Alt-Text](/call-go-in-python/graph.png)

Auf der y-Achse ist logarithmisch die benötigte Zeit in Sekunden und auf der x-Achse das Limit für die Funktion aufgetragen.
Man kann gut erkennen, dass bei einem linearen Anstieg des Limits Ausführungszeit exponentiell steigt.
Der Funktionsaufruf in Go ist dabei meist eine ganze Größenordnung schneller als der native.

Allerdings macht die leicht verlängerte Aufrufszeit der Go-Funktion dieses Verfahren erst mit einer gewissen Workload richtig effektiv.