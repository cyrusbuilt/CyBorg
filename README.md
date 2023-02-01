# CyBorg
ATX-compliant Z80-based SBC motherboard

![CyBorg](top.svg)

## Synopsis
Being a huge fan of the retro-computing scene, and having built a few different "modern retro" computers from kits that were Z80 and Z180-based, I decided to study their schematics and try to design one myself. The design I settled on is a kind of amalgamation of the [Z80-MBC2](https://github.com/SuperFabius/Z80-MBC2) and my [CyrUX Board](https://github.com/cyrusbuilt/CyrUX) and a variation of the [RC2014 Enhanced Bus](https://smallcomputercentral.com/documentation/specification-rc2014-bus/#enhanced). CyBorg represents my first attempt at a CP/M and [Fuzix](https://github.com/EtchedPixels/FUZIX)-compatible retro computer that you can mount in a standard PC chassis.

## Features
- VGA Display (1280x768 B&W, 1024x720 4 color, 800x600 8 color, 720x520 16 color, 640x480 16 color @ 73Hz, 640x480 16 color @ 60Hz, 640x350 16 color, 512x384 64 color, and 400x300 64 color)
- PS/2 Keyboard and Mouse
- Serial Console
- Amplified Mono Audio
- Battery-backed Hardware Real-Time Clock
- Z80 CPU @ 8MHz
- 128K SRAM
- ATX 2.01-compliant
- Header for microSD card reader
- Front Panel Headers
- BP-80E Expansion Slots x3
- iLoad Intel-HEX loader firmware embedded.
- Optionally boots CP/M 2.2, CP/M 3, QP/M 2.71, Forth, BASIC, and Fuzix.

## Feature Break-Down
The features on this board are implemented using what I call the "Northbridge" and the "SouthBridge". Both chips are responsible for different areas of the board. The Southbridge operates identically to my CyrUX board, but with ATX power management and boot sequence control added to it. The following is a breakdown of what each is and does:

## Northbridge
1) Atmel ATMEGA32A-PU
2) Specs:
	* 8-bit MCU @ 16MHz
	* 40-pin DIP package
	* 32KB Flash
	* 8KB EEPROM
	* 16KB RAM
3) Responsibilities:
	* Choreograph data between CPU, RAM, and data bus.
	* Control and provide interface for RTC (DS1307).
	* Control and provide interface for I/O expansion (via MCP23017).
	* Control onboard buzzer/speaker.
	* Communicate with Southbridge.
	* Expose OpCodes for system programming.
	* Provides system clock (4/8MHz).
	* Provides system ROM and bootstrapping.
	* RS-232 UART.

## Southbridge
1) [Adafruit Huzzah ESP32](https://www.adafruit.com/product/4172)
2) Specs:
	* 32bit dual-core Tensilica LX6 MCU @ 240MHz.
	* 520KB SRAM.
	* 802.11b/g/n HT40 Wi-Fi.
	* Dual-mode Bluetooth (classic and BLE).
	* 4MB Flash
	* PCB antenna
3) Responsibilities:
	* ATX Power management.
	* Boot sequence control.
	* [FabGL-based](https://github.com/fdivitto/FabGL) Terminal System which provides:
	* PS/2 Keyboard and Mouse input.
	* Amplified Mono Audio Output (LM386).
	* VGA Display Output.
	* Possible future networking support.

## Core
1) Zilog Z80 CMOS CPU @ 8MHz
	* Main System Processor.
2) AS6C1008-55PCN 128KB SRAM (banked 64K x 2)
	* Main System RAM.

## Expansion Bus Slots
The expansion bus slots are an unofficial variation of the RC2014 Enhanced bus I call "BP-80E" or "BP-80 Enhanced" which is kind of a misnomer given the actual pinout of the unofficial BP-80 bus definition, but seemed appropriate for my purposes (at least until I come up with a better name for it). It is pin-compatible with the RC2014 Standard and Enhanced bus slots and therefore should be compatible with existing add-on boards, but adds the following changes:
1) Pins 41 - 56 map to the I/O pins on the MCP23017 I/O expander and therefore gives us 16 shared GPIO pins (3.3V).
2) Pins 39 and 79 are additional GND.
3) Pin 78 is +12V.
4) Pin 77 is +3.3V.
5) Pins 37 and 38 are I2C SDA & SCL (respectively).

## Jumpers and Headers
1) Header H4: RS-232 Serial from the Northbridge/Main system. This is also tied to the DB-9 Serial port next to the VGA connector.
2) Header H3: ICSP header used to program/debug the Northbridge.
3) Header H2: This is essentially an SPI bus header used to attach the SD Card Reader.
4) Header H1: RS-232 serial header that goes directly to the Southbridge and used to program/debug the Southbridge. Intended to be used with a USB-to-Serial FTDI cable.
5) Jumper JP1: (should be jumpered for normal operation) enables power to the Southbridge via the 5VSB line from the ATX power supply. Unjumper this to enable power from the FTDI cable.
6) Jumpers JP2 and JP3: (should be jumpered for normal operation) enables the Serial connection to the Southbridge. Unjumper these to use just H1 or DB-9 serial port.
7) JP5: Front-panel reset switch. Momentarily short to reset the system. Same as pressing RESET button.
8) JP6: Front-panel power button. Momentarily short to power system on. Short for >= 10s to power off. Same as pressing ON/OFF button.
9) JP7: Front-panel USER button. Press and hold during power on to get boot mode selection menu. Same as pressing USER button.
10) JP8: Front-panel power LED header. Same as PWR OK LED.
11) JP9: Front-panel SD card activity LED. Same as SD ACT LED.
12) JP10: Front-panel boot mode LED. Same as USER LED.
13) JP11: Enable Serial RX line from Southbridge to Northbridge. Should be enabled for normal operation. Disable (remove jumper) when flashing firmware. This may not be strictly necessary, but I was running into odd behavior with the ESP32 while flashing and disconnecting the RX line from the rest of the system seemed to alleviate the issue. So a simple jumper seemed to be the simplest fix until I have a more concrete solution.

## Boot Sequence
Being an ATX-compliant board, this system uses an ATX-20 power connector intended to be used with an ATX Power Supply. The ATX power management and boot sequence works as follows: The Southbridge receives it's power from the 5VSB line, which is always on as long as the ATX power supply has power (but not necessarily "running"). On power-on the Southbridge enters boot stage 0, which *only* initializes the debug port and the ATX power management system, then waits for the user to press the power button.

When the users presses power, it then drives the ATX PC_ON line LOW to turn on the ATX supply and waits for the PWR_OK line to go HIGH (as indicated by the PWR OK LED). When the PWR_OK signal is received, the Southbridge enters boot stage 1, which initializes the VGA display and PS/2 keyboard, followed by boot stage 2 which initializes the PS/2 mouse and audio. At the same time the Northbridge should have received power via the +5V line and entered boot stage 0, which initializes the RS-232 serial (and debug) interface and waits for the RUN signal line to go HIGH. The last step of the Southbridge's boot stage 2 should drive the RUN_3V3 line high and wait (5s timeout) for the Northbridge to complete bootstage 1 and start receiving serial data. When the Northbridge sees the RUN signal it should enter boot stage 1 which should first announce it's firmware version to the serial port then configure the WAIT and WAIT_RES lines then check the USER button for boot mode selection. Once the Southbridge sees serial data, it has completed it's own boot sequence and will continuously poll the power button for presses. If the user presses the power button for >= 10s, then the Southbridge will force-power down by driving the PC_ON line HIGH which will immediately drop out power from the power supply forcing the rest of the system off. The Southbridge will then do a soft-reboot to re-init back to boot stage 0.

Back to the Northbridge, after checking for boot mode selection and storing the flag, it enters boot stage 2 which initializes all the appropriate address and data lines on the system bus, then the logical RAM banks, system clock, and console lines, then moves to boot stage 3. Stage 3 reads the CPU speed mode, boot selection, and CP/M auto-exec flags from EEPROM, initializes the I2C interface and scans for the I/O expander and RTC, then outputs boot info to serial, then moves to boot stage 4. Stage 4 tries to mount the SD card (if present) and presents the user with boot mode selection menu if the USER line was LOW at boot and then proceeds to load the selected (via menu or from EEPROM or fallback to default) boot image into RAM (either from flash or from SD card), then moves to boot stage 5. Stage 5 holds the CPU in RESET, then initializes the system clock, then flushes the serial buffer, then releases the RESET line so the CPU begins executing.

Additional detail can be found in the firware projects for the [Northbridge](https://github.com/cyrusbuilt/CyBorg-Northbridge) and [Southbridge](https://github.com/cyrusbuilt/CyBorg-Southbridge).

## Project Structure
All design files are in the "schematics" folder. It contains the JSON exports for the [EasyEDA](https://easyeda.com/) schematic and PCB CAD files, the BOM in CSV format, PDFs of the schematic and PCB designs, and then in schematics/gerber are the production gerber files.

## Possible Revision Changes
1) Add fan headers
2) Switch to ATX24 power connector (far more common)