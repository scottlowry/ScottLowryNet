---
title: "Replacing the Bambu P1S Camera with a Raspberry Pi"
date: 2025-06-02
draft: false
summary: "Use a Raspberry Pi for higher resolution print monitoring."
tags: ["Linux", "Raspberry Pi", "camera-streamer"]
---

# Introduction

I love my Bambu Labs P1S 3D printer. It's by far the coolest piece of technology I own. The only downside is the webcam for remote monitoring prints is abysmal: 720p at only .5 frames per second. This is due to the limited processing power of the ESP32 microcontroller that the P1S uses. A Raspberry Pi, even an older 3B model, is a lot more powerful. I just happen to have a couple laying around plus a 5MP 1080p camera module so I set up a better solution.

## Getting Started

I used the following:
- **Raspberry Pi 3B** (newer Pis would also work)
- **Arducam 5MP 1080p OV5647** camera module*
- **[camera-streamer](https://github.com/ayufan/camera-streamer)** to capture and stream video over HTTP
- **Caddy** as a reverse proxy

**This camera module is old and has limitations which I'll expand on bellow. Use a newer Arducam camera module or the official Raspberry Pi v2 or v3 camera modules for better image quality and more features using camera-streamer.*

## Installation

1. I first flashed the latest version of 32-bit Raspberry Pi OS Lite (Bookworm) onto a micro SD card using the [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager) tool. 

2. I then updated the OS
```bash
sudo apt-get update && sudo apt-get dist-upgrade -y
```

3.  It's recommended to give the GPU enough memory for JPEG re-encoding so I added the following line to the end of `/boot/firmware/config.txt`:
```
gpu_mem=128
```

4. I next tried to install camera-streamer v0.2.8 using the precompiled package on GitHub. That didn't work and resulted in the following error:
```
camera-streamer: symbol lookup error: /lib/aarch64-linux-gnu/libcamera.so.0.1: undefined symbol: _ZN7libpisp22compute_optimal_strideER24pisp_image_format_config
```

5. This is a known issue as camera-streamer relies on libcamera which broke campatibilty after recent updates. There's a [PR](https://github.com/ayufan/camera-streamer/pull/169) to fix the issue that hasn't been merged into the main branch yet so I compiled it myself.
```bash
sudo apt-get -y install libavformat-dev libavutil-dev libavcodec-dev libcamera-dev liblivemedia-dev v4l-utils pkg-config xxd build-essential cmake libssl-dev
git clone https://github.com/ayufan-research/camera-streamer.git --recursive
cd camera-streamer
git fetch origin pull/169/head:pr-169
git checkout pr-169
make
sudo make install
```

6. I then needed to figure out what stream formats the camera module supported so I ran:
```bash
v4l2-ctl -d /dev/video0 --list-formats-ext
```
This showed me I had the option of using `YUYV` which is recommended by camera-streamer.

7. camera-streamer defaults to using `http://localhost:8080` so I ran it with the following conditions so I could access it from my Mac. It also sets ideal settings for the camera.
```bash
sudo camera-streamer \
  --http-listen=0.0.0.0 \
  --http-port=80 \
  --camera-type=libcamera \
  --camera-format=YUYV \
  --camera-width=1920 \
  --camera-height=1080 \
  --camera-fps=15 \
  --camera-nbufs=2 \
  --camera-snapshot.height=900 \
  --camera-stream.height=900 \
  --camera-video.height=900
```

The camera-streamer site has several options like snapshot, stream, video, and WebRTC. This is where the older camera module I'm using shows it's limitations. Stream works but uses MJPEG which is bandwidth heavy. Video has a significant delay which makes it unusable and also doesn't work in all browsers. WebRTC would be ideal except the camera module is too old to support it.

8. camera-streamer provides example `systemd` services for running at startup. Here's mine for the older camera module:
```bash
sudo cat <<EOF > /etc/systemd/system/camera-streamer.service
[Unit]
Description=camera-streamer for Arducam Pi Camera 5MP
After=network.target
ConditionPathExists=/sys/bus/i2c/drivers/ov5647/10-0036/video4linux

[Service]
ExecStart=/usr/local/bin/camera-streamer \
  --http-listen=127.0.0.1 \
  --http-port=8080 \
  --camera-type=libcamera \
  --camera-path=/base/soc/i2c0mux/i2c@1/ov5647@36 \
  --camera-format=YUYV \
  --camera-width=1920 \
  --camera-height=1080 \
  --camera-fps=15 \
  --camera-nbufs=2 \
  --camera-snapshot.height=900 \
  --camera-video.height=900 \
  --camera-stream.height=900

DynamicUser=yes
SupplementaryGroups=video i2c
Restart=always
RestartSec=10
Nice=10
IOSchedulingClass=idle
IOSchedulingPriority=7
CPUWeight=20
AllowedCPUs=1-2
MemoryMax=250M

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl enable /etc/systemd/system/camera-streamer.service
sudo systemctl start camera-streamer.service
```

9. The `systemd` service is using the default `localhost:8080` and since the only feature of camera-streamer I care about is stream I installed Caddy as a reverse proxy that points directly to the stream.
```bash
sudo apt-get install caddy -y
```
Edited `/etc/caddy/CaddyFile`:
```
        handle / {
        rewrite * /stream
        reverse_proxy localhost:8080
        }
```

10. Now if I visit the Raspberry Pi at `http://<pi-IP>` it will proxy to `http://localhost:8080/stream`.

## Final Thoughts

This is only marginally better than the built-in camera. Even though the camera module supports 1080p at 30 fps, I've limited the stream to 900p at 15 fps due to bandwidth. I plan to replace this module with the official Raspberry Pi v2 module at some point so WebRTC can be used with full 1080p resolution.