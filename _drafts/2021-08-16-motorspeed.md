---
layout: post
title:  "Geschwindigkeitsmessung mit dem Raspi"
date:   2021-08-16
tags: Raspberry Speedsensor
---

Um die Drehzahl eines Motors zu messen, kann man z.B. den Speedsensor LM393 verwenden. Es handelt sich dabei um eine Infrarot-Gabellichtschranke die mit einem Encoder-Rad ausgeliefert wird. 

((TODO: Foto))

Eine Anleitung findet man hier: 
https://joy-it.net/files/files/Produkte/SEN-Speed/SEN-Speed-Anleitung-20201015.pdf

Der Speedsensor hat vier Anschlüsse, wovon aber nur drei genutzt werden:
* VCC - Spannungsversorgung 3,3 bis 5 V
* GND - Ground
* D0 - digitales Signal für den Zustand der Schranke

Da das mitgelieferte Encoderrad kein für Lego-Achsen passendes Achsloch hat, habe ich stattdessen eine Lego-Riemenscheibe (4185) verwendet. Während das mitgelieferte Encoder-Rad 20 Löcher hat, hat die Riemenscheibe nur 6 Löcher, was die Messung ungenauer macht. 

## Funktionstest

Um den Speedsensor zu testen, kann man ihn wie folgt an den Raspi anschließen:
* VCC am Sensor an 3,3 V am Raspi
* GND am Sensor an Ground am Raspi
* D0 am Sensor an GPIO 27 am Raspi

((TODO:Foto))

Mit folgendem Programm kann man die Lichtschranke testen, beispielsweise indem man einen Karton in die Schranke hält.

```python 
from gpiozero import DigitalInputDevice
from signal import pause

def activated():
    print("Offen")

def deactivated():
    print("Geschlossen")    

speedsensor = DigitalInputDevice(27)

speedsensor.when_activated = activated
speedsensor.when_deactivated = deactivated

pause()
```

Wir verwenden hier bei die Klassse DigitalInputDevice aus der gpiozero-Bibliothek und definieren zwei Funktionen, die ausgeführt werden, wenn die Schranke geschlossen bzw. die Schranke geöffnet wird.

## Geschwindigkeitsmessung

Um die Geschwindigkeit zu messen, platzieren wir die Riemenscheibe als Encoder-Rad in die Lichtschranke und treiben diese mit einem Lego-Motor an, der zunächst noch direkt mit der Lego-Batteriebox verbunden ist.

((TODO: Foto))

```python 
from gpiozero import DigitalInputDevice
from time import sleep

# Methode die zaehlt wie oft die Lichtschranke ausloest
def count(self):
    global counter
    counter = counter + 1

counter = 0
interval = 1 # sekunde
loecher = 6 

speedsensor = DigitalInputDevice(27)
speedsensor.when_activated = count
speedsensor.when_deactivated = count

while True:
    print("Umdrehungen pro Sekunde:" + str(round(counter/(loecher*2),1)))
    counter = 0
    sleep(interval)
```
Hier werden sowohl das Öffnen als auch das Schließen der Schranke gezählt. Jede Sekunde werden die gezählten Umdrehungen ausgeben und der Zähler anschließen zurückgesetzt.

Mit diesem Verfahren kann man die Drehzahl von Motoren ermitteln und mit anderen Motoren vergleichen

## Messung zur Steuerung verwenden

Man kann natürlich auch die gemessenen Werte verwenden, um den Motor zu steuern, also beispielsweise einen Motor nur eine zuvor festgelegte Anzahl von Umdrehungen durchführen lassen oder ein Fahrzeug damit eine bestimmte Strecke vorwärts fahren lassen. Hierzu folgt ein weiterer Beitrag.
