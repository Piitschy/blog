--- 
title: 'Go-Funktionen in Python aufrufen'
description: 'Wie kann man Go-Funktionen in Python aufrufen?'
pubDate: 'Jul 08 2022'
heroImage: '/picography-laptop-notepad-desk-1.jpg'
---

# Ziel
Python ist einfach sehr sehr langsam.
Nun könnte man natürlich auch einfach die benötigte Funktion in C oder C++ schreiben und anschließend mit `ctypes` in Python importieren ... 
oder man macht das gleiche mit Go-Funktionen.

Go ist schnell geschrieben und einfach zu lesen - die steile Lernkurve erlaubt es außerdem die Grundkonzepte dieser Sprache an einem Nachmittag zu erfassen.
Gerade vor diesem Hintergrund ist Go als kompilierte Sprache verhältnismäßig schnell.
Die Nutzung von Goroutines ermöglicht sehr komfortables parallelisieren von Tasks - was man von Pythons Multithreading nicht behaupten kann.

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

Um die Go-Funktionen nutzen zu können, müssen sie mit dem Go-Compiler kompiliert werden. 
Mit der der Flag `-buildmode=c-shared` nutzt der Compiler die Dekoratoren in den Kommentaren über der Funktionsdefinition, 
um die 

```bash
go build -o go_func.so -buildmode=c-shared ./func.go
```

## Python Script

```python
from ctypes import *
from typing import Callable
from array import array
go = cdll.LoadLibrary("./go_func.so")

# call go function
r=go.add(10,20)

print(r)
```

# Performance

Nun stellt sich natürlich die Frage: Wie sieht es mit der Performance aus?
Um diese Frage zu beantworten machen wir einen kleinen Benchmark.

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

limit = 10 # 20, 30, 40
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

Wir führen das ganze nun unter Verwendung zweier Limits aus

### Ergebnis

```
limit 10
Python: 3.760399977181805e-05
Go: 0.000290665600005034
```

```
limit 20
Python: 0.0019388470000194502
Go: 0.00036239000019122614
```

```
limit 30
Python: 0.2242325689999234
Go: 0.00844102600012775
```

```
limit 40
Python: 29.484940967000057
Go: 1.5551268180001898
```

Bei einem Limit von 10 ist die Python-Funktion noch schneller, da der Funktionsaufruf einer Funktion aus einer C-Library etwas mehr Zeit benötigt. Bei längeren Berechnungen ist das Ergebnis jedoch Eindeutig: Go-Funktionen aufrufen lohnt sich!