---
layout: post
title:  'Aufbau, Antrieb und Lenkung des Roboterfahrzeugs'
date:   2020-09-14 9:00:00 +0200
tags: Lego Motor Zahnrad
---

Hier sind folgende Überlegungen wichtig:
* Anzahl, Art und Größe der Räder (Antriebsräder / Angetriebene Räder)
* Verbindung von Motoren und Antriebsrädern 
* Art der Lenkung
* Befestigung der Fremdkomponenten

Lego Technic bietet hier unendlich viele Möglichkeiten, hier die von mir geplanten Modelle:

## Fahrzeug mit zwei angetriebenen Rädern und einem oder zwei Stützrädern

Das Fahrzeug hat zwei Räder und ein oder zwei Stützräder. Jedes der Räder wird mit einem DC-Motor verbunden. Da diese separat angesteuert werden können, kann sich jedes Rad unabhängig vom anderen drehen. So lässt sich das Fahrzeug lenken. Die Stützräder helfen, dass das Fahrzeug nicht umkippt.

Im folgenden Beispielfahrzeug sind zwei M-Motoren verbaut:

![Fahrzeug mit zwei angetriebenen Rädern und einem Stützrad](/images/foto_modell_a.jpg)

## Fahrzeug mit zwei Ketten

Auch hier ist das Antriebsrad jeder der beiden Ketten mit einem DC-Motor verbunden, so lässt sich jede Kette einzeln drehen. Stützräder brauchen wir hier keine. Die Motoransteuerung ist die gleiche wie beim Fahrzeug mit zwei angetriebenen Rädern.

Für die Kettenvariante nutzen wir den Stunt-Racer (B-Modell) als Ausgangsmodell. Statt Spoiler und hochklappbarer Verkleidung habe ich eine hochklappbare Halterung für den Microcontroller gebaut.

![Fahrzeug mit zwei Ketten](/images/foto_stunt_racer_mod.jpg)

## Fahrzeug mit Lenkung

Das - noch zu bauende - Fahrzeug hat vier Räder. Die beiden hinteren Räder werden gemeinsam von einem DC-Motor angetrieben. Für die Lenkung wird ein Servomotor genutzt.


## Einsatz von Zahnrädern

Die Geschwindigkeit kann einerseits durch Pulsweitenmodulation bei der Ansteuerung der Motoren reguliert werden. Oftmals ist es aber auch sinnvoll, zwischen Motorachse und Radachse Zahnräder einzufügen, um das Fahrzeug schneller oder langsamer zu machen.

Beispiel: 
1. Anschluss Motorachse an 8-Zähne-Zahnrad
2. 8-Zähne-Zahnrad treibt 24-Zähne-Zahnrad an
3. Rad auf gleicher Achse wie 24-Zähne-Zahnrad

Im folgenden Bild sieht man dieses Beispiel:
![Foto Zahnräder](/images/foto_zahnraeder_untersetzung.jpg)

Damit dreht sich bei einer Umdrehung der Motorachse die Radachse 8/24-mal, als 1/3 mal. Also anders gesagt: Für eine Umdrehung des Rads sind drei Umdrehungen des Motors erforderlich.

Beim Einsatz von Zahnräder gilt:
* möglichst wenige Zahnräder verwenden
* klein auf groß gibt eine kleinere Drehzahl, aber ein höheres Drehmoment (langsamer - Untersetzung)
* groß auf klein gibt eine höhere Drehzahl, aber ein kleineres Drehmoment (schneller - Übersetzung)

## Befestigung von Fremdkomponenten

Wie man die Einplatinencomputer und sonstigen Bauteilen mit Lego verbindet, muss man ausprobieren, beispielsweise:

* Anschrauben
* Ankleben (Heißkleber, doppelseitiges Klebeband)
* Gummi
* Kabelbinder
* Bau eines Lego-Gehäuses für die Komponente
* Einsatz von Verbindungselementen aus dem 3-D-Drucker

Zu berücksichtigen ist, wie dauerhaft man die Verbindung braucht, bzw. wie fest diese sein muss.

Hier ein paar Beispiele:
* Aufkleben eines kleinen Breadboards auf eine 4x4-Platte
* Verbindung des Servomotors mit Heißkleber an 2 2x3-Platten bzw. Verschraubung mit einer 2x2-Platte
* Gummi-Befestigung des Calliopes
* Doppelseitiges Klebeband und Gummi für den Step-Down-Konverte
