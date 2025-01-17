# ADLSC, Atoomnet's Digital Led String Controller

ESP32 based 5V digital LED controller build for use with [WLED](https://kno.wled.ge/).

![](ADLSC.jpg)

# Design choices

The chosen ESP32 classic is the ESP32-WROVER-E with PSRAM on a [ESP32-DevKitC V4 board](https://docs.espressif.com/projects/esp-idf/en/release-v4.2/esp32/hw-reference/esp32/get-started-devkitc.html).

The ESP32 must be able to control the power to the LEDs. That means a high-side power switch.

The ESP32 will be powered by the same 5V power supply. A 0.5A 1812 polyfuse with 5V bidirectional TVS diode will used to protect the low power electronics.

5V will be monitored by a MAX809M reset ic. When the power supply voltage is too low the ESP32 will be turned off and in turn the LEDs will be turned off.

A N-channel MOSFET (AOD442) will be used as a 5V reverse polarity protection.

The power to the LEDs can be injected in 2 sections. Both are fused by a 5x20mm slow blow glass fuse paired with 5V bidirectional TVS diodes (like SMBJ5.0CA). Red LEDs will be parallel connected to easily identify a blown fuse.

# BOM

See Kicad directory for PCB BOM.

PCB BOM Notes:

- Instead of [Weidmueller 1824810000 clamp terminals](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1824810000) you could also use [Weidmueller 1760520000](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1760520000) or [1760510000](https://catalog.weidmueller.com/catalog/Start.do?localeId=en_DE&ObjectID=1760510000) or generic 5.08mm pcb screw terminals.
- I made the fuse clip footprint to fit https://www.aliexpress.com/item/4000841928644.html, but other could also fit.

To complete the PCB you will also need:

- 2x 19pin 2.54mm pin header (possibly already included with the ESP32 board)
- 2x 19pin 2.54mm pin socket
- 1cm<sup>2</sup> of kapton tape

Then you will also need:

- Mean Well 5V 10A power supply: IRM-60-5ST
- 5V digital LED strip or matrix

# PCB assembly

Open the [Kicad interactive BOM](Kicad/bom/ibom.html). Use it to mark which component you have placed and soldered.

Recommended soldering order:

- Q2, Q4
- U2
- Q3
- U1
- D2, D6, D9, D10, D12
- D1, D3, D8, D11
- D4, D5, D7, D13
- D14
- Q1
- F1
- R5, R12, R16, R20
- R1, R3, R11, R22
- R21
- R2, R8, R10, R14, R15, R18, R19, R23
- C1, C3
- C2

Clean the top of the PCB.

Continue soldering:

- J1, J3, J4, J6
- A1, use the ESP32 module to align 2x19 pin socket.

Remove the ESP32 module.

Put a fuse in 2 fuse clips. Do this 2 times.\
Cover the bottom of the fuse clips with some kapton tape. This mechanically insulates the clips from the PCB.

Solder the fuse clips (with the fuses in it) into F2 and F3.

Remove fuses. Clean the bottom of the PCB.

# Testing

Set power supply max current to 0.1A. Connect power supply to J1. Slowly increase to 5V.\
You should notice less than 10mA current. D2 turns on.

Reverse the polarity of the power supply.\
D1 turns bright red. Current is about 10mA.

Restore polarity. Increase max current to 2A. Short C2, then turn on power supply.\
Polyfuse F1 will blow, supply current should significantly lower (to about 250mA), D3 will turn bright red.

# Firmware

While the ESP32 modules is not mounted to the PCB, [install `ESP32_WROVER.bin`](https://kno.wled.ge/basics/install-binary/).

Configure:
- WiFi
  - Network Name
  - A unique mDNS address
  - AP SSID and password
  - Disable WiFi sleep
- LED
  - Current limit to 5A
  - Output to the matrix you have
    - For a 80x8 matrix: WS281x, 55mA, GRB, 640 length
    - Data GPIO: 25
    - Relay GPIO: 4, invert.
    - Default brightness: 128
    - Brightness factor: 25%
- 2D configuration
   - Number of panels: 1
   - Panel 0
     - 1<sup>st</sup> LED: Top Left
     - Serpentine
     - Dimensions: 80x8

# More testing

Connect ESP32 module to PCB. Set power supply current to 0.5A, 5V.\
Turn on power supply. Now observe:
- Current is about 150mA
- D2 green is on
- D6 green is on
- D9 green is on
- D12 green is on
- D8 red is on
- D11 red is on

Press and hold `EN` button of ESP32 module. All LEDs should turn off except D2 (and the onboard red led of the ESP32 module). Release `EN`.

Put fuses in F2 and F3. D8 and D11 will turn off, D6 and D12 will get brighter.\
Wait, did you turn off power supply while you put in the fuses?

Now really turn off power supply. Increase current to 5A.

Connect (part of) the LED matrix and start playing with WLED.

# Wiring

## High voltage

Consult an expert on how to wire mains voltage to the Mean Well 5V power supply.

## Low voltage

Suggestion: use black wire color for GND. Grey for 0V. Red for 5V. Blue for data.

When consulting https://en.wikipedia.org/wiki/American_wire_gauge you should:

- use AWG18 (0.75mm<sup>2</sup>) or thicker wiring to connect the 5V dc from the power supply using a red and grey wire to the 5V of the power supply
- use AWG20 (0.5mm<sup>2</sup>) or thicker wiring to connect 5V power to LED strip or matrix.

You may use any wire thickness (challenge accepted) to connect data1 (and data2) to the LED strip. Or use the same AWG20.

## Fuse rating for power supply

If you choose to use a different or lower amperage 5V power supply, then you must also use lower power glass fuses. Reserve 1A for the low power components (like the ESP32) then divide by 3 for the fuse rating. For a 10A power supply: (10A - 1A) / 3 = 3A fuse. Also make sure the power supply can handle short circuits by switching off or go into hiccup mode.

# ESP32 pins

| Name     | ESP32 pin | Usage                               |
| -------- | --------- | ----------------------------------- |
| DATA1    | IO25      | LED data                            |
| DATA2    | IO26      | LED clock or data for second string |
| POWER_ON | IO4       | LED power relay                     |
| SDA      | IO21      | I2C data                            |
| SCL      | IO22      | I2C clock                           |
| CS0      | IO4       | SPI chip select 0                   |
| CLK      | IO18      | SPI clock                           |
| MISO     | IO19      | SPI data in                         |
| MOSI     | IO23      | SPI data out                        |

# Expansion PMOD connectors

You could add you own custom functionality using the I2C and SPI PMOD connectors. Consult the schematic on what extra components you should solder to the PCB. These components are excluded from the BOM.

Note: the 3V3 on the I2C and SPI PMOD connectors come from the ESP32 3V3 regulator, keep current usage below 100mA.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
