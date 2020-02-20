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

```
[/]# cat /proc/cpuinfo
Processor       : ARMv6-compatible processor rev 7 (v6l)
BogoMIPS        : 479.23
Features        : swp half thumb fastmult vfp edsp java 
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x0
CPU part        : 0xb76
CPU revision    : 7

Hardware        : FH8833
Revision        : 0000
Serial          : 0000000000000000
```

```
[/]# cat /proc/interrupts 
           CPU0       
  2:          0   FH_INTC  fh_wdt
  3:      31001   FH_INTC  System Timer Tick
  7:      14570   FH_INTC  ispp
  8:      29134   FH_INTC  ispf
  9:      43592   FH_INTC  vpu
 10:      33966   FH_INTC  pae
 11:       1793   FH_INTC  fh_i2c
 12:          0   FH_INTC  fh_i2c
 13:          0   FH_INTC  jpeg
 14:       8872   FH_INTC  bgm
 16:          0   FH_INTC  fh_aes.0
 17:          0   FH_INTC  fh_mci
 18:      29154   FH_INTC  fh_mci
 20:       1257   FH_INTC  fh_sadc.0
 21:          0   FH_INTC  fh_spi.1
 22:          0   FH_INTC  fh_spi_slave.0
 23:          0   FH_INTC  fh_dmac
 28:          0   FH_INTC  fh_spi.0
 30:      12703   FH_INTC  ttyS
 32:      68802   FH_INTC  VMM-BUS
 33:          0   FH_INTC  fh_rtc.0
 36:          0   FH_INTC  fh_pwm
Err:          0
```

```
[/]# cat /proc/iomem 
a0000000-a1dfffff : System RAM
  a0024000-a0389fff : Kernel text
  a038a000-a03c287f : Kernel data
e0300000-e0303fff : fh_dmac.0
  e0300000-e03003ff : fh_dmac
e2000000-e2003fff : fh_mci.0
e2200000-e2203fff : fh_mci.1
e8200000-e8203fff : fh aes mem
  e8200000-e8203fff : fh_aes
f0200000-f0203fff : fh_i2c.0
  f0200000-f0203fff : fh_i2c
f0300000-f0303fff : FH_GPIO.0
  f0300000-f0303fff : FH_GPIO
f0400000-f0403fff : fh_pwm.0
  f0400000-f0403fff : fh_pwm
f0500000-f0503fff : fh spi0 mem
  f0500000-f0503fff : fh_spi
f0600000-f0603fff : fh spi1 mem
  f0600000-f0603fff : fh_spi
f0640000-f0643fff : fh spi2 mem
  f0640000-f0643fff : fh_spi_slave
f0700000-f0703fff : ttyS.0
f0800000-f0803fff : ttyS.1
f0900000-f0903fff : fh_i2s.0
f0a00000-f0a03fff : fh_acw.0
f0b00000-f0b03fff : fh_i2c.1
  f0b00000-f0b03fff : fh_i2c
f0d00000-f0d03fff : fh_wdt.0
  f0d00000-f0d03fff : fh_wdt
f1200000-f1203fff : fh sadc mem
  f1200000-f1203fff : fh_sadc
f1500000-f1503fff : fh_rtc.0
  f1500000-f1503fff : fh_rtc
f1600000-f1603fff : fh_efuse.0
  f1600000-f1603fff : fh_efuse
f4000000-f4003fff : FH_GPIO.1
  f4000000-f4003fff : FH_GPIO
```

```
[/]# cat /proc/kmsg 
<5>[    0.000000] Linux version 3.0.8 (root@mail.cerlluars.com) (gcc version 4.3.2 (crosstool-NG 1.19.0) ) #77 Mon Jun 3 17:55:52 CST 2019
<4>[    0.000000] CPU: ARMv6-compatible processor [410fb767] revision 7 (ARMv7), cr=00c5387f
<4>[    0.000000] CPU: VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
<4>[    0.000000] Machine: FH8833
<4>[    0.000000] Ignoring unrecognised tag 0x41000601
<4>[    0.000000] Memory policy: ECC disabled, Data cache writeback
<7>[    0.000000] On node 0 totalpages: 7680
<7>[    0.000000] free_area_init_node: node 0, pgdat c03a8780, node_mem_map c03c3000
<7>[    0.000000]   Normal zone: 60 pages used for memmap
<7>[    0.000000]   Normal zone: 0 pages reserved
<7>[    0.000000]   Normal zone: 7620 pages, LIFO batch:0
<7>[    0.000000] pcpu-alloc: s0 r0 d32768 u32768 alloc=1*32768
<7>[    0.000000] pcpu-alloc: [0] 0 
<4>[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 7620
<5>[    0.000000] Kernel command line: console=ttyS0,115200 root=/dev/mtdblock2 rootfstype=squashfs init=/squashfs_init mem=30M mtdparts=spi_flash:320K@0x0(uboot),0x1F0000@0x50000(kernel),0x420000@0x240000(rootfs),0x1a0000@0x660000(config),8M@0x0(all)
<6>[    0.000000] PID hash table entries: 128 (order: -3, 512 bytes)
<6>[    0.000000] Dentry cache hash table entries: 4096 (order: 2, 16384 bytes)
<6>[    0.000000] Inode-cache hash table entries: 2048 (order: 1, 8192 bytes)
<6>[    0.000000] Memory: 30MB = 30MB total
<5>[    0.000000] Memory: 26544k/26544k available, 4176k reserved, 0K highmem
<5>[    0.000000] Virtual kernel memory layout:
<5>[    0.000000]     vector  : 0xffff0000 - 0xffff1000   (   4 kB)
<5>[    0.000000]     fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
<5>[    0.000000]     DMA     : 0xffc00000 - 0xffe00000   (   2 MB)
<5>[    0.000000]     vmalloc : 0xc2000000 - 0xfe000000   ( 960 MB)
<5>[    0.000000]     lowmem  : 0xc0000000 - 0xc1e00000   (  30 MB)
<5>[    0.000000]     modules : 0xbf000000 - 0xc0000000   (  16 MB)
<5>[    0.000000]       .init : 0xc0008000 - 0xc0024000   ( 112 kB)
<5>[    0.000000]       .text : 0xc0024000 - 0xc038a000   (3480 kB)
<5>[    0.000000]       .data : 0xc038a000 - 0xc03a8e20   ( 124 kB)
<5>[    0.000000]        .bss : 0xc03a8e44 - 0xc03c2880   ( 103 kB)
<6>[    0.000000] SLUB: Genslabs=13, HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
<6>[    0.000000] NR_IRQS:128
<6>[    0.000000] timer mult: 0xfa000000, timer shift: 0x16
<6>[    0.000000] sched_clock: 32 bits at 1000kHz, resolution 1000ns, wraps every 4294967ms
<4>[    0.000000] fh_set_mode in while 
<6>[    0.000000] Console: colour dummy device 80x30
<6>[    0.000000] console [ttyS0] enabled
<6>[    0.188804] Calibrating delay loop... 479.23 BogoMIPS (lpj=2396160)
<6>[    0.250104] pid_max: default: 32768 minimum: 301
<6>[    0.254665] Mount-cache hash table entries: 512
<6>[    0.259455] CPU: Testing write buffer coherency: ok
<6>[    0.263860] hw perfevents: enabled with v6 PMU driver, 3 counters available
<6>[    0.271469] devtmpfs: initialized
<6>[    0.277239] NET: Registered protocol family 16
<6>[    0.282762] fh8833 board init
<6>[    0.294105] hw-breakpoint: found 6 breakpoint and 1 watchpoint registers.
<6>[    0.297997] hw-breakpoint: maximum watchpoint size is 4 bytes.
<6>[    0.326774] bio: create slab <bio-0> at 0
<6>[    0.332112] fh_dmac fh_dmac.0: FH DMA Controller, 6 channels
<6>[    0.342872] cfg80211: Calling CRDA to update world regulatory domain
<6>[    0.349036] Switching to clocksource fh_clocksource
<6>[    0.401704] NET: Registered protocol family 2
<6>[    0.403395] IP route cache hash table entries: 1024 (order: 0, 4096 bytes)
<6>[    0.410496] TCP established hash table entries: 1024 (order: 1, 8192 bytes)
<6>[    0.416823] TCP bind hash table entries: 1024 (order: 0, 4096 bytes)
<6>[    0.423059] TCP: Hash tables configured (established 1024 bind 1024)
<6>[    0.429222] TCP reno registered
<6>[    0.432338] UDP hash table entries: 256 (order: 0, 4096 bytes)
<6>[    0.438048] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes)
<6>[    0.444569] NET: Registered protocol family 1
<6>[    0.483143] squashfs: version 4.0 (2009/01/31) Phillip Lougher
<6>[    0.489561] JFFS2 version 2.2. (NAND) © 2001-2006 Red Hat, Inc.
<6>[    0.494927] msgmni has been set to 51
<6>[    0.498824] NET: Registered protocol family 38
<6>[    0.501477] io scheduler noop registered (default)
<6>[    0.507838] PWM driver, Number: 8, IO base addr: 0xc2068000
<6>[    0.512273] ttyS.0: ttyS0 at MMIO 0xf0700000 (irq = 30) is a ttyS
<7>[    0.516198] fh serial probe done
<6>[    0.516294] ttyS.1: ttyS1 at MMIO 0xf0800000 (irq = 31) is a ttyS
<7>[    0.522327] fh serial probe done
<6>[    0.524330] brd: module loaded
<6>[    0.535663] loop: module loaded
<6>[    0.541062] CLK misc driver init successfully
<4>[    0.546492] m25p80 spi0.0: found mx25l6405d, expected m25p80
<6>[    0.549281] m25p80 spi0.0: mx25l6405d (8192 Kbytes)
<4>[    0.554104] DEBUG-CMDLINE-PART: parsing <320K@0x0(uboot),0x1F0000@0x50000(kernel),0x420000@0x240000(rootfs),0x1a0000@0x660000(config),8M@0x0(all)>
<4>[    0.566978] DEBUG-CMDLINE-PART: partition 4: name <all>, offset 0, size 800000, mask flags 0
<4>[    0.575234] DEBUG-CMDLINE-PART: partition 3: name <config>, offset 660000, size 1a0000, mask flags 0
<4>[    0.584187] DEBUG-CMDLINE-PART: partition 2: name <rootfs>, offset 240000, size 420000, mask flags 0
<4>[    0.593145] DEBUG-CMDLINE-PART: partition 1: name <kernel>, offset 50000, size 1f0000, mask flags 0
<4>[    0.602021] DEBUG-CMDLINE-PART: partition 0: name <uboot>, offset 0, size 50000, mask flags 0
<4>[    0.610381] DEBUG-CMDLINE-PART: mtdid=<spi_flash> num_parts=<5>
<5>[    0.616177] 5 cmdlinepart partitions found on MTD device spi_flash
<4>[    0.622684] Do the authentication in kernel...
<5>[    0.669589] Creating 5 MTD partitions on "spi_flash":
<5>[    0.671813] 0x000000000000-0x000000050000 : "uboot"
<5>[    0.679655] 0x000000050000-0x000000240000 : "kernel"
<5>[    0.684939] 0x000000240000-0x000000660000 : "rootfs"
<5>[    0.689986] 0x000000660000-0x000000800000 : "config"
<5>[    0.695214] 0x000000000000-0x000000800000 : "all"
<6>[    0.704034] console [netcon0] enabled
<6>[    0.704837] netconsole: network logging started
<6>[    0.710199] fh_rtc fh_rtc.0: rtc core: registered rtc as rtc0
<6>[    0.715499] i2c /dev entries driver
<6>[    0.718838] I2C driver:
<6>[    0.718855]       platform registration... 
<6>[    0.724325]       Clock: 15000khz, Standard-mode HCNT:LCNT = 62:74
<6>[    0.730015]       tx fifo depth: 16, rx fifo depth: 16
<6>[    0.735937]       I2C - (dev. name: fh_i2c - id: 0, IRQ #11
<6>[    0.735954]               IO base addr: 0xc20a0000)
<6>[    0.743500] I2C driver:
<6>[    0.743515]       platform registration... 
<6>[    0.749359]       Clock: 15000khz, Standard-mode HCNT:LCNT = 62:74
<6>[    0.755144]       tx fifo depth: 16, rx fifo depth: 16
<6>[    0.762013]       I2C - (dev. name: fh_i2c - id: 1, IRQ #12
<6>[    0.762031]               IO base addr: 0xc20a8000)
<4>[    0.769717] fh_wdt_drv_probe, start watchdog! 
<4>[    0.772915] [wdt] set topval: 5
<6>[    0.777829] card0 disconnected!
<6>[    0.781778] TCP cubic registered
<6>[    0.782170] NET: Registered protocol family 17
<6>[    0.786550] lib80211: common routines for IEEE802.11 drivers
<7>[    0.792116] lib80211_crypt: registered algorithm 'NULL'
<5>[    0.792145] Registering the dns_resolver key type
<6>[    0.796720] VFP support v0.3: implementor 41 architecture 1 part 20 variant b rev 5
<6>[    0.807371] fh_rtc fh_rtc.0: setting system clock to 1970-01-01 00:00:00 UTC (0)
<6>[    0.815168] aes driver registered
<6>[    0.826012] VFS: Mounted root (squashfs filesystem) readonly on device 31:2.
<6>[    0.830350] Freeing init memory: 112K
<6>[    5.307241] Video Memory Manager
<7>[    5.307663] media_mem_parse_cmdline: s=anonymous,0,0xA1e00000,33728K.
<7>[    5.307695] media_mem_parse_cmdline: i=4.
<7>[    5.307726] vmm:anonymous,a1e00000,20f0000.
<4>[    5.356345] load rtthread_arc.bin at 0xa3f00000, set ring buffer at 0xa3ef0000 with size 0x10000
<6>[    5.362301] get irq offset: 0
<6>[    5.371335] chn0 dev nr: 252
<6>[    5.383090] VBus loaded: 255 in blocks, 255 out blocks
<6>[    5.385350] VBUS driver loaded.
<6>[    5.388424] VBUS load rtthread_arc.bin for rtvbus
<4>[    5.919140] firmware: rtthread_arc.bin loaded
<4>[    5.920800] firmware: rtthread_arc.bin started
<6>[    5.924987] ARC is running.
<4>[    6.030156] media_process: module license 'Proprietary' taints kernel.
<4>[    6.033840] Disabling lock debugging due to kernel taint
<5>[    6.074193] media_process_module_init 250: media process init success!!
<4>[    6.112476] module->phys_addr=0xe8400000  module->virt_addr=0xc2200000 
<6>[    6.116202]       isp - (dev. name: ispp - IRQ #7
<6>[    6.116214]               IO base addr: 0xc2200000)
<4>[    6.124261] module->phys_addr=0xe8500000  module->virt_addr=0xc2400000 
<6>[    6.130622]       isp - (dev. name: ispf - IRQ #8
<6>[    6.130634]               IO base addr: 0xc2400000)
<4>[    6.138585] 
<4>[    6.138593]       stat buffer size: 18KBytes
<5>[    7.130182] vpu_module_init 652: vpu init success!!
<7>[    7.477043] pae_cpu_rest_lb2 2126: [PAE]---RESET PAE---
<7>[    7.510873] pae_load_firmware 36: START LOAD ENC FIRMWARE V2 ...
<7>[    7.511004] pae_load_firmware 39: LOAD ENC FIRMWARE FINISH!
<5>[    7.511031] pae_module_init 808: PAE init success!
<5>[    7.592690] jpeg_module_init 179: JPEG init success!
<5>[    7.662590] bgm_module_init 459: bgm init success!!
<6>[    7.891517] exFAT: Version 1.2.9
<6>[    7.927095] pwm motor dev nr: 251
<6>[    7.938510] PWM_CLOCK = 500000
<7>[    7.939668] vpu_parse_cmdline 298: VPU Revc Command: buf_0_3
<7>[    7.939714] vpu_parse_cmdline 434: set command success! (buf)
<7>[    7.940131] vpu_parse_cmdline 298: VPU Revc Command: buf_1_2
<7>[    7.940178] vpu_parse_cmdline 434: set command success! (buf)
<7>[    7.940568] pae_parse_cmdline 354: ENC Revc Command: viw_2
<7>[    7.940625] pae_parse_cmdline 671: set command success! (viw)
<7>[    7.941184] pae_parse_cmdline 354: ENC Revc Command: stm_2883584
<7>[    7.941245] pae_parse_cmdline 671: set command success! (stm)
<4>[    7.942718] SD1:
<4>[    7.942759] 
<4>[    9.166391] mmc1: card claims to support voltages below the defined range. These will be ignored.
<6>[    9.185779] mmc1: new high speed SDIO card at address 0001
<4>[   10.938505] HW EFUSE
<4>[   10.938545] 0x000: 29 81 03 CC 00 00 50 00    00 00 04 CC 0A 0C 00 00
<4>[   10.944175] 0x010: 23 23 23 24 24 24 28 28    28 29 29 02 FF FF FF FF
<4>[   10.950490] 0x020: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   10.956804] 0x030: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   10.963119] 0x040: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   10.969433] 0x050: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   10.975748] 0x060: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   10.982063] 0x070: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   10.988377] 0x080: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   10.994692] 0x090: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.001007] 0x0a0: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.007322] 0x0b0: FF FF FF FF FF FF FF FF    20 17 1C 00 00 00 00 FF
<4>[   11.013636] 0x0c0: FF 11 00 10 00 FF 00 FF    00 00 FF FF FF FF FF FF
<4>[   11.019951] 0x0d0: 3E 10 01 12 23 FF FF FF    20 04 4C 02 79 F1 21 02
<4>[   11.026265] 0x0e0: 0C 00 22 04 00 08 00 32    FF 21 02 0C 00 22 2A 01
<4>[   11.032580] 0x0f0: 01 00 00 00 00 00 00 00    00 00 00 00 02 00 FF FF
<4>[   11.038895] 0x100: 00 00 00 00 00 00 00 00    00 00 00 00 00 00 00 00
<4>[   11.045209] 0x110: 00 EB 00 6E 01 00 00 00    00 FF AC 5D 5C EF B5 9F
<4>[   11.051526] 0x120: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.057838] 0x130: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.064153] 0x140: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.070468] 0x150: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.076782] 0x160: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.083097] 0x170: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.089412] 0x180: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.095727] 0x190: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.102041] 0x1a0: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.108356] 0x1b0: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.114671] 0x1c0: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.120985] 0x1d0: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.127300] 0x1e0: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.133615] 0x1f0: FF FF FF FF FF FF FF FF    FF FF FF FF FF FF FF FF
<4>[   11.139928] 
<4>[   11.567014] MSC msc_proc_create: c237c000
<4>[   13.351356] fh_wdt_ioctl, watchdog is enabled! 
<4>[   16.780859] [MSC] msc_cmd_handler stop 5
<4>[   16.845239] kernel space : ENCRPTGET_DATA1=====================
<4>[   16.851204] kernel space : ENCRPTGET_DATA1=====================
<7>[   16.918473] vpu_parse_cmdline 298: VPU Revc Command: cap_0_1920_1080
<7>[   16.918525] vpu_parse_cmdline 434: set command success! (cap)
<7>[   16.918859] vpu_parse_cmdline 298: VPU Revc Command: cap_1_640_360
<7>[   16.918909] vpu_parse_cmdline 434: set command success! (cap)
<7>[   16.919368] fh_media_open 95: media_process open success.
<7>[   16.920178] pae_enc_isr 4086: [PAE]---PAE RUN!!---
<7>[   16.932641] pae_open 587: pae open success!
<7>[   16.933111] jpeg_open 35: jpeg open success.
<7>[   16.933484] vpu_open 486: vpu open success.
<7>[   16.933738] bgm_open 292: bgm open success.
<7>[   16.935249] remap_pfn_range start~end: 0x41143000 0x41158000, pyh:0xa1e00 , nocached.
<7>[   16.936659] remap_pfn_range start~end: 0x43db0000 0x446c6000, pyh:0xa1e15 , nocached.
<7>[   16.937363] remap_pfn_range start~end: 0x4222f000 0x422e4000, pyh:0xa272b , nocached.
<7>[   16.938175] remap_pfn_range start~end: 0x446c6000 0x44a15000, pyh:0xa27e0 , nocached.
<7>[   16.941824] vpu_reg_set_chn_size 1144: chan 0 1040 1088 1920 1920
<7>[   16.941871] vpu_reg_set_chn_size 1144: chan 0 1040 1088 1920 1920
<7>[   16.941933] check_dst_chan 459: [unlegality chan]:chan 1 0x0 0 0
<7>[   16.941964] check_dst_chan 459: [unlegality chan]:chan 5 0x0 0 0
<7>[   16.942215] vpu_reg_set_chn_size 1144: chan 1 346 368 640 640
<7>[   16.942257] vpu_reg_set_chn_size 1144: chan 1 347 368 640 640
<7>[   16.942308] check_dst_chan 459: [unlegality chan]:chan 2 0x0 0 0
<7>[   16.943014] remap_pfn_range start~end: 0x44adf000 0x44c9c000, pyh:0xa2b2f , nocached.
<7>[   16.949827] check_dst_chan 459: [unlegality chan]:chan 17 0x0 0 0
<7>[   16.952233] remap_pfn_range start~end: 0x44d78000 0x45691000, pyh:0xa2cf0 , nocached.
<7>[   18.832653] remap_pfn_range start~end: 0x40081000 0x40083000, pyh:0xa3609 , nocached.
<7>[   18.954102] remap_pfn_range start~end: 0x46fe2000 0x476fb000, pyh:0xa360b , nocached.
<7>[   18.955002] check_dst_chan 459: [unlegality chan]:chan 7 0x0 0 0
<7>[   18.955478] remap_pfn_range start~end: 0x477b3000 0x478b1000, pyh:0xa3d24 , nocached.
<7>[   18.955791] check_dst_chan 459: [unlegality chan]:chan 8 0x0 0 0
<7>[   33.118861] remap_pfn_range start~end: 0x40153000 0x40154000, pyh:0xa3e22 , nocached.
<7>[   33.119256] remap_pfn_range start~end: 0x41159000 0x4115b000, pyh:0xa3e23 , nocached.
<7>[   33.156412] remap_pfn_range start~end: 0x40154000 0x40155000, pyh:0xa3e25 , nocached.
<7>[   33.156647] remap_pfn_range start~end: 0x4115b000 0x4115e000, pyh:0xa3e26 , nocached.
<7>[   37.536496] remap_pfn_range start~end: 0x40155000 0x40156000, pyh:0xa3e29 , nocached.
<7>[   37.536699] remap_pfn_range start~end: 0x41162000 0x41163000, pyh:0xa3e2a , nocached.
<7>[   37.538278] remap_pfn_range start~end: 0x419d9000 0x419da000, pyh:0xa3e2b , nocached.
<7>[   37.538459] remap_pfn_range start~end: 0x422e4000 0x422e5000, pyh:0xa3e2c , nocached.
<4>[   44.909088] rxerr: port ch=0x00, rxs=0x000000f9
<6>[  243.220773] ------------- dump register -------------
<4>[  243.222958] cnt:0x0
<4>[  243.225121] offset:0x0
<4>[  243.227342] fail:0x0
<4>[  243.229500] alarm_cnt:0x0
<4>[  243.232013] int stat:0x0
<4>[  243.234575] int en:0x0
<4>[  243.236781] sync:0x2
<4>[  243.238911] debug:0x0
<6>[  243.241250] -------------------------------------------
```

```
[/]# cat /proc/meminfo 
MemTotal:          26656 kB
MemFree:            1248 kB
Buffers:            1828 kB
Cached:             8040 kB
SwapCached:            0 kB
Active:             8764 kB
Inactive:           6528 kB
Active(anon):       5460 kB
Inactive(anon):       24 kB
Active(file):       3304 kB
Inactive(file):     6504 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:          5444 kB
Mapped:             5160 kB
Shmem:                60 kB
Slab:               3312 kB
SReclaimable:        772 kB
SUnreclaim:         2540 kB
KernelStack:        1160 kB
PageTables:          728 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:       13328 kB
Committed_AS:     750544 kB
VmallocTotal:     983040 kB
VmallocUsed:       19364 kB
VmallocChunk:     957348 kB
```

```
[/]# cat /proc/misc 
 45 fh_audio
 46 bgm
 47 jpeg
 48 pae
 49 vpu
 50 isp
 51 media_process
 52 vmm_userdev
 53 network_throughput
 54 network_latency
 55 cpu_dma_latency
130 watchdog
 56 fh_clk_miscdev
 57 fh_dma_misc
 58 fh_efuse
 59 fh_sadc
 60 fh_pinctrl
 61 fh_pwm
 62 encrypt
 63 rfkill
```

```
[/]# cat /proc/modules 
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
[/]# cat /proc/version 
Linux version 3.0.8 (root@mail.cerlluars.com) (gcc version 4.3.2 (crosstool-NG 1.19.0) ) #77 Mon Jun 3 17:55:52 CST 2019
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


