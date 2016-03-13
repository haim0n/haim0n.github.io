---
title: my-raspberry-pi-based-distributed-backup
layout: post
categories: Raspberry Pi, BitTorrent Sync, Distributed Storage
---

# Installation

1. Download the latest Raspbian from: http://downloads.raspberrypi.org/raspbian/images/
2. unzip and write the image
3. boot the pi
4. 
{% highlight bash %}
login: pi
password: raspberry
{% endhighlight}
4. update the software
{% highlight bash %}
sudo apt-get update -y && \
apt-get install rpi-update && \
rpi-update && \
echo 'max_usb_current=1' >> /boot/config.txt && \
reboot
{% endhighlight}
5. wait for the pi to boot
6. resize the root partition
7. add swap 
8. mount the external storage and update the /etc/btsync/debconf-default.conf storage_path to store under the external storage



# TBD
* SMB support
* NFS support
* Encryption
* web management and config (all nodes)
* Dropbox && Google drive support
* Behind firewall 

