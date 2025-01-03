# CyBorg
ATX-compliant Z80-based SBC motherboard

![CyBorg](top.png)

## DISCLAIMER
This board is very much in the alpa stages and subject to design changes. This is my first SBC design and I'm learning as I go. As such this board is certainly not intended for production purposes.

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
The features on this board are implemented using what I call "ViCREM" (formerly "Northbridge") and "KAMVA" (formerly "SouthBridge"). Both chips are responsible for different areas of the board. The Southbridge operates identically to my CyrUX board, but with ATX power management and boot sequence control added to it. The following is a breakdown of what each is and does:

## ViCREM (formerly 'Northbridge')
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

## KAMVA (formerly 'Southbridge')
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
6) Pin 60 is Card Presence indicator
7) Pin 61 is Card Enable
8) /IORQ, /WR, and /RD signals are all unique to each slot.

## Jumpers and Headers
1) Header H4: RS-232 Serial from the Northbridge/Main system. This is also tied to the DB-9 Serial port next to the VGA connector.
2) Header H3: ICSP header used to program/debug the Northbridge.
3) Header H2: This is essentially an SPI bus header used to attach the SD Card Reader.
4) Header H1: RS-232 serial header that goes directly to the Southbridge and used to program/debug the Southbridge. Intended to be used with a USB-to-Serial FTDI cable.
5) Jumper JP1: (should be jumpered for normal operation) enables power to the Southbridge via the 5VSB line from the ATX power supply. Unjumper this to enable power from the FTDI cable.
6) Jumpers JP3 and JP4: (should be jumpered for normal operation) enables the Serial connection to the Southbridge.
7) JP5: Front-panel reset switch. Momentarily short to reset the system. Same as pressing RESET button.
8) JP6: Front-panel power button. Momentarily short to power system on. Short for >= 10s to power off. Same as pressing ON/OFF button.
9) JP7: Front-panel USER button. Press and hold during power on to get boot mode selection menu. Same as pressing USER button.
10) JP8: Front-panel power LED header. Same as PWR OK LED.
11) JP9: Front-panel SD card activity LED. Same as SD ACT LED.
12) JP10: Front-panel boot mode LED. Same as USER LED.

## Boot Sequence
Being an ATX-compliant board, this system uses an ATX-24 power connector intended to be used with an ATX Power Supply. The ATX power management and boot sequence works as follows: KAMVA receives it's power from the 5VSB line, which is always on as long as the ATX power supply has power (but not necessarily "running"). On power-on KAMVA enters boot stage 0, which *only* initializes the debug port and the ATX power management system, then waits for the user to press the power button.

When the user presses and releases the power button, it then drives the ATX PC_ON line LOW to turn on the ATX supply and waits for the PWR_OK line to go HIGH (as indicated by the PWR OK LED). When the PWR_OK signal is received, the Southbridge enters boot stage 1, which initializes the VGA display, PS/2 keyboard and mouse, and audio, followed by boot stage 2 which signals ViCREM to boot. At the same time ViCREM should have received power via the +5V line and entered boot stage 0, which initializes the RS-232 serial (and debug) interface and waits for the RUN signal line to go HIGH. The last step of KAMVA's boot stage 2 should drive the RUN_3V3 line high and wait (5s timeout) for ViCREM to complete bootstage 1 and start receiving serial data. When ViCREM sees the RUN signal it should enter boot stage 1 which should first announce it's firmware version to the serial port then configure the WAIT and WAIT_RES lines then check the USER button for boot mode selection. Once KAMVA sees serial data, it has completed it's own boot sequence and will continuously poll the power button for presses. If the user presses and holds the power button for >= 10s, then KAMVA will force-power down by driving the PC_ON line HIGH which will immediately drop out power from the power supply forcing the rest of the system off. KAMVA will then do a soft-reboot to re-init back to boot stage 0.

Back to ViCREM, after checking for boot mode selection and storing the flag, it enters boot stage 2 which initializes all the appropriate address and data lines on the system bus, then the logical RAM banks, system clock, and console lines, then moves to boot stage 3. Stage 3 reads the CPU speed mode, boot selection, and CP/M auto-exec flags from EEPROM, initializes the I2C interface and scans for the I/O expander, RTC, BUSCTLR and IOEXP chips, and CyBorgSPP interface then outputs boot info to serial, then moves to boot stage 4. Stage 4 tries to mount the SD card (if present) and presents the user with boot mode selection menu if the USER line was LOW at boot and then proceeds to load the selected (via menu or from EEPROM or fallback to default) boot image into RAM (either from flash or from SD card), then moves to boot stage 5. Stage 5 holds the CPU in RESET, then initializes the system clock, then flushes the serial buffer, then releases the RESET line so the CPU begins executing.

Additional detail can be found in the firware projects for [ViCREM](https://github.com/cyrusbuilt/CyBorg-Northbridge) and [KAMVA](https://github.com/cyrusbuilt/CyBorg-Southbridge).

## Project Structure
All design files are in the "schematics" folder. It contains the JSON exports for the [EasyEDA](https://easyeda.com/) schematic and PCB CAD files, the BOM in CSV format, PDFs of the schematic and PCB designs, and then in schematics/gerber are the production gerber files.

## I want one!
I have no plans to make CyBorg into a commercial product nor do I intend to sell kits. So if you want to build one, you'll need to take the gerber files and have the PCB made, then order the parts listed in the BOM and assemble it yourself. You'll need to be familiar with soldering through-hole components (I recommend an adjustable temperature soldering iron) and at least a multimeter (I often use a logic probe too)) and some wire snips to cut the legs on things like resistors and capacitors to size after you solder them. **However**, at this time, I would hold off on building one until I've finalized the design.

## Cool! I built one! Now what?
You'll need an ATX power supply (you can even use a Pico PSU). The next thing you'll want to do is flash the KAMVA (formerly known as the "Southbridge") firmware. Go see the project for that [here](https://github.com/cyrusbuilt/CyBorg-Southbridge). After you've done that, you'll need to flash the ViCREM (formerly known as "Northbridge") firmware. Go see the project for that [here](https://github.com/cyrusbuilt/CyBorg-Northbridge).

## Possible Revision Changes
1) Add fan headers
2) Change expansion bus slots to different connector type.

## Sponsorship/Special Thanks
A big special thanks to [PCBWay](https://www.pcbway.com) for their sponsorship on this project! It can't be stated enough the quality of their boards and their service. I've been using PCBWay to produce my boards for years now and I've always been really impressed. Fast shipping, fast service, reasonable cost, and the quality has always been stellar. Their pre-production team has even caught mistakes I didn't notice before sending my designs in. The ordering process is simple: just upload a zip file of your gerbers (their site even renders a preview of what the board will look like!) and then you can tweak parameters if needed (including color) and quantity, pick your shipping method, and that's it! They've even produced a series of videos about the whole process you can view [here](https://www.pcbway.com/pcb-service.html?step=1#miao1).

Just look at this beauty! (I'm a huge fan of their black PCBs)
![CyBorg_Board](board.jpg)

## Credits
- CyBorg's core is derived from the works of "Just4Fun" a.k.a "SuperFabius" ([https://github.com/SuperFabius](https://github.com/SuperFabius)) (parts of the hardware design and IOS firmware code).
- Southbridge firmware implements the FabGL library and is essentially a modified version of the AnsiTerminal example by Fabrizio Di Vittorio ([http://www.fabglib.org/index.html](http://www.fabglib.org/index.html))
- Northbridge firmware implements the PetitFS library by ChaN ([http://elm-chan.org/fsw/ff/00index_p.html](http://elm-chan.org/fsw/ff/00index_p.html))
- Southbridge firmware implements the ButtonEvent library by Renato Ferriera ([https://github.com/renatoferreirarenatoferreira/ebl-arduino/tree/master/ButtonEvent](https://github.com/renatoferreirarenatoferreira/ebl-arduino/tree/master/ButtonEvent))
- ViCREM (Northbridge) firmware uses [MightyCore](https://github.com/MCUdude/MightyCore) bootloader by [MCUDude](https://github.com/MCUdude)