---
layout: post
title:  "Kamera am Raspberry Pi"
date:   2020-07-25 17:08:49 +0200
tags: raspberrypi camera
---

Als Kameramodul für den Raspberry Pi verwenden wir die Pi NoIR Camera V2. Man muss die Kamera anschließen und dann über raspi-config aktivieren. Hierfür gibt es bereits einige Tutorials, beispielsweise

https://maker.pro/raspberry-pi/tutorial/how-to-interface-pi-noir-v2-camera-with-raspberry-pi

https://tutorials-raspberrypi.de/aufnahmen-mit-dem-offiziellen-kamera-modul-des-raspberry-pi/

Hilfreich auch die ausführliche offizielle Hilfe

https://www.raspberrypi.org/documentation/configuration/camera.md
https://www.raspberrypi.org/documentation/raspbian/applications/camera.md

## Infos zur Kamera (USB-Kamera)

Mögliche Auflösungen anzeigen:

$ v4l2-ctl --list-formats-ext


## Fotos mit raspistill (Raspi-Kamera)

```
raspistill -o test.jpg
```

## Fotos mit fswebcam (USB-Kamera)


Mit fswebcam kann man einzelne Bilder der Webcam aufnehmen.

```
$ fswebcam image.jpg
```

Eine gute Erklärung der Optionen findet man hier:
http://www.netzmafia.de/skripten/hardware/Webcam/


## Videos mit raspivid (Raspi Camera)


Was allerdings nicht geht, ist die Preview von raspistill und raspivid remote anzuschauhen (z.B. über RDP)
https://www.raspberrypi.org/forums/viewtopic.php?t=58384 


## Fotos und Videos mit PiCamera (Raspi Camera)


In einem Python-Programm kann man mit [PiCamera](https://picamera.readthedocs.io) Fotos machen.

```
from time import sleep
from picamera import PiCamera
camera = PiCamera()
camera.resolution = (1024, 768)
camera.start_preview()

#camera warm-up time
sleep(2)
camera.capture('image.jpg')
```



## Streaming
https://raspberrypi.stackexchange.com/questions/23182/how-to-stream-video-from-raspberry-pi-camera-and-watch-it-live
https://dantheiotman.com/2017/08/23/using-raspivid-for-low-latency-pi-zero-w-video-streaming/

https://raspberrypi.stackexchange.com/questions/26675/modern-way-to-stream-h-264-from-the-raspberry-cam

https://wiki.marcluerssen.de/index.php?title=Raspberry_Pi/Camera_streaming


https://www.videolan.org/streaming-features.html

### Livestream über Flask
https://www.youtube.com/watch?v=zfBHD4v8hD0

github.com/EbenKouao/pi-camera-stream-flask

### RPi-Cam-Web-Interface

https://elinux.org/RPi-Cam-Web-Interface


### Streaming über HTTP
raspivid -o - -t 0 -w 640 -h 480 -fps 40 |cvlc -vvv stream:///dev/stdin --sout '#standard{access=http,mux=ts,dst=:8081}' :demux=h264


Parameter raspivid:
-o  Output Dateiname; "-" steht für Standardout
-t  Timeout - wie lange läuft das Programm in ms; 0 heißt dauerhaft (Stopp mit Ctrl-C)
-w  Breite
-h  Höhe

Parameter cvlc:
-vvv  Debugmodus hoch



### Livestream über RTSP
https://raspberry-projects.com/pi/pi-hardware/raspberry-pi-camera/streaming-video-using-vlc-player

raspivid -o - -t 0 -n -w 600 -h 400 -fps 12 | cvlc -vvv stream:///dev/stdin --sout '#rtp{sdp=rtsp://:8554/}' :demux=h264

Anschauen kann man es dann z.B. mit dem VLC-Media-Player

###  Livestream übr v4l2
sudo modprobe bcm2835-v4l2
cvlc v4l2:///dev/video0 --v4l2-width 1920 --v4l2-height 1080 --v4l2-chroma h264 --sout '#standard{access=http,mux=ts,dst=0.0.0.0:12345}'

### Livestream über tcp

Auf Raspi:
    raspivid -t 0 -l -o tcp://0.0.0.0:3333
Auf Rechner im Netzwerk
    vlc tcp/h264://ameise:3333
    oder über VLC - Netzwerkstream öffnen tcp/h264://ameise:3333

### Livestream über Netcat

https://dantheiotman.com/2017/08/23/using-raspivid-for-low-latency-pi-zero-w-video-streaming/

auf dem Rechner, der das Bild empfangen soll:
    netcat -l -p 4444 | mplayer -fps 60 -cache 2048 -
auf dem Raspi:
    raspivid -t 0 -w 1280 -h 720 -o - | nc 192.168.178.50 4444

Achtung: Rechnername funktioniert nicht!

Echt schnell

### gstreamer
https://raspberrypi.stackexchange.com/questions/22288/how-can-i-get-raspivid-to-skip-h264-encoding-getting-rid-of-5-second-latency-s


### UV4L

### mjpg_streamer
https://raubal-it.com/index.php/2020/11/27/raspberry-pi-raspi-als-webcam-nutzen/
Stapel-Buch

## Nutzung in Kivy
https://www.youtube.com/watch?v=1XcKfstKMFQ

## Nutzung in OpenCV
https://www.youtube.com/watch?v=1XcKfstKMFQ

https://www.youtube.com/watch?v=WQeoO7MI0Bs