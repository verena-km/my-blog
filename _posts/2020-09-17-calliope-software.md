---
layout: post
title:  "Calliope - Programmierung"
date:   2020-09-17
tags: Calliope Makecode Debugging
---

## Editoren

Folgende Editoren stehen für die Programmierung des Calliope mini zur Verfügung

Webbasiert:
* [Makecode](https://makecode.calliope.cc/)
* [Makecode aktuelle Betaversion](https://makecode.calliope.cc/beta#editor)
* [Open Roberta Lab](https://lab.open-roberta.org/)

Android / iOS:
* Calliope mini App

### Makecode

[Makecode](https://makecode.microbit.org/) ist eine webbasierte Programmierumgebung von Microsoft. Makecode gibt es auch für den micro:bit, den Circuit Playground Express, für Minecraft, Lego Mindstorms, Cue, Arcade und Chibi Chip.

Das Git-Repository für den Calliope-Makecode ist hier:
[https://github.com/microsoft/pxt-calliope] (https://github.com/microsoft/pxt-calliope)

In Makecode kann man von einer Block-Ansicht in eine Javascript-Ansicht wechseln, in der neusten Beta-Version auch in eine Python-Ansicht.

Die neue Beta-Version von Makecode bietet auch die Möglichkeit für den Code ein Github-Repository anzulegen.

### Open Roberta Lab

[Open Roberta Lab](https://lab.open-roberta.org/) ist eine vom Fraunhofer IAIS entwickelte webbasierte Programmierumgebung. Mit Open Roberta Lab kann man neben dem Calliope auch [weitere "Roboter"](https://www.roberta-home.de/kids/die-roboter/) programmieren.


## Debugging über serielle Schnittstelle

Beim Erstellen und Testen von Programmen möchte man sich manchmal die aktuellen Werte von Variablen anschauen. Die 5x5-Matrix ist dafür nicht immer geeignet. Man kann aber die Daten über ein USB-Kabel mit an einen PC senden. Dafür gibt es das Modul `Seriell`, das in Makecode unter "Fortgeschritten" befindet. 

Beispielsweise kann man mit folgendem Code auf dem Calliope sich die Werte des Temperatursensors auf dem PC anzeigen lassen:

![Serielle Schnittstelle](/images/makecode_serial_demo.png) 

Auf dem PC benötigt man ein Terminal-Emulator-Programm, für Linux z.B. picocom.

Mit folgendem Befehl kann man sich im Linux-Terminal mit dem Calliope verbinden und die von dort über die Schnittstelle gesendeten Daten anschauen:
```
$ picocom -b 115200 /dev/ttyACM0
```
Zum Beenden muss man `Strg` drücken und dann ohne Loslassen nacheinander `a` und `q` drücken.
