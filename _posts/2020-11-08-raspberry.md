---
layout: post
title:  "Der Raspberry Pi"
date:   2020-11-08
tags: Raspberry Microcontroller Pinout
---

##  Der Raspberry Pi

Der Raspberry Pi ist ein Einplatinencomputer, bei dem meist ein angepasstes Linux als Betriebssystem zum Einsatz kommt. Er ist ein kleiner vollständiger Computer, an den man einen Monitor, eine Tastatur und eine Maus anschließen kann. Die Ausstattung ist abhängig vom Modell. Ich nutze die folgenden Raspis:
* Raspberry Pi Zero W
* Raspberry Pi 3 Model B Plus Rev 1.3

![Raspberry Pi 3 Model B Plus und Raspberry Pi Zero W](/images/foto_raspberry_3_und_zero.jpg) 

Diese Raspis verfügen über 40 GPIO-Pins, WLAN und Bluetooth und einen Kameraanschluss. Eine analoge Soundausgabe ist mit dem Raspberry Pi Zero W nicht möglich.

Programmiert wird der Raspi oft in Python. Es sind aber auch andere Programmiersprachen möglich. Dabei kann man das Programm direkt auf dem Rapi erstellen.

## Inbetriebnahme eines Raspbery Pi 

Ich beschreibe hier, wie man einen Raspberry Pi in Betrieb nehmen kann, ohne dass man an diesen direkt einen Monitor, eine Tastatur und eine Maus anschließen muss. Man kann dann auf den Raspi von einem PC über WLAN per ssh oder über eines der Remote-Desktop-Protokolle VNC oder RDP zugreifen.

### Schreiben der SD-Karte

Man braucht zunächst einen PC, mit dem man eine SD-Karte schreiben kann, sowie eine leere SD-Karte mit mindestens 4 GB. Da ich einen Linux-PC verwende, beschreibe ich hier die Vorgehensweise unter Linux.

Zunächst laden wir das neuste Image von "Raspberry Pi OS" von der Raspberry-Pi-Website [https://www.raspberrypi.org/downloads/raspberry-pi-os/](https://www.raspberrypi.org/downloads/raspberry-pi-os/) herunter. Hier kann man wählen zwischen:

* Raspberry Pi OS (32-bit) with desktop and recommended software
* Raspberry Pi OS (32-bit) with desktop
* Raspberry Pi OS (32-bit) Lite

Hier empfiehlt sich die zweite Option zu wählen. Falls man auf eine grafische Oberfläche auf dem Raspi verzichten kann ist auch die letzte Variante gut.

Nun schreiben wir das Image auf die SD-Karte. Hier muss man unter Linux zunächst feststellen, welchen Device-Namen, das SD-Karten-Device hat. Dazu ruft man folgenden Befehl einmal mit und einmal ohne eingelegte SD-Karte auf:
```
$ lsblk
```
Der Name der nur bei eingelegter SD-Karte erscheint ist der Device-Name, bei mir z.B. /dev/sdb. Dieser ist in den nachfolgenden Befehlen zu verwenden.

Je nach dem, was auf der SD-Karte drauf ist, wird diese automatisch gemounted. Bevor man auf die Karte was schreiben kann, muss man sie unmounten.

Mit dem Befehl 

	$ mount | grep /dev/sdb

kann man sehen, was von der Karte alles gemounted ist.

Die entsprechenden Device-Dateien kann man mit dem Befehl umount unmounten,
beispielsweise:

	$ umount /dev/sdb5
	$ umount /dev/sdb7
	$ umount /dev/sdb6

Danach kann man mit 
	$ sudo parted /dev/sdb mklabel msdos

eine neue Partitionstabelle erstellen.

Das heruntergeladene Datei (bei mir 2020-08-20-raspios-buster.armhf.zip) wird zunächst entpackt.

	$ unzip 2020-08-20-raspios-buster.armhf.zip

Danach kann diese mit dem dd-Befehl auf die SD-Karte übertragen werden:

	$ sudo dd if=2020-08-20-raspios-buster.armhf.img of=/dev/sdb bs=1M

Das geht eine ganze Weile. 

### Vorbereitung der SD-Karte für SSH und WLAN

Nachdem die Daten übertragen wurden, bereiten wir die SD-Karte für den Betrieb am Raspi ohne Monitor und Tastatur vor. Dafür muss man 
* ssh aktivieren
* ein WLAN konfigurieren, mit dem sich der Raspi später verbinden kann.

Um dies zu konfigurieren entfernen wir die SD-Karte aus dem Computer und stecken sie dann gleich wieder ein. Daraufhin werden zwei Verzeichnisse von der SD-Karte automatisch gemounted. 

Im Verzeichnis /boot auf der Karte legt man eine leere Datei mit dem Namen `ssh` an. Diese sorgt beim erstmaligen Start dafür, dass der ssh-Daemon aktiviert wird. Danach wird die Datei gelöscht.

Weiter legt man im ebenfalls im Verzeichnis /boot eine Datei mit dem Namen wpa_supplicant.conf an. Diese hat den folgenden Inhalt:
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
	ssid="YOUR_SSID"
	psk="YOUR_WIFI_PASSWORD"
	key_mgmt=WPA-PSK
}
```
Dabei sind die SSID und das Passwort des zu nutztenden WLANs einzutragen.

Nach dem Unmount von /boot und /rootfs kann nun die SD-Karte entnommen und in den Raspi gesteckt werden.

### Start und Erstkonfiguration

Nachdem nun die SD-Karte im Raspi steckt kann dieser mit Strom versorgt werden und er startet dann automatisch. Wenn alles glatt läuft, baut er eine Verbindung zu dem angegebenen WLAN auf und erhält von dort seine IP-Adresse. 

Ein frisch installierter Raspi hat zunächst den Rechnernamen `raspberrypi`. Befindet man sich im gleichen Subnetz wie der Raspi, kann man in der Regel über diesen Namen auf den Raspi zugreifen. Falls das nicht klappt, kann man im WLAN-Router schauen, welche IP-Adresse der Raspi bekommen hat, und über diese zugreifen.

```
$ ssh pi@raspberrypi
```

Das Startpasswort steht auf 'raspberry'. Wir ändern es gleich.

Wichtige Konfigurationseinstellungen kann man nun mit

```
$ sudo raspi-config
```

durchführen. Nutzt man `raspi-config` aus dem Terminal erscheint die Konfigurationsoberfläche, die man gut mit der Tastatur bedienen kann: Mit den Pfeiltasten kann man die einzelnen Optionen auswählen und mit `Tab` nach unten zum Bestätigen bzw. zum Abbrechen der Dialoge springen   Wir ändern folgendes:

* Unter Punkt 1 das Passwort
* Unter Punkt 2 - N1 den Hostname

Danach bringen wir die Softwarepakete auf dem Raspi auf den neusten Stand:
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

##  Remote-Zugriff auf den Raspi

Es gibt mehrere Möglichkeiten, wie man von einem PC auf einen Raspi zugreifen kann. Es kommt dabei auf das verwendete Betriebssystem, die Art der Nutzung und die persönlichen Vorlieben an. Einige Möglichkeiten für den Zugriff von einem Linux-PC möchte ich hier zeigen:

### Zugriff über SSH

Diese Art des Zugriffs haben wir oben schon gezeigt. Statt jedesmal ein Passwort eingeben zu müssen kann man auf dem PC auch ein Schlüsselpaar generieren und den öffentlichen Schlüssel auf den Raspi kopieren:

```
$ ssh-keygen -t rsa -b 4096
$ ssh-copy-id -i ./id_rsa.pub pi@steinlaus
```

Mit dem Dateimanger auf dem Linuxsystem (z.B. Gnome) kann man sich dann ebenfalls mit dem Raspi verbinden, indem man folgende Serveradresse eingibt: `sftp://pi@steinlaus/`

Darüber kann man dann beispielsweise auch Dateien und Verzeichnisse auf dem Raspi anlegen und mit den Programmen bearbeiten, die man auf dem PC installiert hat. So kann man z.B. den Editor nutzen, den man gewohnt ist.

### Nutzung von Visual Studio Code mit der Remote-SSH-Extensions

Visual Studio Code (VSCode) ist ein beliebter Editor von Microsoft. Mit der Remote-SSH-Extension kann man mit VSCode auch Dateien bearbeiten, die auf einem entfernten Server liegen, der über ssh erreicht werden kann. Zudem kann man aus VSCode heraus Terminals auf dem Server öffnen. 

Das ganze funktioniert nicht nur auf einem Linux-PC, sondern auch unter Windows.

Aber wegen zu wenig RAM leider für den Zugriff auf einen Raspberry Pi Zero [https://github.com/microsoft/vscode-remote-release/issues/669](https://github.com/microsoft/vscode-remote-release/issues/669)

### Zugriff über das Remote-Desktop-Protokoll

Sofern man den Raspi mit grafischer Oberfläche installiert hat, kann man auf diese mit RDP zugreifen, nachdem man das aktiviert hat.

Unter Windows kann man `Microsoft Remote Desktop` hierfür verwenden, unter Linux z.B. das Programm `Remmina`

Um auf den Raspi den RDP-Server zu aktivieren muss man zunächst den VNC-Server deinstallieren und dann den xrdp-Server installieren
```
$ sudo apt-get purge realvnc-vnc-server
$ sudo apt-get install xrdp
```

## Pinout und Nummerierung

Eine schöne Übersicht gibt es hier: [https://de.pinout.xyz/](https://de.pinout.xyz/)

Bei der Nummerierung der Pins muss man aufpassen, denn es gibt mehrere:

* die pyhsikalische Pinnummer
* die BCM-Nummer
* die von der WiringPi-Bibliothek verwendete Nummer


## Programmierung der GPIO-Pins mit Python

Python und viele der benötigten Bibliotheken sind auf dem Raspi vorinstalliert. Für die Nutzung der GPIO-Pins gibt es in Python mehrere Bibliotheken:

* RPI.GPIO
* RPIO
* pigpio
* gpiozero
* WiringPi


### RPi.GPIO

Die Bibliothek [RPI.GPIO](https://pypi.org/project/RPi.GPIO/) wird in vielen Beispielen genutzt, um  die GPIO-Pins anzusteuern. Hier kann man gut die einzelnen Schritte der Beschaltung der Pins nachvollziehen. 

Man kann angeben, welches Nummerierungsschema man verwenden will: GPIO.BCM oder GPIO.BOARD

Hier das An- und Ausschalten einer LED an GPIO 23 als Beispiel in RPi.GPIO
```python
import RPi.GPIO as GPIO
import time
GPIO.setmode(GPIO.BCM)	# Verwendung des BCM-Nummerierungsschemas
GPIO.setup(23, GPIO.OUT)
GPIO.output(23, GPIO.HIGH)
time.sleep(0.5)
GPIO.output(23, GPIO.LOW)
```

### RPIO

Neben RPi.GPIO gibt es noch [RPIO](https://pypi.org/project/RPIO/), damit habe ich mich allerdings bislang noch nicht beschäftigt.

### pigpio

Eine weitere Bibliothek zur Ansteuerung der Pins  ist [pigpio](http://abyz.me.uk/rpi/pigpio/). Damit man pigpio in Python-Programmen nutzen kann, muss unter Linux der pigpiod-Daemon gestartet sein. Auf der Kommandozeile kann man dies machen mit:

```
sudo systemctl start pigpiod
```
Um den pigpiod-Daemen dauerhaft zu starten

```
sudo systemctl enable pigpiod
```
Durch die systemd-Konfiguration unter /lib/systemd/system/pigpiod.service wird pigpiod mit der Option -l gestartet, was bedeutet, dass das Remote-Socket-Interface deaktiviert ist. Man kann damit nur vom lokalen Rechner auf den pigpiod zugreifen.

Pigpio kann nämlich auch dazu verwendet werden, die GPIO-Pins des Raspi von einem anderen Computer aus über Netzwerk anzusteuern. Um dies zu aktivieren ruft man "sudo raspi-config" auf und geht auf "5 Interfacing Options" - "P8 Remote GPIO aktivieren".

Verwendet weren bei pigpio die BCM-Nummern der Pins.

An- und Ausschalten einer LED:
```python
import pigpio
import time
pi = pigio.pi()
pi.set_mode(17, pigpio.OUTPUT)
pi.write(17,1)
time.sleep(0.5)
pi.write(17,0)
```

Ein weiterer Vorteil von pigpio ist das über die Bibliothek bereitgestellte PWM-Signal, das beispielsweise eine Jitter-freie Ansteuerung eines Servos ermöglicht.

### gpiozero

Die Bibliothek [gpiozero](https://gpiozero.readthedocs.io) baut auf den anderen Biblotheken auf. Sie stellt für viele Kompontenten wie LED, Schalter, Motor etc. eigene Klassen zur Verfügung. Man arbeitet damit auf einem höheren Abstraktionslevel. Welcher "Unterbau" verwendet wird kann man über die sog. "Pin-Factory" bestimmen. Hier stehen rpigpio, rpio, pigpio und native zur Verfügung.

An- und Ausschalten einer LED:
```python
from gpiozero import LED
from time import sleep
led = LED(17)
led.on()
sleep(1)
led.off()
```
### Wiring Pi

[Wiring Pi](http://wiringpi.com/) ist eine weitere GPIO-Bibliothek. Wiring Pi wird nicht mehr weiterentwickelt, so dass wir es hier auch nicht mehr weiter betrachten.
