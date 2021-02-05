---
layout: post
title:  "Stromversorgung des Roboterfahrzeugs"
date:   2020-09-12 9:00:00 +0200
tags: Stromversorgung Step-Down-Konverter Lego-Kabel
---

Der Roboter und damit sowohl das genutzte Board sowie die weiteren genutzten Komponenten benötigen eine passende Stromversorgung.

Dafür kann man Batterien oder Akkus verwenden. Man muss allerdings darauf achten, welche der Komponenten welche Spannung benötigen, bzw. anderen Komponenten bereitstellen. Hier eine kurze Auflistung:

* Raspberry Pi: 5 Volt
* Calliope: 3 Volt oder 5 Volt
* Arduino: 3,3 Volt oder 5 Volt
* DC-Motor: unterschiedlich
* Servomotor: 5 Volt
* Ultaschallsensor: 5 Volt

## Mobile Stromversorgungen

### Lego-Power-Functions Batteriebox

Zu den Lego-Power-Functions gehört die Batteriebox #8881, für 6-AA-Batterien (6 x 1,5 Volt = 9 Volt, bzw. 6 x 1,2 Volt = 7,2 Volt bei der Nutzung von Akkus).

![Lego Power-Functions-Batteriebox](/images/foto_lego_batteriebox1.jpg) 

Die Lego-Batteriebox lässt sich gut mit Lego-Technic-Bausteinen verbinden und somit auch gut in ein Lego-basierendes Robotermodell einbauen. Man sollte aber bei der Konzeption auch an den Batteriewechsel denken. 
Also:
* entweder so einbauen, dass man sie zum Wechsel der Batterien gut ausbauen kann
* oder so einbauen, dass  man an beide Seiten dran kommt, denn es sind jeweils drei Batterien pro Seite

Um Pins mit Plus- und Minuspol aus der Batteriebox zu erhalten nimmt man am besten ein angepasstes Lego-Verlängerungskabel. Wie man das herstellt, ist in diesem Video beschrieben: 

[https://youtu.be/bXkCcqNjf98](https://youtu.be/bXkCcqNjf98)

Das Ergebnis sieht dann so aus:

![Modifiziertes Lego-Verlängerungskabel](/images/foto_legokabel.jpg) 

### Callipe-Mini-Batteriebox

Zum Calliope Mini gehört eine Batteriebox für 2-AAA-Batterien (3 Volt), was für den Betrieb des Calliope genügt, aber nicht für die Motoren und weitere Komponenten mit einem Spannungsbedarf von > 3 Volt. Die Batteriebox wird mit dem mitgelieferten  JST-PH2.0-Stecker an den Calliope gesteckt. Damit sie nicht rumbaumelt kann man mit einem Gummi am Calliope befestigen, oder ein extra Case mit integriertem Batteriefach besorgen.

![Calliope Batteriebox)](/images/foto_calliope_mit_batteriebox.jpg) 


### Weitere Batterieboxen

Für meine Tests habe ich noch
* eine 4 x AA-Batteriebox
* eine 8 x AA-Batteriebox

verwendet.

### Powerbank

Eine weitere Möglichkeit zur Stromversorgung ist eine Powerbank (5 Volt). Die hat aber (zumeist) den Nachteil, dass sie bei zu kleinen Strömen automatisch nach ein paar Sekunden abschaltet.

## Eine Stromversorgung für alles

Batterien, Akkus und Powerbanks haben ein relativ hohes Gewicht und man will nicht unötiges Gewicht bewegen. Deshalb ist die Frage interessant, ob und wenn ja wie man mit einer Spannungquelle auskommt.

Eine Möglichkeit dabei ist die Verwendung eines sog. Step-Down-Konverters.

An den "LM2596S DC-DC Step-down Spannungsregler Dual USB 3A 9V-36V to 5V Power Module" kann man eine Eingangsspannung von 6 bis 9 Volt anschließen - also beispielsweise die Lego-Batteriebox - und erhält dann an den zwei integrierten USB-Ports jeweils 5 Volt zur Versorgung von Calliope, Raspi oder Arduino.

![Step-Down-Konverter](/images/foto_stepdownconverter.jpg) 

So kann man also die Spannung für die Motoren direkt aus einer Batteriebox beziehen und eine 5-Volt-Spannung für das Board über einen der USB-Ausgänge der Batteriebox.

Alternativ - noch nicht getestet - wäre die Nutzung eines Step-up-Konverters um aus den 5 Volt der Powerbank eine höhere Spannung für die Motoren zu erzeugen.

## Versorgung weiterer Komponenten

### Raspberry Pi und Arduino
Sowohl vom Raspberry Pi als auch vom Arduino kann man externe Komponenten mit 3 V oder 5 V Spannung versorgen.

### Calliope
Beim Calliope gibt es kein 5-V-Ausgang, man kann aber die 5 Volt für die Versorgung weitere Komponenten (z.B. Servo oder Ultraschallsensor) vom o.g. Step-Down-Konverter beziehen. Das genutzte Modell hat aber nur 5-Volt-USB-Ausgänge. Was also tun? Glücklicherweise fand sich in der Steckerkiste ein USB-Adapter (USB-A Stecker auf PS2 Buchse). Wenn man diesen an eine USB-Buchse des Stepdown-Konverters steckt kann man auf der anderen Seite in der PS2-Buchse auf Pin 4 die 5 Volt und auf Pin 3 Ground abgreifen.

Folgender Plan zeigt die Belegung der Pins:

![PS2 Pinout](https://upload.wikimedia.org/wikipedia/commons/thumb/1/17/MiniDIN-6_Connector_Pinout.svg/150px-MiniDIN-6_Connector_Pinout.svg.png)

Hier sieht man den USB-Adapter mit den herausgeführten Pins für 5 Volt und Ground:
![USB-PS2-Adapter mit Pins für 5 Volt und Ground](/images/foto_usb_ps2_adapter.jpg) 



 





