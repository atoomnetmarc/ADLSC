# ADLSC, Atoomnet's Digital Led String Controller

ESP32 based Neopixel controller build for use with WLED.

# Design choices

I want to make use of the [WLED project](https://kno.wled.ge/), so an ESP32 classic seems a good choice.

The chosen ESP32 classic is the ESP32-WROVER-E with PSRAM on a [ESP32-DevKitC V4 board](https://docs.espressif.com/projects/esp-idf/en/release-v4.2/esp32/hw-reference/esp32/get-started-devkitc.html).

The ESP32 must be able to control the power to the Neopixels. That means a high-side power switch capable of carrying the current of 640 Neopixels that are all 100% on. A current estimate made with [wled-calculator](https://wled-calculator.github.io/) results in a power consumption of 131.4W or 26.3A.

The ESP32 will be powered by the same 5V power supply. A 0.5A 1812 polyfuse with 5V bidirectional TVS diode will used to protect the low power electronics.

5V will be monitored by a MAX809M reset ic. When the power supply voltage is too low the ESP32 will be turned off and in turn the Neopixels will be turned off.

A N-channel MOSFET (AOD442) will be used as a 5V reverse polarity protection.

The power to the Neopixels will split in 2 sections and both be fused by a 5x20mm 3A slow blow glass fuse paired with 5V bidirectional TVS diodes (like SMBJ5.0CA). Red LEDs will be parallel connected to easily identify a blown fuse.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)