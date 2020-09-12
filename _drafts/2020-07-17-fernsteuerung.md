---
layout: post
title:  "Möglichkeiten zur Fernsteuerung"
date:   2020-07-17 21:08:49 +0200
tags: Fernsteuerung 
---

Für die Fernsteuerung eines Microcontrollers und des damit gesteuerten Fahrzeugs bieten sich folgende Möglichkeiten an:
* Nutzung von Bluetooth / Funk
* Nutzung von WLAN
* Nutzung einer Infrarot-Fernbedienung
* Akustische Steuerung / Sprachsteuerung

# Fernsteuerung über Bluetooth / Funk

Der Callipe verfügt über Bluetooth, der Rasberry Pi ebenfalls (ab Model 3B). Den Arduino kann man mit einem entsprechenden Shield nachrüsten. Da auch Smartphones über Bluetooth verfügen, liegt die Idee nahe, das Fahrzeug von Smartphone aus über Bluetooth zu steuern. Entsprechende Apps machen dies möglich.

Eine Alternative ist es, einen anderen Microcontroller als "Fernbedienung" zu verwenden. Beim Calliope bietet sich dass an, da er über einen Beschleunigungssensor und zwei Taster verfügt, was man gut für die Fernsteuerung verwenden kann.

Der Calliope verfügt neben Bluetooth auch über ein eigenes Funk-Protokoll, um mit anderen Calliopes zu kommunizieren.

# Nutzung von WLAN

Während beim Calliope und Arduino kein WLAN-Modul direkt verbaut ist, bringt der Raspberry Pi WLAN (ab Model 3B) mit.

Man kann dann entweder:
* ein vorhandenes WLAN nutzen oder 
* der RaspberryPi stellt eines zur Verfügung, in das sich dann das fernsteuernde Gerät (z. B. Smartphone) einbuchen kann.

Für die Fernsteuerung kann dann der Raspi beispielsweise ein Web-Interface bereitstellen, auf das mit einem Browser vom Smartphone aus zugegriffen werden kann.

Eine Möglichkeit, dies technisch zu realisieren, ist über das Python-Webframework Flask.

# Nutzung einer Infrarot-Fernbedienung

Auch Infrarot-Fernbedienungen können für eine Fernsteuerung genutzt werden. Dazu benötigt man ein Infrarot-Empfänger, der über einen digitalen Eingang angeschlossen wird. Für das Auslesen der Signale nutzt man beim Raspberry Pi dar Programm LIRC, für den Arduino gibt es die Bibliothek "irremote".

Auch für den Calliope gibt es eine Möglichkeit einen IR-Empänger über Seeed anzuschließen: [https://github.com/MKleinSB/pxt-IR-Calliope](https://github.com/MKleinSB/pxt-IR-Calliope)

# Akustische Steuerung / Sprachsteuerung

Im Calliope ist ein Microfon verbaut, dass die Stärke der Umgebungsgeräusche aufnehmen kann. Darüber könnte man eine einfache Steuerung realisieren, z.B. über Klasch-Signale.

Weitergehende Möglichkeiten wird vor allem der Raspi bieten - das ist dann aber ein extra Thema.








