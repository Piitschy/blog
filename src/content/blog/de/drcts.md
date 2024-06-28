--- 
title: 'drcts - Directus Migration CLI'
description: 'drcts ist ein Go-CLI-Tool um directus Datenbanken zu migrieren. Die einfache Handhabung und die Möglichkeit, die Datenbanken in einem Schritt zu migrieren, machen drcts zu einem nützlichen Tool für Entwickler, die mit directus arbeiten.'
pubDate: 'Jun 28 2024'
heroImage: '/picography-laptop-notepad-desk-1.jpg'
tags:
    - directus
    - migration
    - cli 
    - go 
    - golang
    - tool 
---

<!--toc:start-->
- [Introduction](#introduction)
- [Installation](#installation)
- [Usage](#usage)
<!--toc:end-->

# Directus Migration CLI

## Introduction

[`drcts`](https://github.com/Piitschy/drcts) ist ein Go-CLI-Tool um [directus](https://directus.io/) Datenbanken zu migrieren. 
Die einfache Handhabung und die Möglichkeit, die Datenbanken in einem Schritt zu migrieren, machen `drcts` zu einem nützlichen Tool für Entwickler, die mit directus arbeiten.
Wenn beispielsweise eine Produktionsdatenbank in eine Testdatenbank migriert werden soll, ohne, dass Nutzerdaten mit übertragen werden sollen, ist `drcts` genau das, was du suchst!

## Installation

```bash
go install github.com/Piitschy/drcts@latest
```

## Usage

Für die Nutzung von `drcts` benötigst du die Zugangsdaten mit den notwendigen Rechten für die Datenbanken, die migriert werden sollen.
Dafür bestehen zwei Möglichkeiten die Zugangsdaten zu übergeben:
- Umgebungsvariablen
- Flags

[Hier](https://github.com/Piitschy/drcts?tab=readme-ov-file#environment-variables) findest du eine ausführliche Aufführung der notwendigen Umgebungsvariablen

Unter der Haube nutzt `drcts` die [directus API](https://docs.directus.io/api/reference.html) um die Datenbanken zu migrieren, vereinfacht dabei die Handhabung erheblich und führt den Nutzer durch den Prozess. 
So reicht (nach der Installation) ein einfacher Befehl, um die Datenbanken zu migrieren:

```bash
drcts migrate
```

Nun führt `drcts` dich durch den Prozess und fragt dich nach den notwendigen Informationen, um die Datenbanken zu migrieren.

Neben der Migration bietet `drcts` auch die Möglichkeit, Schemata zu speichern, miteinander zu vergleichen und Differenzen zwischen zwei Schemata direkt anzuwenden.
Achte darauf, dass du die notwendigen Rechte hast, um die Datenbanken zu migrieren, da `drcts` keine Rechteprüfung durchführt.
Bedenke auch, dass dieses Tool nur für die Migration von Schemata gedacht ist und keine Daten migriert!

### Schema speichern

```bash
drcts save -o schema.json
```
Mit diesem Befehl wird das Schema der Datenbank in die Datei `schema.json` gespeichert.
Diese Datei kann später beispielsweise zur Erstellung einer Differenzen-Datei genutzt werden.
So lassen sich Schemata der gleichen Datenbank zu verschiedenen Zeitpunkten miteinander vergleichen.

Wenn du `drcts` innerhalb eines Scripts nutzen möchtest, kannst du Credentials und Umgebungsvariablen auch direkt über Flags mitgeben:

```bash
drcts --bu <base-url> --bt <base-token> save -o <output-file>
```

### Schame anwenden 

Wenn du ein gespeichertes Schema auf eine Datenbank anwenden möchtest, nutze den Befehl `apply`:

```bash
drcts apply -i schema.json
```

Nun wird das Schema aus der Datei `schema.json` auf die Datenbank angewendet und das Schema entsprechend angepasst.

### Schema vergleichen

Um das gespeicherte Schema nun mit dem aktuellen Stand der Dirctus-Instanz zu vergleichen, nutze den Befehl `save-diff`:

```bash
drcts save-diff -i schema.json -o diff.json
```

Jetzt kannst du die Differenzen in der Datei `diff.json` einsehen, um zu entscheiden, ob und welche Differenzen du anwenden möchtest.

### Differenzen anwenden 

Um die Differenzen aus der Datei `diff.json` anzuwenden, nutze den Befehl `apply-diff`:

```bash
drcts apply-diff -i diff.json
```

Nun werden die Differenzen auf die Datenbank angewendet und das Schema entsprechend angepasst.

## Hilfe 

Für eine ausführliche Hilfe zu den Befehlen und Optionen, nutze den Befehl `help`:

```bash
drcts help
drcts <command> help
```
oder schaue in der `Readme` des [GitHub-Repositories](https://github.com/Piitschy/drcts) nach.
