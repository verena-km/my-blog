---
layout: post
title:  "Neopixel"
date:   2022-01-22 
tags: Neopixel Raspberry
---



https://learn.adafruit.com/neopixels-on-raspberry-pi


https://github.com/jgarff/rpi_ws281x


## Verkabelung

Ohne externe Stromversorgung ohne Level-Shifter

    Pi 5V to NeoPixel 5V
    Pi GND to NeoPixel GND
    Pi GPIO18 to NeoPixel Din



sudo pip3 install rpi_ws281x adafruit-circuitpython-neopixel

## Programmierung

Programm bleibt hängen

swig/python detected a memory leak of type 'ws2811_t *', no destructor found.

TODO: Testen auf frisch installiertem Raspi
