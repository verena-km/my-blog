---
layout: post
title:  "Infrarotempfänger am Raspberry Pi - Nutzung einer Infrarot-Fernbedienung"
date:   2020-07-23 17:08:49 +0200
tags: raspberrypi fernbedienung infrarot
---

## Verkabelung

Eine Fernsteuerung des Raspi lässt sich auch mit einer Infrarot-Fernbedienung realisieren, Beispielsweise mit der Hauppauge 350. Was man (neben der Fernbedienung) braucht ist ein Infarotempfänger, z.B. den Vishay TSOP4838 IR-Empfänger für ca. 2 Euro und drei Verbindungskabel.

Der TSOP4838 hat 3 Pins, einen für die Versorgungsspannung 3,3 Volt, einen für Ground und einen für die Daten. Diese verbinden wir mit dem Raspi. Der Daten-PIN wird in unserem Beispiel mit dem GPIO-PIN 17 verbunden.

## LIRC installieren

Um unter Linux die Infrarot-Signale zu verarbeiten, benötigen wir das Programm LIRC, dass wir mit folgenden Befehl installieren:

```
$ sudo apt-get install lirc
```

Die Konfiguration von LIRC befindet sich im Verzeichnis `/etc/lirc`. Wir kopieren die nach der Installation dort vorhandene Datei `lirc_options.conf.dist` in die dann verwendete `lirc_options.conf`:

`$ sudo cp /etc/lirc/lirc_options.conf.dist /etc/lirc/lirc_options.conf`

Die `lirc_options.conf` wird dann wie folgt abgeändert: 

```   
driver          = default
device          = /dev/lirc0
```

Der GPIO-Pin der mit dem IR-Empfänger verbunden ist, wird in der Datei `/boot/config.txt` konfiguriert. Dort ist folgender Eintrag vorzunehmen:

```
dtoverlay=gpio-ir,gpio_pin=17
```
Mit folgendem Befehl kann man prüfen, ob LIRC als Dienst läuft.

```
$ sudo systemctl status lircd
```

## Konfiguration einer bestimmten Fernbedienung

Normalerweise kann man mit dem Befehl `irrecord` die Fernbedienung anlernen, dies funktioniert aber unter Raspbian Buster nicht. Infos hierzu siehe:
* [https://raspberrypi.stackexchange.com/questions/104008/lirc-irrecord-wont-record-buster-mode2-works](https://raspberrypi.stackexchange.com/questions/104008/lirc-irrecord-wont-record-buster-mode2-works)
* [https://www.raspberrypi.org/forums/viewtopic.php?t=235256](https://www.raspberrypi.org/forums/viewtopic.php?t=235256)

Ich habe den Lösungvorschlag nicht ausprobiert sondern mich damit beholfen, eine fertige Konfigurationsdatei für meine Fernbedienung zu finden und diese zu nutzen. Eine Liste gibt es unter [http://lirc-remotes.sourceforge.net/remotes-table.html](http://lirc-remotes.sourceforge.net/remotes-table.html)

Für die Hauppauge war das die Datei 
[http://lirc.sourceforge.net/remotes/hauppauge/lircd.conf.hauppauge](http://lirc.sourceforge.net/remotes/hauppauge/lircd.conf.hauppauge) deren Inhalt habe ich in die Datei `/etc/lirc/lircd.conf` kopiert.

Nach Neustart des Daemons mit 

```
$ sudo systemctl restart lircd
```
kann man dann die Funktionsfähigkeit testen mit 
```
$ irw
```

Zu jedem Tastendruck werden zwei bis drei Ausgabezeilen erzeugt:
```
00000000000017b5 00 KEY_PLAY Hauppauge_350
00000000000017b5 01 KEY_PLAY Hauppauge_350
00000000000017b5 02 KEY_PLAY Hauppauge_350
00000000000017a5 00 KEY_OK Hauppauge_350
00000000000017a5 01 KEY_OK Hauppauge_350
00000000000017a5 02 KEY_OK Hauppauge_350
0000000000001781 00 KEY_1 Hauppauge_350
0000000000001781 01 KEY_1 Hauppauge_350
000000000000178b 00 KEY_RED Hauppauge_350
000000000000178b 01 KEY_RED Hauppauge_350
```

## Programme mit der Fernbedienung starten

Mit irexec können - abhängig von der gedrückten Taste auf der Fernbedienung - bestimmte Programme gestartet werden. Was bei welchem Tastendruck passiert, wird in einer Konfigurationsdatei gespeichert, entweder systemweit in der `/etc/lirc/irexec.lircrc` oder für den jeweiligen Benutzer in der Datei `~/.lircrc`

Die Konfigurationsdatei enthält einen Block der folgenden Art für jede belegte Taste:
```
begin
    remote = <Name der Fernbedienung>
    prog = irexec
    button = <Name der Taste>
    config = <Auszuführende Aktion = externes Programm>
    repeat = <Anzahl Wiederholungen (0)>
end
```

Eine Konfiguration sieht beispielsweise so aus:
```
begin
    remote = Hauppauge_350
    prog = irexec
    button = KEY_RED
    config = echo "Es wurde rot gedrückt."
    repeat = 0
end

begin
    remote = Hauppauge_350
    prog = irexec
    button = KEY_1
    config = echo "Es wurde 1 gedrückt."
    repeat = 0
end

begin
    remote = Hauppauge_350
    prog = irexec
    button = KEY_POWER
    config = shutdown now
    repeat = 0
end
```

Man kann dann irexec von der Kommandozeile starten:
```
$ irexec
```
So sieht man beim Druck der Farbtasten die oben konfigurierten Echo-Ausgaben auf der Kommandozeile. Beim Druck auf den POWER-Button wird der Raspberry Pi heruntergefahren. Man kann natürlich auch andere Skripte und Programme starten.
Für eine Funktionalität außerhalb der aktuellen Shell sollte irexec als Daemon laufen. Unter Raspbian Buster ist ein solcher Daemon bereits vorhanden, er muss nur noch aktiviert und gestartet werden.

```
$sudo systemctl enable irexec
$sudo systemctl start irexec
```

Weitere Details zu LIRC sind hier zu finden 
[http://www.netzmafia.de/skripten/hardware/RasPi/Projekt-IR-Fernsteuerung/index.html](http://www.netzmafia.de/skripten/hardware/RasPi/Projekt-IR-Fernsteuerung/index.html)



ir-keytable

https://blog.gordonturner.com/2020/05/31/raspberry-pi-ir-receiver/
https://peppe8o.com/setup-raspberry-pi-infrared-remote-from-terminal/


https://www.instructables.com/Easy-Setup-IR-Remote-Control-Using-LIRC-for-the-Ra/

https://ccse.kennesaw.edu/outreach/raspberrypi/_handouts/ir_receiver.pdf

https://blog.heimetli.ch/raspberry-infrarot-lirc.html

https://askubuntu.com/questions/1272907/reading-ir-codes-from-input-events

https://forum.ubuntuusers.de/topic/infrarot-events-lesen/

https://github.com/peterhinch/micropython_ir