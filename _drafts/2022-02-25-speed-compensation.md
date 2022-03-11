---
layout: post
title:  "Unterschiedliche Geschwindigkeit von Motoren ausgleichen"
date:   2022-02-25 
tags:  Beispielfahrzeug Raspberry
---

Beim [Raspi-Fahrzeug mit zwei Antriebsmotoren]({% post_url 2021-09-21-raspi-car-gpiozero%}) ist es möglich, dass die Motoren nicht exakt dieselbe Geschwindigkeit haben.

Bei der Nutzung von der Robot-Klasse von `gpiozero` fährt das Fahrzeug dann bei einem `forward` ohne Optionen nicht gerade aus. 