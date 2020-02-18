# Introduction

I have one of these cheap "WiFi Smart Cloud Camera", but I don't like the idea to use the cloud and allow one of these devices to connect to my wifi network and discuss with a remote server.

Full specifications (as seen on box stickers):

 - 2MP PTZ Smart Cloud Camera
 - Model: TD-Q6-200W
 - Password: 123456
 - Lens: 3.6mm
 - Power: DC5V
 - Barcode: 20190924001

A "Neye3C User Manual" asks to download the "Neye3C" android App and use the camera's access point to configure the camera.

My embedded linux knowledge is very limited but I would like to kno whether it would be possible to install a custom linux on this hardware (preferably from a clean source), and eventually get a camera serving a simple rtsp server (other things such as motion detection or PTZ would be cool too but let's see what is possible first).

# First approach

I powered the camera and tried to access the wifi access point with my laptop.

I quickly realized that I will have to open the cam and disconnect the speaker screaming loud instructions for wifi connection... Done !

During this, I could have a quick look at the board and I saw `FH8632v2` on the main chip. Could not really access or see anything more at this point (I need a smaller screwdriver).

The cam exposes a wifi access point named `WifiCAM-8049c0ee`. No security here, we can connect and get an ip address from DHCP. Default route and dns server are set to `199.112.211.1`

A quick nmap gives: 
```
Nmap scan report for 199.112.211.1
Host is up (0.0076s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE
21/tcp    open  ftp
23/tcp    open  telnet
53/tcp    open  domain
80/tcp    open  http
8888/tcp  open  sun-answerbook
30000/tcp open  ndmps
```

I manually tried many `root` or `admin` passwords on the telnet server (including the one written on box) in vain.


No way to get inside the ftp server either.

The http page on port 80 asks for installing a plugin and gives a broken link to a `ocxsetup.exe`. Tried to download several ones from the web but could not get one to work in the browser of my virtual machine...

Not sure about the other ports...


# Get telnet access

I tried to use:
 - commix: nothing found, and the http page does not have any form anyway
 - nikto: reports 'interesting directory listing' on /passwd

I put `http://199.112.211.1/passwd` in my browser and GET THE PASSWD FILE... Seriously.

`root:$1$OIqi6jzq$MFDXCYYUxHyGC86C44zRt0:0:0::/root:/bin/sh`

I quickly retrieved the root password from it and logged on telnet.


# Telnet probing

I inserted a sd card in the camera and tried to copy as much as I could from `/` to the sdcard (that gets mounted to /dat/mmcblk0p1).

I tried the following commands to gather informations from system:

```
[/]# uname -a
Linux (FH8632) 3.0.8 #77 Mon Jun 3 17:55:52 CST 2019 armv6l GNU/Linux
```

```
[/]# lsmod
rtl8189ftv 786005 0 - Live 0xbf0b6000
pwm_motor 9820 2 - Live 0xbf0b0000
exfat 93379 0 - Live 0xbf093000
vbus_ac 2741 1 - Live 0xbf08f000 (P)
bgm 36851 1 - Live 0xbf082000 (P)
jpeg 38714 1 - Live 0xbf074000 (P)
enc 199583 1 - Live 0xbf03e000 (P)
vpu 75889 1 - Live 0xbf026000 (P)
isp 25198 1 - Live 0xbf01b000
media_process 26895 6 bgm,jpeg,enc,vpu,isp, Live 0xbf010000 (P)
rtvbus 26122 2 vbus_ac,media_process, Live 0xbf005000
vmm 7510 1 - Live 0xbf000000
```

```
[/]# lsusb
lsusb: /sys/bus/usb/devices: No such file or directory
```

```
[/]# lspci
lspci: /sys/bus/pci/devices: No such file or directory
```

```
[/]# free -m
             total         used         free       shared      buffers
Mem:            26           24            1            0            1
-/+ buffers:                 22            3
Swap:            0            0            0
```

```
[/]# lshw
-sh: lshw: not found
```

```
[/]# lscpu
-sh: lscpu: not found
```

```
[/]# df -m
Filesystem           1M-blocks      Used Available Use% Mounted on
/dev/root                    4         4         0 100% /
tmpfs                       13         0        13   0% /dev
tmpfs                       20         0        20   0% /tmp
tmpfs                        2         1         1  54% /mnt
tmpfs                       13         0        13   0% /var
tmpfs                       13         0        13   0% /home
tmpfs                       20         0        20   0% /dev/shm
/dev/mtdblock3               2         1         1  54% /etc
/dev/mtdblock3               2         1         1  54% /usr/local/etc
/dev/mtdblock3               2         1         1  54% /mnt
/dev/mtdblock3               2         1         1  54% /dat
```

```
[/]# mount
rootfs on / type rootfs (rw)
/dev/root on / type squashfs (ro,relatime)
tmpfs on /dev type tmpfs (rw,relatime,mode=755)
tmpfs on /tmp type tmpfs (rw,relatime,size=20480k)
tmpfs on /mnt type tmpfs (rw,relatime,mode=755)
tmpfs on /var type tmpfs (rw,relatime,mode=755)
tmpfs on /home type tmpfs (rw,relatime,mode=755)
/dev/sys on /sys type sysfs (rw,relatime)
proc on /proc type proc (rw,relatime)
devpts on /dev/pts type devpts (rw,relatime,mode=600)
tmpfs on /dev/shm type tmpfs (rw,relatime,size=20480k)
/dev/mtdblock3 on /etc type jffs2 (rw,relatime)
/dev/mtdblock3 on /usr/local/etc type jffs2 (rw,relatime)
/dev/mtdblock3 on /mnt type jffs2 (rw,relatime)
/dev/mtdblock3 on /dat type jffs2 (rw,relatime)
/dev/mmcblk0p1 on /dat/mmcblk0p1 type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=cp437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
```

```
[/]# ifconfig -a
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:5534 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5534 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:507812 (495.9 KiB)  TX bytes:507812 (495.9 KiB)

p2p0      Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx  
          inet addr:199.112.211.1  Bcast:199.112.211.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5128 errors:0 dropped:40 overruns:0 frame:0
          TX packets:773 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:203290 (198.5 KiB)  TX bytes:100165 (97.8 KiB)

wlan0     Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx  
          inet addr:192.168.0.11  Bcast:192.168.0.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:2594 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```
[/]# ip route
default via 192.168.0.1 dev wlan0 
192.168.0.0/24 dev wlan0  src 192.168.0.11 
199.112.211.0/24 dev p2p0  src 199.112.211.1 
239.239.238.50 dev wlan0 
239.239.239.51 dev wlan0 
```

```
[/]# blkid 
```
# About the dump

A lot of things to be seen here.
First: `/mnt`, `/dat`, `/usr/local/etc`, and `/etc` seem all mounted to the same destination... I don't understand.

The application using most ressources is called `sCamera` (in `/root/edvr/sCamera`).
It stands next to an `update` binary and `update.sh` script that is expecting a file called 'hy_edvr_update.tar.gz'...

Many scripts and executables in `/etc` (probably all accessible from the web server...).  
The `rescue.sh` has the following content:
```
echo "rescue>>>>>>>>>>>>>>>>>>>>>>>"
fw_setenv bootargs 'console=ttyS0,115200 root=/dev/mtdblock2 rootfstype=squashfs init=/squashfs_init mem=30M mtdparts=spi_flash:320K@0x0(uboot),0x1F0000@0x50000(kernel),0x420000@0x240000(rootfs),0x1a0000@0x660000(config),8M@0x0(all)'
fw_printenv bootargs
sync
echo "rescue<<<<<<<<<<<<<<<<<<<<<<<"
```

Found `init` and `squashfs_init` files on `/`.

Several configuration files make reference to a hardware named: `idvr9000`

To be continued...


