---
layout: post
title:  "Calliope Hardware und Pins"
date:   2020-07-19 17:08:49 +0200
tags: calliope pinout sensor
---

Infos zum Calliope gibt es unter [http://https://calliope.cc](http://https://calliope.cc)

Der Calliope hat die folgenden Aktoren integriert:
* LED-Matrix
* Farb-LED
* Tonausgabe

Zudem enthält er die folgenden sensoren
* Tasten A / B
* PIN 0 bis 3 an den "Ecken"
* Lage / Beschleunigung
* Kompass
* Mikrofon
* Temperatur
* Licht

## Pinout

Der Calliope hat folgende Pin-Belegung:

![Calliope-Pinout](https://calliope-mini.github.io/assets/v10/img/Calliope_mini_1.3_pinout_fin.jpg)

Direkt nutzbar sind folgende Pins:
C0, C1, C2, C3 - davon C1 und C2 mit PWM
C16 und C17

Nach Deaktivierung der LED  sind zusätzlich nutzbar:
C4 - C12

Von folgende Pins können statt digitalen Werten auch analoge Werte gelesen werden:
P1, P2, C4, C5, C6, C16 und C17


Um die 6 Motorpins und die darunterliegenden 26 weiteren Pins mit Jumperkabeln zu nutzen ist es hilfreich,an die "Löcher" eine Pinleiste anzulöten.

![Calliope Mini](/images/foto_zwei_calliope.jpg) 

## Motortreiber des Callipe mini

Beim Calliope Mini ist ein einfacher Motortreiber onboard. Mit diesem kann man entweder
* zwei Motoren vorwärts oder
* einen Motor vorwärts und rückwärts betreiben

Auf dem Calliope gibt es dafür die mittlere Pinleiste mit 6 Pins (altes Modell 5 Pins). An diese schließt man einerseits die Spannungsquelle und andererseits den bzw. die Motoren an.

Im sogenannten Einmotorenbetrieb schließt man den Motor wie folgt an:
* der eine Anschluss des Motors wird mit dem Pin für Motor A verbunden (dritter Motorpin von links)
* der andere Anschluss des Motors wird mit dem Pin Motor B verbunden (zweiter Motorpin von links)
* die externe Spannungsquelle (bis 9V) wird mit dem fünften Motorpin von links
* Ground der externen Spannungsquelle wird mit dem sechsten Motorpin von links verbunden.

((TODO: Fritzing / Foto machen))

Im sogenannten Zweimotorenbetrieb sieht das ganze dann so aus:
* Motor A wird mit Pin für Motor A und Ground verbunden (Motorpins 3 und vier)
* Motor B wird mit Pin für Motor B und Ground verbunden (Motorpins)
* die externe Spannungsquelle (bis 9V) wird mit dem fünften Motorpin von links
* Ground der externen Spannungsquelle wird mit dem sechsten Motorpin von links verbunden.

((TODO:Fritzing / Foto machen))

Im Einmotorenbetrieb kann man den Motor so programmieren, dass er sich vorwärts oder rückwärts dreht. Im Zweimotorenbetrieb drehen sich die beiden Motoren immer nur vorwärts.
