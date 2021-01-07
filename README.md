# Ethernet Example for ESP-IDF 4.1 and better

This is an example of using the Olimex ESP32-POE card using Espressif ESP-IDF version 4.1 and higher. It applies to the ISO and non-ISO versions.

In the Olimex repo https://github.com/OLIMEX/ESP32-POE , there are several requests for examples that work with version 4.1, which first showed up in August of 2019.

There were major changes in the initialization sequence in 4.1. From Espressif's example, and Olimex's example, it's
not super trivial to figure out how to configure ethernet, between the old form and the configuration changes needed.

These instructions are correct as of Jan 6, 2021 with ESP-IDF 'latest'. I can't say if they work on ESP-IDF 4.1, or 4.2 release, and if
you want to submit a pull request feel free. They also worked very nicely on my 'rev D' board, and I assume they'll work well with others.


## sdkconfig for Ethernet component

This configuration is under Components / Ethernet.

As Espressif keeps changing `menuconfig` and adding more features, it's best to describe the menu entries that must be changed
and what to change them to. That means you don't have to use the `sdkconfig` I've checked in here, you can delete it,
then go through and change the menu entries that you need.

I've listed the exact settings I have, you'll have to figure out what the defaults currently are in ESP-IDF.

- Support ESP32 Internal EMAC controller (true)
  - PHY interface (RMII)
  - RMII clock mode (Output RMII clock from internal)
  - Output RMII clock from GPI00 (DISABLED)
  - RMII clock GPIO numer (17)
  - Ethernet DMA buffer size bytes (1524)
  - Amount of Ethernet DMA Rx buffers (4)
  - Amount of Ethernet DMA Tx buffers (4)
- Support SPI to Ethernet module (DISABLED)
- Support OpenCores Ethernet MAC (DISABLED)

A few notes. 

The default in this section is RMII, but it's using an RMII Input clock, so that has to be changed. The GPIO number that is default also has to be changed.

There is a forum post from 2019 about packet corruption if the DMA buffer size is less than internet MTU. The default in that field is 512. I don't have evidence that bug still exists, but I've raised the number to 1524 anyway. According to the post, the smaller value works, until you're doing a large transfer (notably a OTA update) at which point the OTA update never succeeds. I've dropped the number of buffers a little to make up for it.

## sdkconfig for Example 

These entries are in the top level of Menuconfig.

- Ethernet Type (Internal EMAC)
- Ethernet PHY device (LAN8720)
- SMI MDC GPIO number (23)
- SMI MDIO GPIO number (18)
- PHY Reset GPIO number (-1)
- PHY Address (0)

If you reset the sdkconfig, some of these don't come up with the default.

## note about LAN8720

There are some forum posts about needing to have an entry for LAN8710a. I found these very confusing because there wasn't any additional statement about how one sets things to LAN8710a, but it seems LAN8720 works now.

## Note about serial ports

The Olimex ESP32-POE series uses a CH341 USB controller. This is not included with some devices (first time I've had an ESP32 with no windows driver preloaded), so I had to go find the driver.

Olimex's distribution https://www.olimex.com/Products/Breadboarding/BB-CH340T/resources/CH341SER.zip worked great for me. Unzip and run the executable inside. This included working with ESP-IDF under Windows WSL1 and using that for serial port programming using this driver.

## note about simplified example

There was a lot of code in the example about SPI. This is not an SPI connected device so I took out those lines for clarity.

# Example overview

The main difference post 4.1 is the addition of the netif concept. What you see in the example code is there are four configs one has to set up and knit together, and it's best if each has their own name, not just `config`, so you can keep everything straight.


## Example Output

```bash
I (394) eth_example: Ethernet Started
I (3934) eth_example: Ethernet Link Up
I (3934) eth_example: Ethernet HW Addr 30:ae:a4:c6:87:5b
I (5864) tcpip_adapter: eth ip: 192.168.2.151, mask: 255.255.255.0, gw: 192.168.2.2
I (5864) eth_example: Ethernet Got IP Address
I (5864) eth_example: ~~~~~~~~~~~
I (5864) eth_example: ETHIP:192.168.2.151
I (5874) eth_example: ETHMASK:255.255.255.0
I (5874) eth_example: ETHGW:192.168.2.2
I (5884) eth_example: ~~~~~~~~~~~
```

Now you can ping your ESP32 in the terminal by entering `ping 192.168.2.151` (it depends on the actual IP address you get).

# References


https://github.com/OLIMEX/ESP32-POE/issues/20 - request for 4.1 example

https://github.com/OLIMEX/ESP32-POE/issues/7 - example doesn't build against latest esp-idf master (2019)

https://github.com/espressif/esp-idf/issues/4454 - corrupt heap errors in Lan8710a

# License

I don't consider this actually licencable. I had to crib from Espressif's example and Olimex's example but neither was correct. It's possible the open source licenses from either one of those examples actually applies. In reality, both Espressif and Olimex want these cards to work and as they've both
published as open soruce these (non-working) examples, it seems unlikely anyone would have the grounds to sue anyone else regarding this repository.

