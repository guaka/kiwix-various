*********************************************************************
*                                                                   *
*                      KIWIX-PLUG                                   *
*                                                                   *
*********************************************************************

This is the documentation about Kiwix-plug for the Raspberry Pi.
Kiwix-plug is a solution allowing everyone to set up an open Wifi
kiosk to deliver contents available in the ZIM format. The solution
is adapated for the Raspberry Pi, use the Kiwix-serve technology and
many open source softwares. This documentation explains how to setup
Kiwix-plug.

This port for Raspberry Pi is still EXPERIMENTAL: it has been tested
only by the developer, so there could be bugs or issues on other
configurations. Please report them on http://reportabug.kiwix.org.

== Overview and installation choices ==

The installation comes in 3 steps:
1. prepare the ZIM files
2. set up the Raspberry Pi
3. install the ZIM files on the storage device

Depending of your hardware and of your required space, you can choose
to install the ZIM files either on an USB key either on the SD card
(if you have a Raspberry Pi model A with a single USB port and you
want to connect an USB Wifi key, you have to install the ZIM files
on the SD card). So you have to choose an USB key or an SD card in
accordance with the required space for the ZIM files.

You can set up the Raspberry Pi either from the Raspberry Pi itself
either with SSH from a remote computer. In the first case, the
Raspberry Pi is called "master" and must be connected to the Internet.
In the second case, the remote computer is called the "master", must
be connected to the Internet, and the Raspberry Pi must be connected
to this computer through the local network.

== Requirements ==
* A Raspberry Pi (http://www.raspberry.org)
* A Wi-Fi USB dongle or any Wi-Fi device, with a Linux driver compatible
  with hostapd: currently hostap, wired, madwifi, test, none, nl80211, bsd
* The system SD card with Raspbian installed there
* Some place on an empty USB key or on the system SD card, to store
  the ZIM files and a few other things
* ZIM files you want to spread
  (http://openzim.org/ZIM_File_Archive)
* A wireline LAN allowing access to Internet with a free RJ45 port
* A master computer with a UNIX and a root access, GNU/Linux is perfect.
  This computer needs a SD card port to setup kiwix-serve in the Raspbian
  system, and possibly a free USB port to put the flash drive to
  setup. You need free space on this system: at least 3x the size
  of the ZIM files you want to copy on the storage device.
* Time: preparing the "master" (mainly downloading the ZIM files and
  computing the full-text search engine indexes) can take hours,
  depending of the size of the contents and the power of your computer.
  Do not wait the last minute!

The setup process has 3 steps, based on 3 scripts:

0 Get the code to do all the stuff

1 Run "setup_master.sh" to prepare everything on your computer. It
  especialy downloads the binaries to install on the plug, compute
  indexes, etc.

2 Run "setup_raspberrypi.sh" to configure your Raspberry Pi.

3 Run "setup_usbkey.sh" to copy everything necessary on your
  storage device

== Retrieving the code ==

The code to setup the Raspberry Pi and the flash drive is available on
Internet here: https://sourceforge.net/p/kiwix/other/

To retrieve it you need to have git (code version control system)
client installed on your computer. If you use a Debian based GNU/Linux
distribution you can type (like Ubuntu): "apt-get install git"
in the console.

To download the code, type:
git clone git://git.code.sf.net/p/kiwix/other kiwix-other

You will get a "kiwix-other/plug" directory. Go inside.

== Setup the master ==

This will do all preparatory work before setting up the Raspberry Pi and
the flash drive.

You need to know at this moment what for contents (ZIM files) you want
to install on the plug computer. Copy thus file in "./data/content/"

Run "./setup_master.sh", this will take many minutes to downloads
binaries for the plug and also compute fulltext indexes for each ZIM
files. Be patient...

Run "./setup_master.sh clean" to clean everything (but not the ZIM
files) was downloaded before re-preparing the master.

== Setup the Raspberry Pi ==

We will now do a minimal configuration of the Raspberry Pi system card.
If your master computer is the Raspberry Pi itself, just execute the
command line below. If your master computer is another computer,
connect your Raspberry Pi to the same local network as the master
computer.

If you configure remotely the Raspberry Pi and if you have changed the
default password or the default user on Raspberry Pi, you will have to
change these at the beginning of 'setup_raspberrypi.sh'.

Run "./setup_raspberrypi.sh", this should do a few things. This will be
fast, because this only install if necessary a few packages, add an
init.d script and initiate the clock.

WARNING: make sure your Wi-Fi USB dongle or Wi-Fi connection has a
Linux driver, and this one is compatible with the Wi-Fi Access Point
software (hostapd). Currently it has only been tested with a D-Link
with the driver rtl8192cu, which requires a special compilation of hostapd.
If your Wi-Fi driver is not nl80211, you have to change this parameter in the
USB key configuration USBKEY/system/conf/hostapd.conf after the files are
copied on the USB key in the next step.

WARNING: before activating Kiwix on your Raspberry Pi, you have to configure
Raspbian to a graphical session (even if the display is not connected) since
the graphical environnement is managing the automatic mount of USB devices.

== Setup the flash storage ==

You need to put a free USB key to your computer and run the script
"./setup_usbkey.sh". This will copy many things to the USB key, so
this can take many minutes, be patient...

If you want to customize the name of the Wi-Fi connection ("KIWIX_PLUG"
by default), open the file "USBKEY/system/kiwix-plug" and change the
name in the first lines.

== Conclusion ==

You should have now a perfectly working kiwix-plug. Remove the RJ45
cable from the Raspberry Pi, put the system SD card and the Wi-Fi device
and restart the Raspberry Pi. You can insert the USB key with the content
either before the starting the Raspberry Pi, either once it is started
(if you remove the USB key, you will have to restart to re-launch the
Wi-Fi connection). You should see now a new Wifi network called
"KIWIX_PLUG" appearing. Connect to it using your laptop/smartophone/tablet,
open a new Web browser windows and go on any website (NB: avoid to select a
HTTPS website). You should land to the Kiwix-plug Welcome page.

Enjoy! To any problem: http://reportabug.kiwix.org

== Bugs/Todo ==
* Managing the auto-mount ourselves instead of relying on the mount by LXDE
* Mount the USB key with fmask=0000,dmask=0022 instead of copying the config
  files in /tmp
* Add a possibility to store the ZIM files and config files in the system SD
  card instead of an external USB key, given the Raspberry Pi model A has
  only one USB plug, which is likely used by a Wi-Fi USB device
* Add some mechanism to properly stop kiwix-serve

== Technical considerations for the port on Raspberry Pi ==

* Mount:
  The USB key could not be mounted during the boot, when /etc/init.d/kiwix-plug
  is launched. If no Kiwix USB key is found, this script waits some Kiwix USB
  key is mounted; this is achieved with the program inotifywait monitoring
  /media. This mechanism use the LXDE feature to auto-mount USB keys, possibly
  usbmount could be used with a hook to launch kiwix-serve.
* Wi-Fi management:
  The Access Point (AP) Wi-Fi is created with hostapd and wpa_supplicant has
  to be deactivated to free up the Wi-Fi interface. The DHCP server dnsmasq has
  to be launched before hostapd to be ready to distribute IP addresses.
* Read rights on the USB key:
  I didn’t succeed in changing the dmask of the FAT32 USB mount, so unprivileged
  programs cannot read the USB key (nginx, awstats, hostapd), so I copied the
  files on the system SD card (exempt the ZIM files, too big); but the binaries
  are quite big and the ZIM are not accessible by nginx. Possibly this could be
  solved by using usbmount. kiwix-serve can read the ZIM files since it has root
  rights.

