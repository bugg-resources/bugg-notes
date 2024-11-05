# Architecture

TODO: explain how the OS build works
Repo is bugg-os

# Building the soundcard driver

Repo is bugg-soundcard-driver. Gets cloned into /opt

The DT blob builds and installs fine in pi-gen. 
The codec gets built against the wrong kernel headers (presumably those of the build machine), so modprobe fails with incorrect format
`modprobe: ERROR: could not insert : Exec format error`

The architecture of the kernel module is correct, though (aarch64).

Can't figure out how to get the kernel module to build correctly on the build machine. Instead, build and install it on the CM4 on the first boot.

# Scratchpad

bugg@bugg-v3:/opt/bugg-recording $ sudo systemctl enable first-boot.service 
Created symlink /etc/systemd/system/multi-user.target.wants/first-boot.service → /etc/systemd/system/first-boot.service.

 ln: failed to create symbolic link '/home/ubuntu/bugg-os/pi-gen/work/BuggOS/stage-bugg-os/rootfs/etc/systemd/system/first-boot-complete.target.wants/first-boot.service': No such file or directory
 