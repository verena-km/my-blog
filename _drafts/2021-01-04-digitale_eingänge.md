---
layout: post
title:  "Digitale Eingänge"
date:   2021-01-04
tags: GPIO Pullup Pulldown Callback
---

Verwendet man einen GPIO-Pin als Eingang, kann man z.B. Schalter anschließen. Um für einen definierten Grundzustand zu sorgen, nutzt man einen Pullup- bzw einen Pulldown-Widerstand.

Wenn man den GPIO-Pin mit dem Pluspol verbindet, braucht man einen Pulldown-Widerstand, wenn man ihn mit dem Ground verbindet braucht man einen Pullup-Widerstand.

## Widerstände in die Schaltung einbauen

Pullup:
* Beim Pullup wird der GPIO-Pin über einen Widerstand mit der Betriebsspannung verbunden. 
* Der Schalter wird auf der einen Seite mit dem GPIO-Pin und auf der anderen Seite mit GND verbunden.

Pulldown:
* Beim Pulldown wird der GPIO-Pin über einen Widerstand mit GND verbunden.
* Der Schalter wird auf der einen Seite mit dem GPIO-PIn und auf der anderen Seite mit der Betriebsspannung verbunden.

Empfohlen ist ein Widerstand mit 10 kOhm.

Detailliertere Erläuterungen zum Thema Pullup- und Pulldown-Widerstände findet man im [Elektronik-Kompendium](https://www.elektronik-kompendium.de/sites/raspberry-pi/2006051.htm).

## Widerstände softwareseitig aktivieren:

Beim Raspi, beim ESP32 und beim Calliope gibt es auch softwareseitig zuschaltbare Pullup- und Pulldown-Widerstände.
Auf der sicheren Seite ist man aber bei der Verwendung eines "echten Widerstandes".

### MicroPython / ESP32

In folgendem Beispiel für den ESP32 mit MicroPython wird Pin 23 mit einem Pullup-Widerstand konfiguriert.

```python
from machine import Pin
taster = Pin(23, Pin.IN, Pin.PULL_UP)

while True:
    print(taster.value())  
```

### Makecode / Calliope

Beim Calliope kann man die Widerstände softwareseitig mit 

```javascript
pins.setPull(DigitalPin.P0, PinPullMode.PullUp)
```

```javascript
pins.setPull(DigitalPin.P0, PinPullMode.PullDown)
```

aktivieren.

### Python / RasperryPi

Beim Raspi kommt es wieder darauf an, welche Bibliothek man verwendet.

RPi.GPIO:
```python
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BCM)
pin = 4
GPIO.setup(pin, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)

while True:
    print(GPIO.input(pin))
```

gpiozero:
```python
from gpiozero import Button
button = Button(4,pull_up=False)
while True:
    print(button.value)
```
(True ist default)


## Ausführen einer Callback-Funktion beim Wechsel von 0 auf 1

Anstatt in einer Schleife den am Eingang anliegeden Wert laufend abzufragen und dabei zu prüfen, ob dieser sich gändert hat, kann man in auch einen Interrupt registrieren, der auf den Wechsel mit der Ausführung einer sog. Callback-Funktion reagiert.

Ein Problem bei Tastern ist möglicherweise das [Prellen (Bouncing)](https://www.mikrocontroller.net/articles/Entprellung). Es kann dazu führen, dass der Interrupt und damit die Callback-Funktion zu häufig ausgeführt werden.

### Python / RasperryPi

Mit RPI.GPIO geht das wie folgt:

```python
import RPi.GPIO as GPIO
import time
from signal import pause

GPIO.setmode(GPIO.BCM)
pin = 4
GPIO.setup(pin, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)

def say_hello(pin):
    print("Hello!")
   
GPIO.add_event_detect(pin, GPIO.RISING, callback=say_hello)
pause()
```
Bei RPi.GPIO gibt es die folgenden Events:
* GPIO.RISING
* GPIO.FALLING
* GPIO.BOTH

Auf jedem Pin kann nur auf eines dieser Events ein Interrupt registriert werden.

Gegen das Prellen kann man einen Bouncetime-Wert in Millisekunden angeben:
```python
GPIO.add_event_detect(pin, GPIO.RISING, callback=say_hello, bouncetime=500)
```

Bei gpiozero sieht man die Nutzung an folgendem Beispiel:

```python
from gpiozero import Button
from signal import pause

def say_hello():
    print("Hello!")

def say_bye():
    print("ByeBye!")    

button = Button(4,pull_up=False)
button.when_pressed = say_hello
button.when_released = say_bye
pause()
```
Die Funktion say_hello wird bei jedem Wechsel von 0 auf 1 ausgeführt, die Funktion say_bye bei jedem Wechsel von 1 auf 0.

Auch hier kann man gegen das Prellen einen Wert (in Sekuncen) für die Bouncetime angeben:
```python
button = Button(4,pull_up=False, bounce_time=0.5)
```

### MicroPython / ESP 32

In MicroPython kann man einen solchen Interrupt wie folgt umsetzen:

```python
from machine import Pin

def callback(p):
    print('pin change', p)

p0 = Pin(0, Pin.IN)
p0.irq(trigger=Pin.IRQ_FALLING, handler=callback)
```

Mit der Methode `irq()` der Klasse Pin legt man einen solchen Interrupt fest und definiert mit `trigger`, ob er bei beim Wechsel von 0 nach 1 (Pin.IRQ_RISING) beim Wechsel von 1 nach 0 (Pin.IRQ_FALLING) oder bei beidem (Pin.IRQ_FALLING\|Pin.IRQ_RISING) ausgeführt werden soll.

Was nicht geht, ist für RISING und FALLING getrennte IRQs zu definiert. Es wird nur einer ausgeführt.

Um Debouncing in Software muss man sich in MicroPython selbst kümmern.

Hier eine Lösung, bei der eine bestimmte Anzahl gleicher aufeinander folgender Werte erreicht sein muss und erst dann die eigentliche Callback-Funktion ausgeführt wird:



Weitere Infos: [https://micronote.tech/2020/02/Timers-and-Interrupts-with-a-NodeMCU-and-MicroPython/](https://micronote.tech/2020/02/Timers-and-Interrupts-with-a-NodeMCU-and-MicroPython/])

Hier eine Lösung mit einem Timer:
[https://kaspars.net/blog/micropython-button-debounce](https://kaspars.net/blog/micropython-button-debounce)

### Calliope

siehe extra Post