---
title: "Setting Up a Cache Proxy for Debian Updates"
date: 2025-06-12
draft: false
summary: "How to set up Apt-Cacher Ng to cache Debian updates"
tags: ["Linux", "Debian", "Ubuntu", "Proxmox", "Raspbian", "Apt-Cacher Ng"]
---

# Introduction

I have over a dozen Debian and Debian-based (Proxmox, Ubuntu, Raspbian) instances running in my home infra currently. Most of them use the same Debian repositories for `apt` updates. I figured I can save some time and bandwidth by setting up a cache proxy using **Apt-Cacher Ng** to cache updates for common packages. 

# Installation

1. I set up a minimal LXC container using Debian 12 on my Proxmox server:
```bash
pct create 112 RAID:vztmpl/debian-12-standard_*.tar.zst \
  --arch amd64 \
  --hostname ACNg \
  --cores 1 \
  --memory 512 \
  --swap 0 \
  --unprivileged 1 \
  --features nesting=1 \
  --net0 name=eth0,bridge=vmbr1,firewall=1,gw=10.0.0.1,hwaddr=BC:24:11:D5:5A:6A,ip=10.0.0.12/24,tag=1,type=veth \
  --rootfs local-zfs:2 \
  --mp0 /RAID/Cache,mp=/var/cache/apt-cacher-ng \
  --onboot 1 \
  --ostype debian
```
This created a Debian 12 container with minimal specs, static IP on VLAN1, and a mountpoint to use the host's RAID array for cache storage.

2. Then I installed **Apt-Cacher Ng**:
```bash
lxc-attach 112
# Now inside the container
apt update
apt install apt-cacher-ng -y
```

3. Due to the container being unpriviledged, there was some additional set up required for the `apt-cacher-ng` user account to be able to write in `/var/cache/apt-cacher-ng` mapped to `/RAID/Cache` on the host.

Ran this on host to add `apt-cacher-ng` user/group and set ownership:
```bash
useradd -u 100103 -g 100112 -M -s /usr/sbin/nologin apt-cacher-ng
groupadd -g 100112 apt-cacher-ng
chown -R apt-cacher-ng:apt-cacher-ng /RAID/Cache
```

See [Unprivileged LXC Write Permissions on Proxmox Mountpoints](/posts/unpriviledged-lxc-write-permissions/) for more info on why I needed to do this.

4. I then edited `/etc/apt-cacher-ng/acng.conf` to set the IP and hostname:
```
BindAddress: 10.0.0.12 acng.lan.scottlowry.net
```

5. Once all set up, I'm able to view stats for `apt-cacher-ng` by visiting `http://acng.lan.scottlowry.net:3142/acng-report.html` in a browser. 

6. I also created a `cron` job to warm the cache every morning at 7am:
```
0 7 * * * /usr/bin/apt update && /usr/bin/apt upgrade -d -y
```

# Client Set Up

Next I set up clients to use the proxy. I use Ansible to manage them in bulk so I wrote the following playbook to config `apt` to use the proxy.
```yaml
- name: Configure APT cache proxy
  hosts: all
  tasks:
    - name: Set APT cache proxy config
      copy:
        dest: /etc/apt/apt.conf.d/00aptproxy
        content: |
          Acquire::http::Proxy "http://acng.lan.scottlowry.net:3142";
          Acquire::https::Proxy "DIRECT";
        owner: root
        group: root
        mode: '0644'
```
```bash
ansible-playbook apt-cache-proxy.yml
```

#  Drawbacks

There are some drawbacks to this:
1. `apt-cacher-ng` can only cache repositories using HTTP but some third-party repositories require HTTPS. This isn't a big deal for me as most third-party repositories I use are one-offs that wouldn't benefit from caching anyway. It did require adding a second line to the proxy config for `apt` to skip the proxy and use HTTPS directly to avoid errors. 

2. It caches per distro and architecture. This makes sense but limits the functionality for one-offs. All of my LXC containers use Debian 12 amd64 which will share a cache but then my two Raspberry Pis which are running different releases of Raspbian won't. There will be a Raspbian 11 armhf cache and a Raspbian 12 armhf cache since I'm using both versions. It's still handy to have the cache in case I want to spin up another Pi with either OS. 

# Final Thoughts

This was really easy to set up. It'll probably only save me a few gigs per month in bandwidth but it's still worth it to me. I have Xfinity with a 1TB/mo limit so anything to avoid hitting the limit helps. 