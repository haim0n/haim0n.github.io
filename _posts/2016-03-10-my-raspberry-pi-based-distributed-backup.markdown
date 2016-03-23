---
title: Complete Guide to Raspberry Pi Based Distributed Backup
layout: post
categories: Raspberry Pi,Raspbian, Syncthing, Distributed Backup
comments: true
---

# What problem on earth are we trying to solve here ?
> “We demand rigidly defined areas of doubt and uncertainty!”  
- Douglas Adams, The Hitchhiker's Guide to the Galaxy

Full disclosure: I'm a PARANOID. It got worse since my kids were born and the amount of digital media I have to backup
now (pictures, videos, albums and you name it). Since I got sick of always worring that my precious data one day will 
be on it's way to a digital heaven because of the usual things like:
* a fire breaks down in my house when I'm away
* a flood flushes my home 
* theft, planes crashing and other hell freaking stuff.

**So, why not going to the cloud ?**

Well for starters, being a paranoid involves the unwillingness to share any data I own with anyone besides the people
I actually trust (sounds weird, I know).
And let's face it - that cloud materia is not that cheap. Google's Drive for my needs (~700GB) will cost me around 120$/year.

# Suggested Solution
> "My home is my castle, but it's not bulletproof - so lets build more castles!"  
- Some paranoid

![Home setup](/assets/dist_backup_setup.png)

What we'll build is a distributed network of file servers, which will replicate all the precious data across multiple sites.
The replication is done by [Syncthing](https://syncthing.net) software.


# 1. Setup Ingridients (Two of each)
* Raspberry Pi 2 Model B
* External (USB) Hard Drive
* USB power adapter (duh)

# 2. Raspberry Pi preparation
**Download the latest Raspbian image:** 
{% highlight bash %}
wget http://downloads.raspberrypi.org/raspbian/images/raspbian-2016-03-18/2016-03-18-raspbian-jessie.zip && \  
unzip 2016-03-18-raspbian-jessie.zip
{% endhighlight %}

**Burn the resulting image to your SD card device (sample setup /dev/sdc)**
{% highlight bash %}
sudo dd if=2016-02-26-raspbian-jessie-lite.img of=/dev/sdc bs=1M oflag=direct
{% endhighlight %}

**Preparing the first device**

* Insert the SD card to the Raspberry
* Plug in the external USB storage
* Plug in the ethernet cable

**After first boot**

{% highlight bash %}
login: pi
password: raspberry

# update the software and power up the external usb drive
sudo apt-get update -y && \
apt-get install rpi-update && \
rpi-update && \
echo 'max_usb_current=1' >> /boot/config.txt && \

# set the device name
hostnamectl set-hostname pi1.local && \

# switch to systemd as init
apt-get install -y libpam-systemd && \
sudo systemctl restart systemd-logind.service && \
sed -ie 's/$/ init=\/bin\/systemd/' /boot/cmdline.txt && \

# resize the root partition
raspi-config --expand-rootfs && \
reboot
{% endhighlight %}

Wait for the pi to boot and:
{% highlight bash %}
# format the drive with ext4
mkdir /mnt/backup && \
sudo mkfs.ext4 /dev/sda1 && \ 

# mount the external storage under /mnt/backup
echo '/dev/sda1    /mnt/backup     ext4    defaults    0 0' >> /etc/fstab && \
mount -a && \
touch /mnt/backup/test_file && \
rm /mnt/backup/test_file && \
echo "successfully mounted /mnt/backup" && \
reboot
{% endhighlight %}

# 3. Syncthing Installation
{% highlight bash %}
wget -O - https://syncthing.net/release-key.txt | sudo apt-key add - && \
echo "deb http://apt.syncthing.net/ syncthing release" | sudo tee -a /etc/apt/sources.list.d/syncthing-release.list && \
sudo apt-get update && \
apt-get install syncthing -y && \
systemctl --user enable syncthing
{% endhighlight %}

# 4. Syncthing Configuration

Start syncthing, to create the initial set up.
{% highlight bash %}
syncthing
{% endhighlight %}
Kill the process by Ctrl+C in the terminal.

{% highlight bash %}
# edit the configuration
sed -ie 's/127.0.0.1/0.0.0.0/' /home/pi/.config/syncthing/config.xml && \
sed -ie 's/tls=\"false\"/tls=\"true\"/' /home/pi/.config/syncthing/config.xml && \

# start the service
sudo systemctl --user start syncthing
{% endhighlight %}

You are now should be able to connect from your computer to the first pi's web management interface via 
[pi1.local:8384](http://pi1.local:8384) address.

# Next Steps
1. Go over the steps to configure at least another device. 
2. Configure your data settings for syncthing as described in a very good manner [here](https://docs.syncthing.net/intro/getting-started.html#configuring).
3. File a complaint if things are broken for you <haim.daniel@gmail.com>