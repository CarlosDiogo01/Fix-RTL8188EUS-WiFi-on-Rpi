Enable Wi-Fi USB dongles using Realtek RTL818xxxx based chips in RPi 1, 2 and 3.
====

Fixing problems enabling RTL8188EUS, RTL8188EU, RTL8188ETV and other RTL818xxx chips in Raspberry Pi Debian based distros.

This tutorial will guide you to solve problems with USB WiFi dongles that uses some of the previous Realtek Wi-Fi chips on Raspberry Pi systems.

Problem
------------------------

After version 4.19 of GNU/Linux kernel, it seems that some USB Wi-Fi devices that uses certain Realtek chips (RTL818xxx based chips) has lost their out-of-the-box support in Raspberry Pi 1, 2 and 3.
The users will only see the following symptom after connecting these dongles:

![No wireless interfaces found](https://i.ibb.co/598Ld0Z/2020-04-10-162444-1824x984-scrot.png)


This tutorial aims to solve the problems enabling USB Wifi dongles using a [TP Link TL-WN725N](https://www.tp-link.com/us/home-networking/usb-adapter/tl-wn725n/) (RTL8188EUS chip), in a RPi 2 with [Raspbian](https://www.raspbian.org/RaspbianImages).

Tutorial setup
------------------------
- Raspberry Pi 2 
- Raspbian GNU/Linux 10 (buster)
- Kernel version: 4.19.97-v7+
- USB WiFi dongle: [TP Link TL-WN725N](https://www.tp-link.com/us/home-networking/usb-adapter/tl-wn725n/) 


Requirements
------------------------
- A recent GCC compiler and make/dkms;
- Ethernet cable in RPi 2 with alive internet connection;


## Steps

For the next steps make sure that your network cable is plugged in with internet connection.
Your USB WiFi dongle should also be plugged in the RPi during all the procedure.

1. Install Kernel Headers:

	```sh
	$ sudo apt-get install raspberrypi-kernel-headers
	```
	
2. Clone this repo:
	
	```sh
	$ git clone https://github.com/CarlosDiogo01/Fix-RTL8188EUS-WiFi-on-Rpi.git
	```
	
3. Compile and install the driver:
	
	```sh
	$ make all
	$ make install
	```
	
	You can also use DKMS for this step, if you prefer.
	The driver in this repo was prepared to be used in a Raspberry Pi arch system only. 
	If you want to port the driver for other arch (e.g i386) you need to change the platform configuration in Makefile.
	To ensure the RPi arch make compatibility, the Makefile has been configured to **BCM2709**:
	
	```sh
	...
	CONFIG_PLATFORM_I386_PC = n
	CONFIG_PLATFORM_BCM2709 = y
	CONFIG_PLATFORM_ANDROID_X86 = n
	...
	```

4. Insert modules in Linux Kernel:
	
	```sh
	$ sudo insmod /lib/modules/4.19.97-v7+/kernel/drivers/net/wireless/8188eu.ko
	$ sudo modprobe lib80211
	$ sudo modprobe cfg80211
	$ sudo modprobe 8188eu 
	```
	
Notice that kernel version for `insmod` may need to get it right.

5. Reboot your RPi. **Do not get plug off the ethernet cable**.

6. Check if the device is properly recognized:
	
	```sh
	$ lsusb | grep RTL8188
	```
	
7. Edit the config file **wpa_supplicant.conf**:
	
	```sh
	$ sudo geany /etc/wpa_supplicant/wpa_supplicant.conf
	```
	
	For this step you'll need an in-range SSID with fully functional internet connection and his password.
	For example purposes, i'll use my Access Point `CarlosHouse_Hotspot` and password: `62abc1o51l2j`.

	This information will be included in the file and his content should be exactly as follows:
	
	```sh
	ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
	ap_scan=1
	update_config=1

	network={
	ssid="CarlosHouse_Hotspot"
	psk="62abc1o51l2j"
	}
	```

8. Edit the **interfaces** file:

	```sh
	$ sudo geany /etc/network/interfaces
	```
	The file content should be modified to be exactly as follows:	
	
	```sh
	# Include files from /etc/network/interfaces.d:
	source-directory /etc/network/interfaces.d
	 
	auto lo
	iface lo inet loopback
	 
	auto eth0
	iface eth0 inet manual
	 
	allow-hotplug wlan0
	iface wlan0 inet manual
	```
	
9. Restart RPi. When the RPi has restarted, you should now noticed the Wifi icon connected to the SSID you entered in step 7:

	![WiFi Icon](https://i.ibb.co/M9Y9WPj/2020-04-10-192302-1824x984-scrot.png)


Troubleshooting:
------------------------

If after this tutorial you still could not enable the WiFi dongle on your RPi the cause can be:

* This driver is not fully compatible with your specific kernel version. I only tested this procedure with version 4.19.97-v7+ of the kernel.
* There is an hardware problem with your dongle, or with the USB port you're using in your Pi;
* Your USB WiFi dongle does not use one of the RTL8188xxx chips listed in the first line;
* Your RPi is having trouble recognizing the WiFi device. 

For this last point, you should also try the additional procedure if you really know that your chip is from Realtek:

1. List all of your USB connected devices:
	
	```sh
	$ lsusb
	```
	
	If your chip was prorperly recognized by your RPi, you should see a Realtek device listed:

	![Listed Realtek chip](https://i.ibb.co/b6MV4np/2020-04-10-195508-836x134-scrot.png)

2. In the previous screenshot, we found a RTL818xxxx based device. 
So i will search for a **RTL818xxxx** driver in the next command. 
You should perform the next command according with the chip reference listed before.

	```sh
	$ sudo apt-get update && apt-cache search RTL818
	```
	
	![Firmware found](https://i.ibb.co/Kr0N4yT/2020-04-10-201957-684x134-scrot.png)

3. Install this driver from package manager:
	```sh
	$ sudo apt-get install firmware-realtek
	```

4. Perform the step 7, and step 8 and 9.
