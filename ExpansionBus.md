# BP-80E Expansion Bus

CyBorg uses a custom Expansion Bus used for add-on cards called "BP-80 Enhanced" (aka. "BP-80E"). This bus specification is essentially a variation of the [Official RC2014 Enhanced Bus Specification](https://smallcomputercentral.com/documentation/specification-rc2014-bus/#enhanced) with the following changes:

1) Row 2 Pins 1 - 16 are used by the 16 GPIOS provided by the MCP23017 I/O expander chip located next to the first bus connector (these pins were orginally unused).
2) Row 1 and Row 2 pins 37 - 40 were originally specified for USER functions (which could be anything), which I've assigned to extra grounds, power pins with additional voltages (+3.3V and +12V) and interrupt cascade.

With the exception of the above pin assignment changes, all other pin definitions remain the same and as such, *should be* compatible with official [RC2014](https://rc2014.co.uk/) add-on cards and cards designated as being "Designed for RC2014" with the caveat of any card that specifically uses any of the above pins for other purposes would, of course, not be compatible and could therefore potentially cause damage to both this board and any board you plug into intended for use in RC2014-compatible systems. **USE WITH CAUTION**.

# LEGAL DISCLAIMER
I am not liable for any damage to any device you interface with this board, particularly if you ignore the above information. While I did adopt a *similar* expansion bus, they are NOT the same and should not be treated as such. This board should not be considered to be a RC2014-compatible system. They are fundamentally different! I only offer the above info because I have indeed tested at least one add-on board designated as being "Designed for RC2014" without issue, but you should excersize caution and if you don't know for sure, don't try it!

# Expansion Bus Pinout

The following table describes the pin definitions:

| Row  | Pin # | Signal | Description                       |
| :--- | :---  | :---   | :---                              |
| 1 (R)| 1     | A15    | Address bit 15                    |
| 1 (R)| 2     | A14    | Address bit 14                    |
| 1 (R)| 3     | A13    | Address bit 13                    |
| 1 (R)| 4     | A12    | Address bit 12                    |
| 1 (R)| 5     | A11    | Address bit 11                    |
| 1 (R)| 6     | A10    | Address bit 10                    |
| 1 (R)| 7     | A9     | Address bit 9                     |
| 1 (R)| 8     | A8     | Address bit 8                     |
| 1 (R)| 9     | A7     | Address bit 7                     |
| 1 (R)| 10    | A6     | Address bit 6                     |
| 1 (R)| 11    | A5     | Address bit 5                     |
| 1 (R)| 12    | A4     | Address bit 4                     |
| 1 (R)| 13    | A3     | Address bit 3                     |
| 1 (R)| 14    | A2     | Address bit 2                     |
| 1 (R)| 15    | A1     | Address bit 1                     |
| 1 (R)| 16    | A0     | Address bit 0                     |
| 1 (R)| 17    | GND    | Ground                            |
| 1 (R)| 18    | +5V    | +5V DC                            |
| 1 (R)| 19    | /M1    | Opcode fetch cycle                |
| 1 (R)| 20    | /RESET | Northbridge/CPU reset             |
| 1 (R)| 21    | CLK    | CPU Clock                         |
| 1 (R)| 22    | /INT   | Maskable Interrupt                |
| 1 (R)| 23    | /MREQ  | Memory Request                    |
| 1 (R)| 24    | /WR    | Write data                        |
| 1 (R)| 25    | /RD    | Read data                         |
| 1 (R)| 26    | /IORQ  | I/O Request                       |
| 1 (R)| 27    | D0     | Data bit 0                        |
| 1 (R)| 28    | D1     | Data bit 1                        |
| 1 (R)| 29    | D2     | Data bit 2                        |
| 1 (R)| 30    | D3     | Data bit 3                        |
| 1 (R)| 31    | D4     | Data bit 4                        |
| 1 (R)| 32    | D5     | Data bit 5                        |
| 1 (R)| 33    | D6     | Data bit 6                        |
| 1 (R)| 34    | D7     | Data bit 7                        |
| 1 (R)| 35    | TX     | Serial Transmit (UART)            |
| 1 (R)| 36    | RX     | Serial Recieve (UART)             |
| 1 (R)| 37    | SDA    | I2C Serial Data                   |
| 1 (R)| 38    | SCL    | I2C Serial Clock                  |
| 1 (R)| 39    | GND    | Ground                            |
| 1 (R)| 40    | IEO    | Interrupt cascade (out)           |
| 2 (L)| 41    | GPA7   | IOEX Port A bit 7*                |
| 2 (L)| 42    | GPA6   | IOEX Port A bit 6*                |
| 2 (L)| 43    | GPA5   | IOEX Port A bit 5*                |
| 2 (L)| 44    | GPA4   | IOEX Port A bit 4*                |
| 2 (L)| 45    | GPA3   | IOEX Port A bit 3*                |
| 2 (L)| 46    | GPA2   | IOEX Port A bit 2*                |
| 2 (L)| 47    | GPA1   | IOEX Port A bit 1*                |
| 2 (L)| 48    | GPA0   | IOEX Port A bit 0*                |
| 2 (L)| 49    | GPB0   | IOEX Port B bit 0*                |
| 2 (L)| 50    | GPB1   | IOEX Port B bit 1*                |
| 2 (L)| 51    | GPB2   | IOEX Port B bit 2*                |
| 2 (L)| 52    | GPB3   | IOEX Port B bit 3*                |
| 2 (L)| 53    | GPB4   | IOEX Port B bit 4*                |
| 2 (L)| 54    | GPB5   | IOEX Port B bit 5*                |
| 2 (L)| 55    | GPB6   | IOEX Port B bit 6*                |
| 2 (L)| 56    | GPB7   | IOEX Port B bit 7*                |
| 2 (L)| 57    | GND    | Ground                            |
| 2 (L)| 58    | +5V    | +5V DC                            |
| 2 (L)| 59    | /RFSH  | Refresh                           |
| 2 (L)| 60    | NC     | Not connected/unused              |
| 2 (L)| 61    | NC     | Not connected/unused              |
| 2 (L)| 62    | /BUSACK| Bus acknowledge                   |
| 2 (L)| 63    | /HALT  | Halt/pause execution              |
| 2 (L)| 64    | /BUSREQ| Bus request                       |
| 2 (L)| 65    | /WAIT  | Enter wait state                  |
| 2 (L)| 66    | /NMI   | Non-Maskable Interrupt            |
| 2 (L)| 67    | D8     | Data bit 8**                      |
| 2 (L)| 68    | D9     | Data bit 9**                      |
| 2 (L)| 69    | D10    | Data bit 10**                     |
| 2 (L)| 70    | D11    | Data bit 11**                     |
| 2 (L)| 71    | D12    | Data bit 12**                     |
| 2 (L)| 72    | D13    | Data bit 13**                     |
| 2 (L)| 73    | D14    | Data bit 14**                     |
| 2 (L)| 74    | D15    | Data bit 15**                     |
| 2 (L)| 75    | TX2    | Serial Transmit 2***              |
| 2 (L)| 76    | RX2    | Serial Receive 2***               |
| 2 (L)| 77    | 3V3    | +3.3V DC                          |
| 2 (L)| 78    | +12V   | +12V DC                           |
| 2 (L)| 79    | GND    | Ground                            |
| 2 (L)| 80    | IEI    | Interrupt cascade (in)            |

\* - Denotes signal level is 3.3V. All other signals are at a 5V level.

\** - Denotes data bus signal is present on expansion bus *ONLY* and does not connect to the CPU's data bus.

\*** - Denotes serial signals present on the expansion bus *ONLY* and does not connect to any UART present on the Northbridge or Southbridge.

\/ - Signal prefix denotes a signal that is active low.

See the [Z80 CPU User Manual](https://www.zilog.com/docs/z80/um0080.pdf) for documentation on the Z80-specific signals above.

# Notes
While CyBorg only has 3 expansion bus slots, it is possible to expand that by using a riser card plugged into one of the expansion connectors that provides more slots. This might be tricky to implement if you are using this board in an actual ATX chassis, but I will probably attempt to design a bus expansion board at some point in the future. I personally have used [Stephen Cousin's SC113 Modular Backplane](https://smallcomputercentral.com/sc113-modular-backplane-rc2014/) for this purpose in the interim. Again, you should not try this unless you know what you are doing! In hindsight, I should have probably just used another bus connector, but I wanted to be able to re-use some of my existing add-on cards designed for RC2014 with this system. I may change this in the future though.