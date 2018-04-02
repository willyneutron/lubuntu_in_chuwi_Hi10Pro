# Installing Lubuntu in Chuwi Hi10 Pro

In this guide, I will try to explain all processes I followed to have a working Lubuntu
17.10 in a Chuwi Hi10 Pro tablet. I hope this will be easily adapted to other Linux flavours.

This is the current state of my system:

| Feature 			| Status 		| Notes 
| ------------------------------|-----------------------|-------------
| Internal storage 		| Working 		| Out of the box.
| SD card slot 			| Not working 		| SD card is not detected.
| HW Accelerated graphics 	| Working 		| Out of the box. You can install Intel graphics updater if you want to.
| Micro USB 			| Pending 		| I have no periphals to connect here.
| Keyboard USB hub	  	| Working 		| Out of the box.
| USB type c port 		| Working/Pending 	| Charging works, but I have no periphals to plug in this post and test.
| Keyboard and touchpad 	| Working 		| Out of the box.
| WiFi 				| Working 		| Out of the box.
| Speakers 			| Working 		| Configuration needed.
| Headphone jack 		| Working 		| Configuration needed.
| Battery control		| Working 		| Out of the box.
| Backlight			| Working 		| Configuration needed.
| Touchscreen			| Partially working 	| Imprecise. Does not rotate when screen does.
| Power button			| Working 		| Out of the box.
| Volumen buttons		| Not working 		| They are recognized by the hardware, but there are issues with sound configuration.
| Suspend			| Not working		| Not waking up from suspend.
| Special function keys		| Working		| Backlight controls work with backlight configuration applied. Other can be configurated.
| HDMI				| Pending		| I have no mini-HDMI connector to test it out.
| Bluetooth			| Working		| Configuration needed.
| Stylus			| Pending		| I don't have any stylus to test with.
| Front camera			| Not working		| Not detected
| Back camera			| Not working		| Not detected
| Light sensor			| Working		| If you need to, reading the value is available, but it is not used to change backlight.	
| Accelerometer			| Working		| Configuration needed.

## Motivation
I wanted to replace my old and really used ASUS EeePC netbook with a newer and relative inexpensive
machine, but it turned to be a really harsh task. Finally, I spoted Intel Cherry Trail tablets
and I decided to give them a try with Linux. So I bought this one from a chinese reseller
because it is cheap and it has good hardware. 

## Disclaimer
Some of the processes described in this guide are dangerous and can damage your device or void it's warranty,
so follow this guide on your own risk!

## Hardware
It seems that Chuwi manufactured two slightly different tablets under the name of Chuwi Hi10Pro.
You can distinguish which one you have by looking at serial number:

 - Version 1: Before serial number Hi10 PQ64G42160905000 (included)
 - Version 2: After serial number Q64G42160905000 

This will be important for some of the configurations we will need to perform later, but it is something
really important if you want to perform any kind of factory reset, like flashing BIOS or any of the
OSs that come preinstalled. You can find the original files and tutorials of how to do so in [Chuwi official
forums](http://forum.chuwi.com/thread-2341-1-1.html).

All this guide has been tested using a version 2 Chuwi, so maybe some of the configurations shown in this 
guide could not work in other tablets.

My Chuwi is loaded with:

Part		| Model						| Notes
----------------|-----------------------------------------------|----------
CPU		| Intel Atom x5-Z8350 @ 1.44GHz (x64) [ARK](https://ark.intel.com/products/93361/Intel-Atom-x5-Z8350-Processor-2M-Cache-up-to-1_92-GHz)| Version 1 have a x5-Z8300. Take into account that processor instruction set is x64, but UEFI is 32 bit! 
Graphics	| Integrated Intel Graphics			| -
Audio		| Integrated Intel sound			| -
Motherboard	| Hampoo V100 					| -
RAM		| 4096 MB of DDR3 1066				| -
Screen		| TV101WUM 10.1-inch IPS display @ 1920x1200 px 224DPI	| [More info](http://ultran.ru/sites/default/files/catalog/svetodiody/brend/datasheets/tv101wum-ad0.pdf)
Multitouch controller |  Silead GSLx68y 			| I2c connected
Internal storage| 64GB of flash storage				| Mine came with Windows 10 Home and Android 5.1
WiFi and Bluetooth	| Realtek RTL8723BS 802.11n SDIO        | [Dual chip](http://www.realtek.com/products/productsView.aspx?Langid=1&PFid=59&Level=5&Conn=4&ProdID=375)
Accelerometer	| Bosch BMA250 					| [More info](http://www1.futureelectronics.com/doc/BOSCH/BMA250-0273141121.pdf), i2c connected
Light sensor	| Capella Micro CM3218 				| [More info](http://www.capellamicro.com.tw/EN/product_c.php?id=52&mode=14&search=), i2c connected
USB Hub		| 3.0 USB hub and 2.0 hub 		| They are visible in ```lsusb```. I believe both USB type A connectors in the detachable keyboard are 2.1. 
 
## Installing base system
I didn't want to mess arround with the SOs installed in the internal memory of my tablet, so I finally 
decided to install Lubuntu in a USB stick and boot from there using the USB hub built into the
keyboard. This way, I could also mess arround with my installation with no worries, the warranty of
my tablet will not be voided, and I also will be able to boot other computers with the same dongle. 

So I used a VM (VirtualBox) to install Lubuntu into my USB stick. I created a new blank VM configured
for Linux with no virtual hard disk drives attached. It is very important to enable EFI support, because
the Chuwi won't boot if Lubuntu is not installed in EFI mode. In VirtualBox, you can change this setting
in Configuration > System > Enable EFI.

After that, I loaded Lubuntu 17.10 ISO ([Lubuntu official download site](https://lubuntu.net/lubuntu-1710-artful-aardvark-released/))
into virtual CD-ROM drive and started up my VM. I continued with the installation with no problems.

This USB stick is ready to boot in pretty much every computer, but not in this Baytrail tablets, because
they have a 32 bit UEFI. To address that, we need to get a bootia32.efi and place it inside EFI partition
of our new USB stick. You can get one [here](https://github.com/jfwells/linux-asus-t100ta/blob/master/boot/bootia32.efi)
thanks to John Wells and his Asus Transformer T1000.

The easiest way to include this new file into the EFI partition is to plug it in another computer, mount the EFI partition,
download the file and copy it in place:

```bash
cd /tmp
wget https://github.com/jfwells/linux-asus-t100ta/raw/master/boot/bootia32.efi
sudo mount /dev/sdX1 /mnt
sudo cp bootia32.efi /mnt/EFI/BOOT
sudo umount /mnt
```

You can use ```lsblk``` to know what ```/dev/sdX1``` partition is the one you are looking for.

So now we are ready to boot from our new USB stick. To force the tablet to boot from USB stick, you will
need to enter the BIOS by pressing Del key while Chuwi logo is on screen during tablet boot.

## First boot
With this new 32bit UEFI, you will be able to boot from the USB stick, but the screen will be in vertical mode.
In order to work better while trying to fix everything, log in, and once in the desktop, open a terminal with
Ctrl-Alt-t and type:

```bash
sudo xrandr --output DSI-1 --rotate right
```

This is also a good moment to install some common dependencies that will be needed to build drivers later on:
```bash
sudo apt-get install build-essential git
```

So let's try now to fix everything up.

HW Accelerated graphics
-----------------------

Accelerated graphics work out of the box, but if you try to, for example, play a youtube video, browser will get stuck.
This is not due to video issues but sound issues, we will fix that later.

**NOTE: If you want to, you can install Intel Accelerated Graphics driver update tool. I have notice a very little
improvement in speed and screen fluency, but I have not measured it in any way, it is just subjective. If you are running
your system from a USB stick, installing these drivers could cause graphic crashes if you try to use the USB stick in
other systems.**

This information is based on a guide published in [slimbook.es](https://slimbook.es/tutoriales/linux/132-que-es-y-como-instalar-intel-graphics-update-tool-for-linux-en-ubuntu-15-04-y-15-10)

To install such drivers, we will need to trick Intel installer, because there is no official release for Ubuntu 17.10
(artful), so we will download a release for 17.04 (zesty) and trick the installer to think we have installed this version instead.

Let's download Intel installer for zesty:

```bash
cd /tmp
wget https://download.01.org/gfx/ubuntu/17.04/main/pool/main/i/intel-graphics-update-tool/intel-graphics-update-tool_2.0.6_amd64.deb
```

Let's also download dependencies:

```bash
wget http://es.archive.ubuntu.com/ubuntu/pool/main/p/packagekit/libpackagekit-glib2-16_0.8.12-1ubuntu5_amd64.deb
sudo dpkg -i libpackagekit-glib2-16_0.8.12-1ubuntu5_amd64.deb
sudo apt-get install fonts-symbola
```

Now let's trick the installer. First, make a backup of lsb-release:
```bash
sudo cp /etc/lsb-release /etc/lsb-release.backup
sudo nano /etc/lsb-release 
```

As you may think, modify this file:
```bash
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=17.04
DISTRIB_CODENAME=zesty
DISTRIB_DESCRIPTION="Ubuntu 17.04"
```

Before installing the package, let's import GPG keys:

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 611B903CAB97EA77
sudo apt-get update
```

Now we are ready to install the package and launch it:

```bash
sudo dpkg -i intel-graphics-update-tool_2.0.6_amd64.deb
sudo intel-graphics-update-tool
```

Installer will now pop up. Just install it as normal.
Do not forget to restore lsb-release:

```bash
sudo mv /etc/lsb-release.backup /etc/lsb-release
```

Sound, speakers and headphone jack
----------------------------------
This information has been recopiled from a issue opened in Daniel Otero's guide to adapt Arch Linux
to the Chuwi Hi 10. You can read it [here](https://github.com/danielotero/linux-on-hi10/issues/8).

In order to make sound work, some ALSA configuration files need to be modified. You can download
the fixed configuration files and install them as follows:

```bash
cd /tmp
wget https://raw.githubusercontent.com/plbossart/UCM/218f2a59bf886221319b3c8666818b8c4acafd40/bytcr_rt5651/HiFi.conf
https://raw.githubusercontent.com/plbossart/UCM/218f2a59bf886221319b3c8666818b8c4acafd40/bytcr_rt5651/bytcr_rt5651.conf
mv HiFi.conf /usr/share/alsa/ucm/bytcr-rt5651
mv bytcr_rt5651.conf /usr/share/alsa/ucm/bytcr-rt5651
```

After a reboot, headphone plug will be working, but speakers will not work. This is because a GPIO needs
to be enabled. To do so, a kernel module needs to be modified. Orochimarufan published the patch to do so 
[here](https://github.com/Orochimarufan/linux/commit/00bc6f7). You need to make this modifications to
the file ```sound/soc/codecs/rt5651.c``` placed inside your kernel source code directory and then rebuild
and install the modified new kernel.

See [Building a new kernel](#customizing-and-compiling-linux-kernel) section on this guide to know how to get kernel source code, recompile and
install it.

Automatic headphone plug detection is not working. So you will need to select which sound output you want to activate using
desktop controls in LXDE taskbar.

Backlight
---------
In order to make the backlight work, some kernel flags needs to be changed:

```bash
CONFIG_PWM=y
CONFIG_PWM_CRC=y
CONFIG_I2C_DESIGNWARE_PLATFORM=y
CONFIG_I2C_DESIGNWARE_PCI=y
CONFIG_INTEL_SOC_PMIC=y
CONFIG_DRM_I915=m

CONFIG_PWM_LPSS=y
CONFIG_PWM_LPSS_PCI=y
CONFIG_PWM_LPSS_PLATFORM=y
CONFIG_X86_INTEL_LPSS=y
```

See [Building a new kernel](#customizing-and-compiling-linux-kernel) section on this guide to know how to get kernel source code, and change configuration flags.

Some of these flags has been taken form Daniel Otero's guide to adapt Arch Linux to the Chuwi Hi 10. You can read it 
[here](https://github.com/danielotero/linux-on-hi10/). This was also featured on freedesktop site [here](https://bugs.freedesktop.org/show_bug.cgi?id=85977#c38).

I weren't able to make it work with kernel flags specified by Daniel, I finally needed to change some of the values, as
I read [here](https://github.com/burzumishi/linux-baytrail-flexx10/issues/15).

As you can read in this last quote, you can get/modify brightness configuration by reading/writing in ```/sys/class/backlight/intel_backlight/actual_brightness```.
Also you can read how to set up brightness after boot by adding it to initramfs. After kernel configuration change,
brightness up and down fn keyboard key combinations will also work fine.

Touchscreen
-----------

**NOTE: I am currently working on this. The screen is now detected, but it is terribly imprecise and needs to be calibrated.
Also it does not rotate when the screen does, so it needs manual rotation using xinput as explained in
[this gist](https://gist.github.com/mildmojo/48e9025070a2ba40795c#file-rotate_desktop-sh-L42).**

In order to get the screen working, screen firmware need to be installed. In this case, only one file will be needed:

```bash
cd /tmp
wget https://github.com/onitake/gsl-firmware/blob/master/firmware/chuwi/hi10_pro-z8350/silead_ts.fw?raw=true
```
The community has been extracting firmwares for this kind of touchscreens. You can get more information [here](https://github.com/onitake/gsl-firmware).
Notice that if you have the first version of the Chuwi Hi10 Pro, you must download a different ```silead_ts.fw```!

Once the firmware is downloaded, it needs to be copied in the right place for the system to find it at startup:

```bash
cp silead_ts.fw /lib/firmware/silead
```

Once the firmware is placed, gslx680 drivers need to be installed, so let's download, build and install them:

```
cd /tmp
git clone https://github.com/onitake/gslx680-acpi
cd gslx680-acpi
make
sudo make install
```

```/etc/modules``` will need to be changed, in order to let the system load the new module. This line needs to be added:

```bash
gslx680_ts_acpi
```

If the screen is not working properly after reboot, see ```/var/log/syslog``` for more details.

Volume hardware buttons and fn key combinations on keyboard
-----------------------------------------------------------
Volume hardware buttons are recognized by the system, if you run ```xev``` and press them, it shows that keypresses are
indeed being detected. But in ```~/.config/openbox/lubuntu-rc.xml``` wrong commands are binded to XF86AudioLowerVolume
and XF86AudioRaiseVolume.

Almost every fn combination (blue icons on keyboard) are detected succesfully, but the have no binded action.

If you want to configure any of the fn key combinations, you can modify ```~/.config/openbox/lubuntu-rc.xml``` to bind
them to some script or command. You can also use ```xev``` to know the ID of the key/combination you are pressing. 

Thanks to [jamontes](https://github.com/jamontes) for his help in these issues.

Bluetooth
---------
Bluetooth is working after installing drivers:

```bash
cd /tmp
git clone https://github.com/lwfinger/rtl8723bs_bt
cd rtl8723bs_bt
make
sudo make install
```

Every time you need to activate bluetooth, a script must be run, so is a good idea to set a root cron job to do so at startup.

Create a new directory, copy contents in it:
```bash
sudo mkdir /opt/rtl8723bs/
sudo cp * /opt/rtl8723bs/
sudo crontab -e
```

Then write the cron job:
```bash
@reboot cd /opt/rtl8723bs/ && ./start_bt.sh 
```

Light sensor
------------
The light sensor will work after installing drivers. These can be found [here](https://github.com/burzumishi/linux-baytrail-flexx10/tree/Readme-stage/kernel/modules/cm3218_light_sensor), Thanks to burzumishi for this!

```bash
cd /tmp
git clone https://github.com/burzumishi/linux-baytrail-flexx10/ -b Readme-stage --single-branch
cd linux-baytrail-flexx10/kernel/modules/cm3218_light_sensor
make
sudo make install
sudo modprobe cm3218
```

Once installed, you can read sensor value like this:

```bash
sudo cat /sys/bus/iio/devices/iio\:device1/in_illuminance0_input
```

Accelerometer and screen rotation
---------------------------------

I have prepared a daemon for this task: [bma250 screen autorrotator](https://github.com/willyneutron/bma250-screen-autorotator)


Customizing and compiling Linux Kernel
--------------------------------------

**NOTE: Compiling kernel is a CPU-intensive task, I recommend making the build step in a different (and more powerful) Ubuntu/Lubuntu machine. If you try
to build the kernel in the Chuwi, it would take several hours (or days).**

First of all, lets find out what version of Linux kernel is installed on the Chuwi:
```bash
uname -r
```
In my case it was ```4.13.0-25-generic```.

Now, we will need to download the kernel sources. Remember to verify you are downloading the correct kernel version
obtained in the last step. Of course, you could build and install another kernel version if you want to.
```bash
apt-get source linux-image-4.13.0-25-generic
```

If you are running this commands in the same linux version you want to build and install, you could also refer to
your kernel version like this:

```bash
apt-get source linux-image-$(uname -r)
```

If you are getting an error saying that no source could be found for this packages, make
sure you have ```deb-src``` repositories activated in ```\etc\apt\sources.list```. If they
were disabled, do not forget to perform an ```apt-get update```.


If you have not build a kernel in the system before, some more packages will be needed for the build:
```
sudo apt-get build-dep linux-image-4.13.0-25-generic
```

You will also need to install ncurses:
```bash
sudo apt-get install libncurses-dev
```

This is a good point to applly GPIO modifications in ```sound/soc/codecs/rt5651.c``` specified in "Sound, speakers and headphone jack" section.
[This](rt5651.c.patch) is the patch for this specific kernel version adapted from Orochimarufan's file.

One of the main reasons building the kernel is necesary, is because some configurations need to be changed.
In order to do so, some permissions need to be granted inside kernel source directory:

```bash
chmod a+x debian/rules
chmod a+x debian/scripts/*
chmod a+x debian/scripts/misc/*
```

Now is time to change this configurations, clean the project and edit the configuration:
```bash
fakeroot debian/rules clean
fakeroot debian/rules editconfigs
```

At this point, you will be prompted with several questions about if you want to edit the configurations associated to
each kernel flavour. Normally you will want to modify the configuration for the generic kernel (```amd64/config.flavour.generic```).

When you agree to edit the configuration for a certain kernel flavour, you will get to a screen like this:

![First kernel configuration screen][kernel1]

In this screen, press ```/`` key to search for all configurations that need to be changed (specified in blacklight section of this guide):

![Kernel search screen][kernel2]

Select which one of the search results you want to change by pressing the number specified
in each result:

![Kernel results screen][kernel3]

In this case, ```I2C_DESIGNWARE_PCI``` does not have the correct value, so
press 1 to return to configuration page and be able to change it's value:

![Kernel configuration screen][kernel4]

In this screen is only necessary to press ```Y``` to include this feature:

![Kernel configuration screen with changed configuration][kernel5]

You could also need to press ```N``` or ```M``` to change the value as desired.

For this kernel version, I only needed to change these configurations:

```bash
CONFIG_I2C_DESIGNWARE_PCI=y

CONFIG_PWM_LPSS=y
CONFIG_PWM_LPSS_PCI=y
CONFIG_PWM_LPSS_PLATFORM=y
```

The rest of them (complete list [here](#backlight)) were already correct.

Do not forget to save the changes before selecting exit. After this, we will be asked once
again if we want to change the configuration for the other kernel flavors.

Before building the kernel, let's change version name to be able to distinghish this new
kernel from the older one once installed (or in GRUB for instance) and making it newer than
the stock one. To do so, let's add a modifier to the first version number of ```debian/changelog```:

![Kernel configuration screen with changed configuration][kernel6]


Once all configurations are changed, build the kernel:
```bash
fakeroot debian/rules binary-headers binary-generic binary-perarch
```

If you are getting module error checks, you may want to build with ```skipmodule=true```:
```bash
fakeroot debian/rules binary-headers binary-generic binary-perarch skipmodule=true
```

Several ```.deb```packages must have been generated at this point. Just install them on the
Chuwi:

```bash
dpkg -i *.deb
```

In the case you want to know all kernel packages you have installed, you can get a list of them:

```bash
dpkg -l | grep linux
```

License
-------
<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.



[kernel1]: /images/kernel1.png
[kernel2]: /images/kernel2.png
[kernel3]: /images/kernel3.png
[kernel4]: /images/kernel4.png
[kernel5]: /images/kernel5.png
[kernel6]: /images/kernel6.png