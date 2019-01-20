Set up RPi
==========

Assuming you are using a RPi, the following instructions will guide you through the process of installing and configuring your machine. If you are using the VirtualBox virtual machine, go to [section 2.](2-set-up-virtual-box.md)

+ Download the Raspbian image from [](http://www.raspberrypi.org/downloads)
  + If your less comfortable with the command line you can download the NOOBS package which will set up the install via a GUI
+ Unpack the ZIP file onto your computer
+ Open up a terminal and identify the SD Card
  + `diskutil list`

```
$ diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI                         209.7 MB   disk0s1
   2:                  Apple_HFS Macintosh HD            250.1 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *7.9 GB     disk1
   1:                 DOS_FAT_32 RPI                     7.9 GB     disk1s1
```

+ Unmount the partition of the SD card (`disk1s1` in the above output)
  + `diskutil unmountDisk /dev/disk1s1`

```
$ diskutil unmountDisk /dev/disk1s1
Unmount of all volumes on disk1 was successful
```

+ Copy the image you downloaded above onto the unmounted SD card
  + `sudo dd bs=1m if=2013-09-25-wheezy-raspbian.img of=/dev/disk1` (*this will take a few minutes*)

```
$ sudo dd bs=1m if=2013-09-25-wheezy-raspbian.img of=/dev/disk1
Password:
2825+0 records in
2825+0 records out
2962227200 bytes transferred in 1795.840785 secs (1649493 bytes/sec)
```

+ Unmount and disconnect the SD card, plug it into your RPi, and turn the RPi on

Configure Raspbian
------------------

With a freshly installed instance of Raspbian, we can start to configure the OS for our purposes.

### General ###

+ Upon successful boot, you should be presented with the "Rasberry Pi Software Configuration Tool (raspi-config)"
+ Navigation can be done with the arrow and tab keys, and Enter/Return can be used to select an option
  + Select option 1, *Expand Filesystem*
  + Change the default user password if desired, but write down the new password somewhere safe
  + Enter option 8, *Advanced Options*
    + Enable SSH to allow remote management of the RPi
  + Other settings should be fine with the defaults, but change anything else required, including language and regional settings
+ Once your all done with `raspi-config`, select *Finish* and Reboot the RPi

### Networking ###

Without a network connection you have an inexepensive and underpowered workstation. While this is neat in itself, its more useful to set up the network to update the system and manage it remotely.

### Wireless ###

+ Open up the `/etc/network/interfaces` file in a text editor
  + `nano` can be used to edit text files on your RPi

```
pi@raspberrypi ~ $ sudo nano /etc/network/interfaces
```

+ Ensure that the `/etc/network/interfaces` file looks like below

```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```

+ Edit the `/etc/wpa_supplicant/wpa_supplicant.conf` file to support your Wireless Access Point
  + Change the *ssid* and *psk* (passkey) to the appropriate values

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
        ssid="wlan-legacy"
        psk="soop3rs3cr3t"
# RSN (for WP2) and WPA (for WPA1)
        proto=RSN
# WPA-PSK or WPA-EAP (Pre-Shared or Enterprise)
        key_mgmt=WPA-PSK
# CCMP or TKIP (for WPA2 or WPA1)
        pairwise=CCMP
# OPEN, SHARED or LEAP
        auth_alg=OPEN
}
```

+ Reboot your RPi to accept new network settings
+ Check that your RPi has an IP address by running {{ifconfig}}

```
pi@raspberrypi ~ $ ifconfig
eth0      Link encap:Ethernet  HWaddr b8:27:eb:8a:14:65  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1104 (1.0 KiB)  TX bytes:1104 (1.0 KiB)

wlan0     Link encap:Ethernet  HWaddr 80:1f:02:af:45:15  
          inet addr:192.168.1.116  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:285 errors:0 dropped:308 overruns:0 frame:0
          TX packets:150 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:40740 (39.7 KiB)  TX bytes:22390 (21.8 KiB)
```

+ If your RPi hasn't connected to your network, double check that DHCP is enabled and the ''SSID'' and ''Passphrase'' settings are correct

### Wired ###

+ The RPi ethernet port is auto-sensing which means that you should just be able to connect an RJ-45 cable from a router/switch to your RPi to get going.
+ If you don't have a router or switch handy, the RPi also (theoretically) supports connecting directly to another computer without the need for a crossover cable

ssh
---

+ Now with your RPi connected to a network, you can start managing it via SSH from another computer.
  + Identify the IP address, either via {{ifconfig}} on your RPi or by checking in the router interface
  + From a computer on the same network, {{ssh}} into your RPi

```
$ ssh pi@192.168.1.116
```

+ Enter your password, which you should have written down. If you haven't changed it the default password is {{raspberry}}

```
$ ssh pi@192.168.1.116
pi@192.168.1.116's password:
Linux raspberrypi 3.6.11+ #538 PREEMPT Fri Aug 30 20:42:08 BST 2013 armv6l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Oct 27 21:10:08 2013
pi@raspberrypi ~ $
```

Updates
-------

+ Update the package definitions on your RPi to the latest versions with the {{sudo apt-get update}} command

```
pi@raspberrypi ~ $ sudo apt-get update
Hit http://raspberrypi.collabora.com wheezy Release.gpg
Get:1 http://mirrordirector.raspbian.org wheezy Release.gpg [490 B]
Get:2 http://mirrordirector.raspbian.org wheezy Release [14.4 kB]                            
Get:3 http://archive.raspberrypi.org wheezy Release.gpg [490 B]          
Get:4 http://archive.raspberrypi.org wheezy Release [7,218 B]            
Hit http://raspberrypi.collabora.com wheezy Release                          
Get:5 http://mirrordirector.raspbian.org wheezy/main armhf Packages [7,415 kB]
Get:6 http://archive.raspberrypi.org wheezy/main armhf Packages [10.4 kB]       
Hit http://raspberrypi.collabora.com wheezy/rpi armhf Packages                        
Ign http://archive.raspberrypi.org wheezy/main Translation-en_GB              
Ign http://raspberrypi.collabora.com wheezy/rpi Translation-en_GB             
Ign http://archive.raspberrypi.org wheezy/main Translation-en
Ign http://raspberrypi.collabora.com wheezy/rpi Translation-en
Get:7 http://mirrordirector.raspbian.org wheezy/contrib armhf Packages [23.3 kB]
Get:8 http://mirrordirector.raspbian.org wheezy/non-free armhf Packages [48.0 kB]
Hit http://mirrordirector.raspbian.org wheezy/rpi armhf Packages
Ign http://mirrordirector.raspbian.org wheezy/contrib Translation-en_GB                                                               
Ign http://mirrordirector.raspbian.org wheezy/contrib Translation-en                                                                  
Ign http://mirrordirector.raspbian.org wheezy/main Translation-en_GB                                                                  
Ign http://mirrordirector.raspbian.org wheezy/main Translation-en                                                                     
Ign http://mirrordirector.raspbian.org wheezy/non-free Translation-en_GB                                                              
Ign http://mirrordirector.raspbian.org wheezy/non-free Translation-en                                                                 
Ign http://mirrordirector.raspbian.org wheezy/rpi Translation-en_GB                                                                   
Ign http://mirrordirector.raspbian.org wheezy/rpi Translation-en                                                                      
Fetched 7,519 kB in 49s (151 kB/s)                                                                                                    
Reading package lists... Done
```

+ Then upgrade the actual packages with the {{sudo apt-get upgrade}} command

```
pi@raspberrypi ~ $ sudo apt-get upgrade
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  apt apt-utils base-files cups-bsd cups-client cups-common curl dmsetup dpkg dpkg-dev ghostscript gnupg gpgv idle initscripts
  libapt-inst1.5 libapt-pkg-dev libapt-pkg4.12 libaudio2 libcups2 libcupsimage2 libcurl3 libcurl3-gnutls libdevmapper-event1.02.1
  libdevmapper1.02.1 libdpkg-perl libgs9 libgs9-common liblvm2app2.2 libsmbclient libsystemd-login0 libwbclient0 libxml2
  linux-libc-dev python python-minimal samba-common scratch smbclient squeak-vm sysv-rc sysvinit sysvinit-utils tzdata xserver-common
  xserver-xorg-core
46 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 59.9 MB of archives.
After this operation, 774 kB disk space will be freed.
Do you want to continue [Y/n]? y
Get:1 http://archive.raspberrypi.org/debian/ wheezy/main squeak-vm armhf 1:4.10.2.2793-1 [718 kB]
Get:2 http://mirrordirector.raspbian.org/raspbian/ wheezy/main base-files armhf 7.1wheezy2+rpi1 [81.1 kB]
Get:3 http://mirrordirector.raspbian.org/raspbian/ wheezy/main dpkg armhf 1.16.12 [2,573 kB]
   .
   .
   .
Setting up liblvm2app2.2:armhf (2.02.95-8+rpi1) ...
Processing triggers for menu ...
Processing triggers for initramfs-tools ...
```

[previous](README.md) | [next](3-set-up-virtuoso.md)
