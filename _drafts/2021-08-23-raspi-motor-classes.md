---
layout: post
title: "Motor und Roboter mit gpiozero"
date:   2021-08-16
tags: Raspberry Motor
---

Bereits beim Bluedot-Fahrzeug habe ich die Klasse Motor aus der gpiozero-Bibliothek verwendet.

Um Objekte der Klasse Motor zu erzeugen gibt man die Pins an, an die der Motortreiber am Raspi angeschlossen ist. Gibt man dabei auch einen enable-Pin an, kann man auch die Geschwindigkeit des Motors verändern.

Mit den Methoden forward() und backward() kann man den Motor in die jeweilige Richtung drehen lassen. Als Parameter kann man die Geschwindigkeit (zwischen 0 und 1) angeben.

Mit der Methode stop() wird der Motor gestoppt.

Folgendes Beispielprogramm zeigt die Nutzung:

((TODO Programmcode))

Für Fahrzeuge mit zwei angetriebenen Rädern und einem oder zwei Stützrädern kann man die gpiozero-Klasse Robot verwenden. Objekte der Klasse Robot haben jeweils einen rechten und einen linken Motor, die jeweils durch das Tuple der Pins repräsentiert sind, an die der Motor angeschlossen ist.

Mit den Methoden backward() und forward() kann man das Roboterfahrzeug forwärts und rückwärts fahren lassen. Als Parameter kann man die Geschwindigkeit (zwischen 0 und 1) angeben. Als weiteren Parameter kann man entweder curve_left oder curve_right angeben. Auch hier sind Werte zwischen 0 und 1 möglich. Bei Werten größer 0 fährt das Fahrzeug eine Kurve in die entsprechende Richtung

Weiter gibt es die Methoden left() und right(). Bei diesen dreht das Fahrzeug dadurch, dass ein Motor vorwärts und der andere rückwärts läuft. 

Mit der Methode stop() wird das Fahrzeug gestoppt.

Auch hier ein kleines Beispiel:

((TODO Programmcode))