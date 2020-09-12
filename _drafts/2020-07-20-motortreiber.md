---
layout: post
title:  "Ansteuerung von Gleichstrommotoren"
date:   2020-07-20 21:08:49 +0200
tags: Motor Motortreiber
---



# Gleichstrommotoren

Die Gleichstrommotoren benötigen viel mehr Strom als die Pins des Microcontrollers liefern können. Daher benötigt man einen Motortreiber um den von einer weiteren Spannungsquelle gelieferten Strom über die Pins zu schalten.





## Motormodul mit L298N-Chip

Mit dem Motormodul mit L298N-Chip kann man ein oder zwei Gleichstrommotoren vorwärts und rückwärts betreiben. 

Man kann ihn sowohl für den Raspberry Pi als auch für den Arduino verwenden. Er funktioniert auch am Calliope mini - so können auch dort zwei Motoren vorwärts und rückwärts betrieben werden.

((Bild / Schaltung einfügen))

Der Motortreiber L298 hat folgende Eingänge

* Versorgungsspannung
* Ground

und für Motor A:
* ENA
* IN1
* IN2

und für Motor B:
* ENB
* IN3
* IN4

Folgenden Tabellen kann man entnehmen, wie die Eingänge beschaltet werden müssen:

ENA | IN1 | IN2 | Beschreibung
--- |-----|-----|--------
0 | N/A | N/A | Motor A aus
1 | 0 | 0 | Motor A bremsen
1 | 0 | 1 | Motor A vorwärts
1 | 1 | 0 | Motor A rückwärts
1 | 1 | 1 | Motor A bremsen

ENB | IN3 | IN4 | Beschreibung
--- |-----|-----|--------
0 | N/A | N/A | Motor B aus
1 | 0 | 0 | Motor B bremsen
1 | 0 | 1 | Motor B vorwärts
1 | 1 | 0 | Motor B rückwärts
1 | 1 | 1 | Motor B bremsen

An ENA und ENB kann ein PWM-Signal angelegt und damit die Geschwindigkeit gesteuert werden



## Motorshields

Es gibt für den Arduino und den Raspberry Pi auch Motor-Shields. Diese habe ich bislang noch nicht getestet.