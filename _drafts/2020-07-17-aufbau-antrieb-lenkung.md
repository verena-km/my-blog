---
layout: post
title:  'Aufbau, Antrieb und Lenkung'
date:   2020-07-17 10:50:49 +0200
tags: Lego, Motor
---

Hier sind folgende Überlegungen wichtig:
* Anzahl, Art und Größe der Räder (Antriebsräder / Angetriebene Räder)
* Verbindung von Motoren und Antriebsrädern 
* Art der Lenkung
* Befestigung der Fremdkomponenten

Lego Technic bietet hier unendlich viele Möglichkeiten, hier die von mir geplanten Modelle:

## Basismodell A: Fahrzeug mit zwei angetriebenen Rädern und einem oder zwei Stützrad

Das Fahrzeug hat zwei Räder und ein oder zwei Stützräder. Jedes der Räder wird mit einem DC-Motor verbunden. Da diese separat angesteuert werden können, kann sich jedes Rad unabhängig vom anderen drehen. So läßt sich das Fahrzeug lenken. Die Stützräder helfen, dass das Fahrzeug nicht umkippt.

## Basismodell A: Variante mit Ketten

Auch hier das Antriebsrad jeder der beiden Ketten mit einem DC-Motor verbunden, so läßt sich jede Kette einzeln drehen. Stützräder brauchen wir hier keine. Die Motoransteuerung ist die gleiche wie beim Fahrzeug mit zwei angetriebenen Rädern

## Basismodell B: Fahrzeug mit Lenkung

Das Fahrzeug hat vier Räder. Die beiden hinteren Räder werden gemeinsam von einem DC-Motor angetrieben. Für die Lenkung wird ein Servomotor genutzt.


## Einsatz von Zahnrädern

Die Geschwindigkeit kann einerseits durch Pulsweitenmodulation bei der Ansteuerung der Motoren reguliert werden. Oftmals ist es aber auch sinnvoll, zwischen Motorachse und Radachse Zahnräder einzufügen um das Fahrzeug schneller oder langsamber zu machen.

Beispiel: 
 
1. Anschluss Motor an 12-Zähne-Zahnrad
2. 12-Zähne-Zahnrad treibt 20-Zähne-Zahnrad an
3. 20-Zähne-Zahnrad auf gleicher Achse wie 8-Zähne-Zahnrad
4. 8-Zähne-Zahnrad treibt 24-Zähne-Zahnrad an
5. Rad auf gleicher Achse wie 24-Zähne-Zahnrad

Wir haben also drei Achsen. Die Motorachse, eine Zwischenachse und die Achse des Rads.

1. Bei einer Umdrehung der Motorachse, dreht sich die Zwischenachse 12/20-mal
2. Bei einer Umdrehung der Zwischenachse, dreht sich die Radachse 8/24-mal

Insgesamt dreht sich bei einer Umdrehung der Motorachse die Radachse 12/20 * 8/24 mal, also 1/5 mal. Also anders gesagt für eine Umdrehung des Rads sind fünf Umdrehungen des Motors erforderlich.

Beim Einsatz von Zahnräder gilt:
* möglichst wenige Zahnräder verwenden
* klein auf groß gibt eine kleinere Drehzahl, aber ein höheres Drehmoment
* groß auf klein gibt eine höhere Drehzahl, aber ein kleineres Drehmoment

## Befestigung von Fremdkomponenten

Wie man die Einplatinencomputer und sonstigen Bauteilen mit Lego verbindet, muss man ausprobieren, beispielsweise:

* Anschrauben
* Ankleben (Heißkleber, doppelseitiges Klebeband)
* Gummi
* Kabelbinder
* Bau eines Lego-Gehäuses für die Komponente

Zu berücksichtigen ist, wie dauerhaft man die Verbindung braucht, bzw. wie fest diese sein muss.

Hier ein paar Beispiele:
* Aufkleben eines kleinen Breadboards auf eine 4x4-Platte
* Verbindung des Servomotors mit Heißkleber an 2 2x3-Platten bzw. Verschraubung mit einer 2x2-Platte
* Gummi-Befestigung des Calliopes
* Doppelseitiges Klebeband und Gummi für den Step-Dowm-Konverter

