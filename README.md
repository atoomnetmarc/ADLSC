# ADLSC, Atoomnet's Digital Led String Controller

ESP32 based 5V digital LED controller build for use with [WLED](https://kno.wled.ge/).

# BOM

See Kicad directory for PCB BOM.

To complete the PCB you will also need:

- 2x 5x20mm 3A slow blow glass fuse: T3AL250V

You will also need something like:

- A 5V power supply like the Mean Well 5V@10A power supply: IRM-60-5ST
- 5V digital LED strip or matrix.

If you choose to use a different or lower amperage 5V power supply, then you must also use lower power glass fuses. Reserve 1A for the low power components then divide by 3 for the fuse rating. For example (10A - 1A) / 3 = 3A. Also make sure the power supply can handle short circuits by switching off or go into hiccup mode.

# Design choices

The chosen ESP32 classic is the ESP32-WROVER-E with PSRAM on a [ESP32-DevKitC V4 board](https://docs.espressif.com/projects/esp-idf/en/release-v4.2/esp32/hw-reference/esp32/get-started-devkitc.html).

The ESP32 must be able to control the power to the LEDs. That means a high-side power switch.

The ESP32 will be powered by the same 5V power supply. A 0.5A 1812 polyfuse with 5V bidirectional TVS diode will used to protect the low power electronics.

5V will be monitored by a MAX809M reset ic. When the power supply voltage is too low the ESP32 will be turned off and in turn the LEDs will be turned off.

A N-channel MOSFET (AOD442) will be used as a 5V reverse polarity protection.

The power to the LEDs can be injected in 2 sections. Both are fused by a 5x20mm slow blow glass fuse paired with 5V bidirectional TVS diodes (like SMBJ5.0CA). Red LEDs will be parallel connected to easily identify a blown fuse.

# Firmware

Install `ESP32_WROVER.bin`. Configure current limit at 5A.

# ESP32 pins

| Name   | ESP32 pin | Usage                               |
| ------ | --------- | ----------------------------------- |
| DATA1  | IO25      | LED data                            |
| DATA2  | IO26      | LED clock or data for second string |
| OUT_ON | IO4       | LED power relay                     |
| SDA    | IO21      | I2C data                            |
| SCL    | IO22      | I2C clock                           |
| CS0    | IO4       | SPI chip select 0                   |
| CLK    | IO18      | SPI clock                           |
| MISO   | IO19      | SPI data in                         |
| MOSI   | IO23      | SPI data out                        |

Note: the 3V3 on the I2C and SPI PMOD connectors come from the ESP32 3V3 regulator, keep current usage below 100mA.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
