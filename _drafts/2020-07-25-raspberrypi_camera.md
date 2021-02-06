---
layout: post
title:  "Kamera am Raspberry Pi"
date:   2020-07-25 17:08:49 +0200
tags: raspberrypi camera
---

Als Kameramodul für den Raspberry Pi verwenden wir die Pi NoIR Camera V2. Man muss die Kamera anschließen und dann über raspi-config aktivieren. Hierfür gibt es bereits einige Tutorials, beispielsweise

https://maker.pro/raspberry-pi/tutorial/how-to-interface-pi-noir-v2-camera-with-raspberry-pi


## Fotos mit raspistill

```
raspistill -o test.jpg
```

## Videos mit raspivid


## Fotos und Videos mit PiCamera


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


### Livestream über Flask
https://www.youtube.com/watch?v=zfBHD4v8hD0

github.com/EbenKouao/pi-camera-stream-flask