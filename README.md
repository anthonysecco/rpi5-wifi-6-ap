# Raspberry Pi 5 Wi-Fi 6 Access Point for RV/Boat 🚐📶

![image](https://github.com/user-attachments/assets/587c47b7-064c-4182-a117-315ef1a800a1)


## 📝Project Goal&#x20;

I couldn't find a suitable 2x2 Wi-Fi 6 access point with 2x SMA connectors to connect to my roof-mounted Parsec Great Pyrenees. I wanted something cost-effective that could ideally run OpenWRT with full support. One major advantage of using OpenWRT(https://openwrt.org/) on this setup is the ability to configure the device not only as a Wi-Fi access point but also as a Wi-Fi client (or Wi-Fi as WAN). This feature allows simultaneous station and client modes, making it ideal for connecting to campground Wi-Fi while broadcasting a local network. After some research and testing, I came up with the Bill of Materials (BOM) below.&#x20;

Let's go! 🚀

## 📦Bill of Materials (BOM)&#x20;

* Raspberry Pi 5 (2GB)
* Official Pi 5 Active Cooler
* 1GB or Larger SD Card of your Choice
* 52Pi M02 PCIe Express to Mini PCIe HAT
* AsiaRF AW7915-NPD Wi-Fi 6 Mini PCIe Module
* 30x30x10mm Aluminum Heatsink
* 2x Coaxial Cable 1.13 Diameter SMA RP IPEX Pigtails
* 2x Wi-Fi Dipole Antenna Dual Bands (2.4/5GHz)
* Waveshare Industrial Grade Metal Case (D) for Raspberry Pi 5
* 12V/24V to 5V USB-C Step Down Converter (Type-C Interface, 5A, 25W)
* Parsec Great Pyrenees 11-in-1 Antenna (2 banks of 4x4 Cellular, 1 bank of 2x2 Wifi, GPS)

## 🧩Design Considerations&#x20;

### Integrated Wi-Fi Limitations&#x20;

The integrated Wi-Fi on the RPi5 is decent but not great. It supports both client and AP modes out-of-the-box, which works for quick setups. However, it features only a 1x1 MIMO internal antenna with a transmit power limit of 20 dBm. While it does support 802.11ac with 80 MHz bandwidth, the range and performance are insufficient for coverage outside an RV or boat.&#x20;

### Bandwidth Optimization

The AW7915-NPD supports 80 MHz on 5 GHz, which is adequate for mobile applications. I recommend avoiding UNII 2 (DFS) channels due to radar detection issues that can interrupt connectivity. This is especially important for mobile applications which have radio environments that constantly change. Stick to UNII 1 (36/40/44/48) or UNII 3 (149/153/157/161) channels. For 2.4 GHz, use 20 MHz bandwidth to minimize interference and be a good network citizen.

### Spatial Stream Considerations

The AW7915-NPD supports 2 spatial streams (2x2 MIMO), which is sufficient for mobile setups like RVs and boats. Most client devices such as laptops, tablets, and phones also use 2x2 MIMO due to power and size constraints. While a 4x4 MIMO setup could theoretically improve performance in multi-device environments, the cost and complexity outweigh the marginal benefit, especially in a confined space.&#x20;

### ⚡Power Management&#x20;

The RPi5 can draw significant power during multi-core workloads, but as an access point, CPU demand is usually moderate. A 5V 5A (25W) power supply is recommended. Since the RPi5 does not support PD negotiation, you will need to modify the boot configuration to ensure full power delivery (covered later).&#x20;

The AW7915-NPD Wi-Fi 6 module itself can draw up to 3A on the 3.3V rail, especially when transmitting at maximum power (24 dBm on 2.4 GHz and 23 dBm on 5 GHz). To manage heat, use a suitable heatsink on the Wi-Fi module, as the SoCs can reach up to 110°C during peak load while maintaining performance.&#x20;

## 📝Installation and Configuration

### Hardware 🔨

Hardware assembly is straightforward:

1. Attach the heatsink to the RPI5.
2. Screw the standoffs onto the RPI5 for the HAT.

![image](https://github.com/user-attachments/assets/90fca0e4-18d5-4552-a10a-11c07e917690)

3. Connect the PCIe ribbon cable to the RPI5, then to the HAT.
4. Secure the HAT to the standoffs.
5. Insert the AW7915-NPD into the PCIe slot.

![image](https://github.com/user-attachments/assets/34334264-67a7-4073-9551-c34e5efaa7a9)

6. Affix the heatsink to the Wi-Fi card.

![image](https://github.com/user-attachments/assets/3f1cf3ce-cf71-4b63-b579-5833ef00ac1c)
![image](https://github.com/user-attachments/assets/f6146921-96fa-4abe-a43f-bf0687dcb86b)

7. Punch out necessary holes in the case for the antenna pigtails, Ethernet, USB, etc.
8. Mount the standoffs inside the case.
9. Secure the RPI5 inside the case.
10. Attach the two antenna pigtails to the chassis, then to the wireless card.
11. Attach the cover.
12. Profit🧝‍♂️

### Software 💿

On the software side of things, download the latest image of OpenWRT using Firmware Selector ([https://firmware-selector.openwrt.org/](https://firmware-selector.openwrt.org/)).  This guide was written using verison 24.10.1.

* Search "Raspberry Pi 5"
* Click "Customize installed packages and/or first boot script"
* Add the following packages so the WiFi radio appears on boot: kmod-mt7915e kmod-mt7915-firmware

I find some other packages useful, they're completely optional

| Package              | Description                                           |
| -------------------- | ----------------------------------------------------- |
| lm-sensors           | View CPU temp, Wi-Fi radio temps, heatsink fan speed  |
| htop                 | Nice way to see CPU utilization                       |
| nano                 | Easy text editor                                      |
| luci-app-statistics  | Graphs to visualize CPU, Wi-Fi signal, etc.           |
| collectd-mod-sensors | Enables CPU, Wi-Fi radio temps to be graphed          |
| relayd               | To do an L2 bridge from Wi-Fi Client to LAN if needed |
| luci-proto-relay     | GUI elements to configure the L2 bridge               |

**Preparing the Image**

* Click 'Request Build' - if successful, you'll see the download image buttons after.  Download "Factory (EXT4)". 
* Insert your microSD card into your computer. &#x20;
* I use Balena Etcher ([https://etcher.balena.io/](https://etcher.balena.io/)) to flash the SD card.  Select the image you download, and the SDcard, and flash.
* Once complete, open the contents of the SD card (may need to reinsert) and find /boot/config.txt, open with your favorite text editor

**Tuning RPI5 config.txt** ⚡

This file allows us to make some system changes before OpenWRT loads up.  This can be useful to disable certain things like annoying LEDs blinking and reduce idle power consumption.  Some of these changes are needed to enable the use of the Mini PCIe HAT.  Refer to config.txt in the repo.  You will find a description of each option. 

The optional items (disable HDMI, Onboard Wi-Fi/Bluetooth etc.) all help reduce idle power consumption.  With the AW7915-NPD on, but idle, the system idles around \~4w with these settings.  Under full load it's \~8w.

Once all of these is done, insert the SD card into the RPI5 and fire it up.  The ethernet interface will issue an IP by default.  The default gateway IP will get you the OpenWRT UI.  From there you can configure to your heart's desire.

**Configuration Recommendations**

Here are my suggestions for optimal performance for an RV/Boat:

* 2.4Ghz Radio - Channel 1 or 11 - 20mhz - 802.11ax
* 5Ghz Radio - Channel 149 - 80mhz - 802.11ax

&#x20;

## 🧪 Testing and Performance

To evaluate the performance of the RPi5 access point, I used iPerf3 as a benchmarking tool. The setup involved a Linux server on the local LAN within the van, along with multiple test clients. 

⚠️Caveat Emptor - There are obviously a lot of variables with Wi-Fi.  Height of van roof, what's on the roof, whether you have a ground plane, the height and location of your Wi-Fi client outside the van, and of course any Wi-Fi interference nearby.    

&#x20;

### Test Clients&#x20;

Both clients support 802.11ax (Wi-Fi 6), 2x2 MIMO, 80 MHz, max PHY rate of 1200 Mbps

* ThinkPad Laptop 💻
* iPhone 13 Pro 📱

### Test Setup 1 - Parsec Antenna&#x20;

The Linux server was connected via Ethernet to the local network, while both the laptop and iPhone connected to the access point wirelessly. iPerf3 was used to measure throughput from the clients to the server.  The results?

* The laptop consistently achieved throughput between 400-700 Mbps when inside the van and around 200-400 Mbps when outside the van (\~30ft rear passenger side).
* The iPhone performed similarly, with slightly lower speeds particularly on the transmit side, likely due to antenna and power constraints given its size and battery.

These results indicate that the access point configuration can support high-bandwidth applications and will likely exceed that of the internet connection (4G/5G or Starlink) and should be sufficient for RV or marine applications

👻 The *Van Radio Shadow* effect is very real.  Whichever side the of the roof the antenna is on, performed better on average.  I had, mistakenly, installed the antenna on the driver side rear of the room.  As a result, performance was better on the rear driver side.  For example, the front passenger side could only achieve \~100-200Mbps.  Here's my outdoor Parsec:

![image](https://github.com/user-attachments/assets/2ff5b3a5-6d08-402b-9dc4-9ceed34dfef5)

### Test Setup 2 - Di-Pole Antennas

I wanted to evaluate how using basic dipole antennas would affect performance inside the van. I temporarily replaced the SMA cables with the dipole antennas, and as expected, performance improved. I observed speeds of 500-800 Mbps, roughly a 20% increase in throughput compared to the previous setup.  I did not test outside performance.  

You'll see I opted to use the DIN rail mount.  Here's some Di-Pole testing while monitoring thermals🔥:
![image](https://github.com/user-attachments/assets/76966651-43ab-4133-928e-d829215b2a1f)

## 💡Additional Considerations&#x20;

* **Antenna Placement:** For optimal performance, mount the antenna on the roof, biased to the passenger side of the vehicle. This positioning is recommended because most outdoor activity and Wi-Fi usage typically occur on the passenger side when camping. Placing the antenna here helps mitigate the 'van radio shadow' effect and improves external Wi-Fi coverage. 

* **Cooling:** Install location is important. If the access point is mounted in the upper part of the vehicle, especially inside a cabinet, ambient temperatures can become very high, leading to heat soak issues. Consider using a fan or passive cooling to manage temperatures in hot environments.&#x20;

* **Enclosure:** The metal case helps with heat dissipation but can affect Wi-Fi signal strength. Mount antennas externally if possible. The case can be screwed to a wooden surface or mounted to a DIN rail for secure placement.&#x20;

* **Cable Lengths:** If using an antenna mounted outside the vehicle, route the antenna cables in such a way to minimize the total length. This reduces signal loss to the antenna itself.&#x20;

## 🌱 Future Considerations

As this project continues to evolve, here are a few ideas for future improvements and experiments:

* **Indoor Access Point:** If I find that my Internet connections are regularly exceeding 400Mbps, I may opt to install an additional access point inside the van.  An indoor 802.11ax 2x2 MIMO access point at 80Mhz in the right location can provide 700+ Mbps to most of the vehicle.  At 160mhz that number exceeds 1.2Gbps of actual throughput.  The downside is that 160mhz required the use of DFS for a contiguous block of spectrum.  Also an additional access point means another \~4W of idle power draw.  I'm not sure how well clients will switch when moving between indoor vs outdoor access points.
* **Extended Range Testing:** It may be interesting to conduct range tests on 2.4ghz and 5Ghz various outdoor environments, campgrounds or remote campsites, to gauge signal reliability and throughput.  It could be interesting to see how clients handle switching between the two bands when using the same SSID.

## Conclusion

Overall, I'm happy with the results. 200-500Mbps inside and outside the van is sufficient performance.  This number will likely exceed the available Internet bandwidth out in the wild.  I definitely made a mistake on antenna location.  I should've put that on the passenger side of the roof for a bit extra performance outside, but such is life.  🤷‍♂️

Let me know if you've found this useful!

## 📄License

This project is licensed under the MIT License. For more details, see the LICENSE file.&#x20;
