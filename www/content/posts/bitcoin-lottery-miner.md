---
title: "Setting Up a Bitcoin Lottery Miner"
date: 2025-06-17
draft: false
summary: "Use a USB Block Erupter to solo mine Bitcoin"
tags: ["Bitcoin", "CGMiner"]
---

# Introduction

I still have an old Block Erupter, a small USB powered ASIC for mining Bitcoin. It hashes at a puny 333 MH/s, ~0.00000000003634% of Bitcoin's total network hashrate of 916.36 EH/s at the time of this writing. According to ChatGPT I have a 1 in 2,751,000,000,000 chance of finding a block mining solo. Well, I'm feeling lucky so I set it up to mine as a lottery.

![So You're Telling Me There's a Chance?](/assets/posts/bitcoin-lottery-miner/chance.png#center)

# Installation

I opted to use **[CGMiner](https://github.com/ckolivas/cgminer)** installed in a LXC container with the Block Erupter passed through to mine.

## LXC Set Up

1. I created a Debian 12 container with minimal specs:
```bash
pct create 107 RAID:vztmpl/debian-12-standard_*.tar.zst \
  --arch amd64 \
  --hostname BTC \
  --cores 4 \
  --memory 512 \
  --swap 0 \
  --unprivileged 1 \
  --features nesting=1 \
  --net0 name=eth0,bridge=vmbr1,firewall=1,tag=20,type=veth \
  --rootfs local-zfs:2G \
  --onboot 1 \
  --ostype debian
```

2. Next I needed to configure the container to use the Block Erupter. I ran `lsusb` twice on the host; first with the Block Erupter unplugged and then again plugged in. The second time showed the Block Erupter:
```
Bus 001 Device 005: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
```
Then I set the LXC container's root as owner using the Bus (`001`) and Device (`005`) values:
```bash
chown 100000:100000 /dev/bus/usb/001/005
```
Now I needed to find the major and minor device numbers for the Block Erupter:
```bash
ls -l /dev/bus/usb/001/005
```
Output:
```
crw-rw-r-- 1 100000 100000 189, 4 Jun 17 10:09 /dev/bus/usb/001/005
```
Finally I added the following lines to the container's config (`/etc/pve/lxc/107.conf`) now knowing the device major (`189`) and minor (`4`) values:
```
lxc.cgroupv2.devices.allow: c 189:4 rwm
lxc.mount.entry: /dev/bus/usb/001/005 dev/bus/usb/001/005 none bind,optional,create=file
```

## CGMiner Set Up

Now that the container has the Block Erupter set up it was time to install **CGMiner**.

1. First I installed Git and the required dependencies for **CGMiner**:
```bash
apt install git build-essential autoconf automake libtool pkg-config libcurl4-openssl-dev libudev-dev libusb-1.0-0-dev libncurses5-dev libz-dev -y
```

2. Cloned the repo:
```bash
git clone https://github.com/ckolivas/cgminer.git
cd cgminer
```

3. Set install flags to build for Block Erupter (`Icarus`) use and then built:
```bash
CFLAGS="-O2 -Wall -march=native -fcommon" ./autogen.sh --enable-icarus
make
```

## Mining

1. I used [BitAddress](https://www.bitaddress.org/), a JavaScript client-side Bitcoin wallet generator, to generate a new Bitcoin address for mining payouts. 

2. I opted to use a solo mining pool so I wouldn't need to keep a copy of the Bitcoin blockchain stored locally. I'm using [KanoPool](https://kano.is/). Once I created an account with the newly generated Bitcoin wallet address configured for payout, I was ready to start mining. I started **CGMiner** with the worker I set up on KanoPool configured to use KanoPool's Seattle and Los Angeles servers as they're geographically closest to me:
```bash
cgminer -o stratum+tcp://se.kano.is:3333 -u slowry05.worker -p x -o stratum+tcp://la.kano.is:3333 -u slowry05.worker -p x
```

# Additional Set Up

## Screen & Cron

I installed `screen` in the container so I can run `cgminer` as a background task that I can check on:
```bash
apt install screen -y
screen -dmS BTC cgminer -o stratum+tcp://se.kano.is:3333 -u slowry05.worker -p x -o stratum+tcp://la.kano.is:3333 -u slowry05.worker -p x
```
And then set that screen command as a `cron` job to run after reboot.

## Script to Update LXC Config

One huge drawback to this set up is after a host reboot the Block Erupter's device values change and then the container's `root` loses ownership. To fix this I wrote a script to run after a reboot on the host to find the new values, reset ownership, and update the values in the container's config:
```bash
#!/bin/bash

# Run lsusb and capture the output
lsusb_output=$(lsusb)

# Search for the line containing "Silicon Labs CP210x UART Bridge"
device_line=$(echo "$lsusb_output" | grep "Silicon Labs CP210x UART Bridge")

# Extract the Bus and Device numbers
bus_number=$(echo "$device_line" | awk '{print $2}')
device_number=$(echo "$device_line" | awk '{print $4}' | tr -d ':')

# Make LXC root owner of the device
chown 100000:100000 /dev/bus/usb/$bus_number/$device_number

# Run ls -l to get the device info
device_info=$(ls -l /dev/bus/usb/$bus_number/$device_number)

# Extract the "Major, Minor" and convert to "Major:Minor"
major_minor=$(echo "$device_info" | grep -o "[0-9]\+,[ ]*[0-9]\+")
formatted_major_minor=$(echo "$major_minor" | sed 's/, */:/' | tr -d ' ')

# Path to the LXC config file for container 107
lxc_config_path="/etc/pve/lxc/107.conf"

# Update the LXC configuration file
if [[ -f "$lxc_config_path" ]]; then
    # Update lxc.cgroup.devices.allow line
    sed -i "s|^lxc.cgroup.devices.allow: c [0-9]\+:[0-9]\+ rwm|lxc.cgroupv2.devices.allow: c $formatted_major_minor rwm|" "$lxc_config_path"

    # Update lxc.mount.entry line
    sed -i "s|^lxc.mount.entry: /dev/bus/usb/[0-9]\+/[0-9]\+ .*|lxc.mount.entry: /dev/bus/usb/$bus_number/$device_number dev/bus/usb/$bus_number/$device_number none bind,optional,create=file|" "$lxc_config_path"
else
    exit 1
fi
```

# Final Thoughts

Do I expect to actually mine a block? Hell no. This is just for fun. If I was more serious there's newer USB ASIC devices that run circles around the now ancient Block Erupter. For ~$30 I could use this [NerdMiner 75KH](https://amzn.to/3ZePE3Q) USB ASIC instead which is ~225x faster. It makes more sense to buy $30 of Bitcoin and hold it instead though.