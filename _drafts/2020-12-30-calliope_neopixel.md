---
layout: post
title:  "NeoPixel am Calliope"
date:   2020-12-30
tags: Neopixel Calliope
---

Auch an den Calliope kann man Neopixel anschließen:

* NeoPixel-Ring-Power an Calliope-3V3-Pin
* NeoPixel-Ring-GND an Calliope-GND
* NeoPixel-Data-Input an Calliope-C1

In Makecode geht man dann unter Fortgeschritten - Erweiterungen und wählt dort "neopixel" aus. Man erhält dann zusätzliche Blöcke unter dem Punkt NeoPixel.

Um den Neopixel anzusteuern erzeugt man ein neues Neopixel-Objekt und macht es unter der Variable strip erreichbar. 
Dabei gibt man den Pin an, an dem der Neopixel angeschlossen ist und die Anzahl der Pixel des Neopixel-Objekts (im Beispiel 12)

```javascript
let strip = neopixel.create(DigitalPin.P1, 12, NeoPixelMode.RGB)
strip.showColor(neopixel.colors(NeoPixelColors.White))
strip.setBrightness(12)
basic.forever(function on_forever() {
    if (input.buttonIsPressed(Button.A)) {
        strip.showColor(neopixel.colors(NeoPixelColors.Blue))
    }
    
    if (input.buttonIsPressed(Button.B)) {
        strip.showColor(neopixel.colors(NeoPixelColors.Red))
    }
    
    if (input.buttonIsPressed(Button.AB)) {
        strip.showColor(neopixel.colors(NeoPixelColors.Green))
    }
    
})
```
Man kann z.B.:
* dem ganzen Strip eine bestimmte Farbe zuweisen
* dem Strip einen Regenbogen aus einem Farbspektrum zuweisen
* einzelnen LEDs bestimmte Farben zuweisen
* einen Balkendiagramm anzeigen (Anzahl der Pixel abhängig vom Wert)
* alle Neopixel ausschalten (danach muss noch Anzeigen aufgerufen werden.)
* Verschieben und Rotieren (danach muss noch Anzeigen aufgerufen werden.)
* Helligkeit verändern

