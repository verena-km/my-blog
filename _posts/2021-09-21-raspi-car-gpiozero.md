---
layout: post
title: "Raspi-Fahrzeug mit zwei Antriebsmotoren"
date:   2021-09-21
tags: Beispielfahrzeug Raspberry gpiozero L298 Motor
---

Dieses Raspi-Fahrzeug mit zwei Antriebsmotoren soll als Grundlage für verschiedene Erweiterungen mit Sensoren verwendet werden.

Wir nutzen dafür die Klasse `Robot` aus der [gpiozero-Bibliothek](https://gpiozero.readthedocs.io) . Die Klasse Robot ist für Fahrzeuge gedacht, bei denen jedes der beiden Antriebsräder von einem Motor angetrieben wird und bei Bedarf Stützräder zum Einsatz kommen.

Bereits beim [Fahrzeug mit Lenkung]({%post_url 2021-02-22-beispielfahrzeug-lenkung%}) habe ich die Klasse Motor aus der gpiozero-Bibliothekverwendet.


![Foto Fahrzeug mit zwei Antriebsmotoren](/images/foto_fahrzeug_2m.jpg)

## Komponenten und Aufbau

Man braucht dafür:
* 1 Lego-Batteriebox
* 2 Lego M-Motoren
* 1 Step-Down-Konverter
* 1 Motortreiber L298
* 1 Raspberry Pi Zero
* und diverse Lego-Technic-Teile

Die Motoren wurden nebeneinander angebracht, die Bewegung wird über Kegelzahnräder auf die Räder übertragen. Durch die dabei verwendete Untersetzung drehen sich die Räder langsamer als der Motor.

![Foto Fahrzeug mit zwei Antriebsmotoren](/images/foto_fahrzeug_2m_unten.jpg)

Die Batteriebox wurde oberhalb der Motoren angebracht und zwar so, dass sie sich gut auswechseln läßt.

Oben sind Raspi-Zero, Step-Down-Konverter und Motortreiber im Prototyp mit Gummis angebracht. Über den Step-Down-Konverter wird der Raspi über USB mit Strom aus der Batteriebox versorgt, der Motortreiber L298 wird über die Eingangspins des Step-Down-Konverters direkt an die Batteriebox angeschlossen.

Der Raspi steuert die über je zwei Kabel am Motortreiber L298 angeschlossenen Motoren. Geschwindigkeitsänderungen können beim L298 entweder über ein PWM-Signal an den beiden IN-Eingängen oder über ein PWM-Signal am EN-Eingang erreicht werden. Da bei der Nutzzung von `gpiozero` sowieso das PWM-Signal an den IN-Eingängen verwendet wird, benötigen wir keine Verbindung vom Raspi zu den EN-Eingängen, sondern können diese durch Setzen der Jumper auf 'enabled' setzen.

Details der Verkabelung sind im folgenden Schaltplan zu finden.

![Schaltplan Fahrzeug mit Raspi und zwei Antriebsmotoren](/images/fritzing_raspi_dual_motor.png)

## Programmierung

Für Fahrzeuge mit zwei angetriebenen Rädern und einem oder zwei Stützrädern kann man die gpiozero-Klasse Robot verwenden. Objekte der Klasse Robot haben einen rechten und einen linken Motor, die jeweils durch das Tuple der Pins repräsentiert sind, an die der Motor angeschlossen ist.

Mit den Methoden `backward` und `forward` kann man das Roboterfahrzeug vorwärts und rückwärts fahren lassen. Als Parameter kann man die Geschwindigkeit (zwischen 0 und 1) angeben. Als weiteren Parameter kann man entweder `curve_left` oder `curve_right` angeben. Auch hier sind Werte zwischen 0 und 1 möglich. Bei Werten größer 0 fährt das Fahrzeug eine Kurve in die entsprechende Richtung

Weiter gibt es die Methoden `left` und `right`. Bei diesen dreht das Fahrzeug dadurch, dass ein Motor vorwärts und der andere rückwärts läuft. 

Mit der Methode `stop` wird das Fahrzeug gestoppt.

Im folgenden Programm wird ein Robot-Objekt erzeugt, verschiedene Fahraktionen werden dreimal durchgeführt und das Fahrzeug dann gestoppt.

```python
from gpiozero import Robot
from time import sleep

robot = Robot((24,23),(12,25))

for i in range(3):
    robot.forward(curve_right=.5)
    sleep(1)
    robot.backward(speed=.7)
    sleep(1)
    robot.right()
    sleep(1)
    robot.left()
    sleep(1)
robot.stop()
```

Durch die Nutzung der gpiozero-Klasse Robot kann man über die Property `value` die aktuelle Fahrbewegung abfragen und diese auch auf einen anderen Wert setzen:

```python
from gpiozero import Robot
from time import sleep

robot = Robot((24,23),(12,25))

robot.forward(curve_right=.5)
print(robot.value)
sleep(1)
robot.value(1,1)
sleep(1)
robot.value(0,0)
```

Es werden dafür Tupel (ein Wert für links, ein Wert für rechts) verwendet. Die Werte liegen zwischen -1 (volle Geschwindigkeit rückwärts) und 1 (volle Geschwindigkeit vorwärts).

Diese Möglichkeit, den Status abzufragen oder zu setzen gibt es auch bei anderen Klassen aus der gpiozero-Bibliothek. Je nach Klasse sind das dann boolsche Werte, Wertebereiche oder Tupel.

Zudem ist möglich, dem Robot in der Property `source` ein iterierbares Objekt zuzuordnen, das die Werte enthält, die nacheinander als `value` genutzt werden sollen. Dies kann wie im folgenden Beispiel eine Liste mit Tupeln für die Fahrbewegungen sein. Mit der Property `source_delay` kann man festlegen, wie lange die einzelnen Werte gültig sein sollen.

```python
from gpiozero import Robot
from time import sleep

robot = Robot((24,23),(12,25))
instructions = [(1,1),(.5,.5),(-1,-1),(0,0)]
robot.source_delay = .5
robot.source = instructions
sleep(robot.source_delay * len(instructions))
```

Es kann aber auch ein Generator sein, der die Tupel, erzeugt. Im nachfolgenden Beispiel erzeugt dieser zufällige Tupel mit Werten zwischen -1 und 1.

```python
from gpiozero import Robot
from signal import pause
import random

def rand_robot_tuple():
    while True:
        yield (random.uniform(-1,1),random.uniform(-1,1))

robot.source =  rand_robot_tuple()
robot.source_delay = .5
pause()
```

Diese Art der Nutzung ist ebenfalls bei mehreren Klassen der gpiozero-Bibliothek möglich. Mit diesesm Mechanismus können z.B. auch die Werte von einem Eingabegerät direkt an ein Ausgabegerät weitergegeben werden. Beispiele dafür sind in der [gpiozero-Doku](https://gpiozero.readthedocs.io/en/stable/source_values.html) zu finden. 


