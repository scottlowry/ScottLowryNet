---
title: "Unprivileged LXC Write Permissions on Proxmox Mountpoints"
date: 2025-06-12
draft: false
summary: "How to configure user mappings between unprivileged LXC containers and the Proxmox host to enable write permissions on mountpoints"
tags: ["LXC", "Proxmox"]
---

# Introduction

My Proxmox host has a 12TB RAID array that I use to store content for various services running in LXC containers. I prefer using unpriviledged containers but this causes service accounts within containers to not have write access to RAID folder mountpoints. This is because unprivileged LXC containers run with a different UID/GID namespace than the host system. When a user is created inside a container (e.g., `useradd -m myuser`), that user gets a UID that's mapped to a different UID on the host system. By default, Proxmox uses a mapping range starting from 100000, so:

- Container UID 1000 → Host UID 101000
- Container UID 1001 → Host UID 101001
- And so on...

# Solution

To fix this, the container users need to be added to Proxmox. Then ownership can be given to folders used as mountpoints. I'm going to use the [Apt-Cacher Ng](/posts/apt-cache-proxy) service as an example. 

1. First I need to find the UID and GUID for the `apt-cacher-ng` user/group in the container. 
```bash
cat /etc/passwd | grep apt-cacher-ng
```
Output:
```
apt-cacher-ng:x:103:112::/var/cache/apt-cacher-ng:/usr/sbin/nologin
```
This informs me that the UID is 103 and the GUID is 112.

2. Now I need to add the user/group to the host, being sure to prepend 100 to the UID and GUID:
```bash
useradd -u 100103 -g 100112 -M -s /usr/sbin/nologin apt-cacher-ng
groupadd -g 100112 apt-cacher-ng
```
This creates the user without a home directory or the ability to log into the host.

3. Finally, I set ownership on the folder being used as the mountpoint on the host:
```bash
chown -R apt-cacher-ng:apt-cacher-ng /RAID/Cache
```

# Conclusion

Now the user for the container service is able to write to the mountpoint.