# MC68HC05 8-bit Microcontroller (IKAT)

In CD-i players the IKAT microcontroller performs CD drive control,
CD-i pointing device, front LED display and general system control functions. It
contains XXX bytes of read-only factory-programmed program ROM and XXX bytes of
RAM.

The data sheet of the base MC68HC05 chip is publicly available but interface
documentation for the IKAT program is not. This document is an attempt to
describe the IKAT functions with enough level of detail so that they can be
emulated in software.

The information in this document has mostly been determined by reverse
engineering CD-i drivers for the CD-i Mono-II and up mainboard generations that
use the IKAT microcontroller.

Because of this, the exact behaviour of many functions is unclear. In some
of these cases the emulation used by [CD-i Emulator] is described.

## Memory map

The following memory map has been derived from the service manuals and reverse
engineering.

All memory addresses are hexadecimal and relative to the base address of the
IKAT. In CD-i players this is always 00310000 so that for example Port A is located at main CPU address 0020001.

The IKAT has an 8-bit connection to the lower part of the main 68000 CPU data
bus and does not have an A0 input pin, so all of the memory addresses below are
odd and the registers must be accessed with byte read or write cycles.

Address | Register | Description
- | - | -
01 | PAWR | Port A write register
03 | PBWR | Port B write register
05 | PCWR | Port C write register
07 | PDWR | Port D write register
09 | PARD | Port A read register
0B | PBRD | Port B read register
0D | PCRD | Port C read register
0F | PDRD | Port D read register
11 | PAWR | Port A status register
13 | PBWR | Port B status register
15 | PCWR | Port C status register
17 | PDWR | Port D status register
19 | ISR | Interrupt status register
1B | ICR | Interrupt control register
1D | YCR | Y control register [sic]

## Concept of operation

Each port accepts a number of command messages and can return a number of
response messages. Some commands will always return response messages, others
will not; response messages can also be returned aynchronously driven by
external input, e.g. from pointing devices.

The length of each message is determined by its first byte. Both commands and
responses are port-specific, e.g. first byte 80 on port B specifies a different
command than first byte 80 on port D.

The IKAT will assert its level-based interrupt output whenever the interrupt
bits for any port are set in bith the ISR and ICR registers.

## Register description

#### P*x*WR - Port *x* write register (*x* = A/B/C/D)

This register is used for writing Port *x* command bytes.

#### P*x*RD - Port *x* read register (*x* = A/B/C/D)

This register is used for reading Port *x* response bytes.

#### P*x*ST - Port *x* status register (*x* = A/B/C/D)

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="PxST[7]">?</td>
    <td title="PxST[6]">?</td>
    <td title="PxST[5]">?</td>
    <td title="PxST[4]">RDIDLE</td>
    <td title="PxST[3]">?</td>
    <td title="PxST[2]">?</td>
    <td title="PxST[1]">?</td>
    <td title="PxST[0]">WRIDLE</td>
</tr>
</table>

This register is used for reading Port *x* status.

The WRIDLE bit is set if command bytes can be written to the
Port *x* write register P*x*WR.

The RDIDLE bit is set if there are no response bytes to be read from the
Port *x* read register P*x*RD.

[CD-i Emulator] always returns the WRIDLE bit set as command processing
does not take emulated time; this might be different on real hardware.

#### ISR - Interrupt status register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ISR[7]">INTD</td>
    <td title="ISR[6]">?</td>
    <td title="ISR[5]">INTC</td>
    <td title="ISR[4]">?</td>
    <td title="ISR[3]">INTB</td>
    <td title="ISR[2]">?</td>
    <td title="ISR[1]">INTA</td>
    <td title="ISR[0]">?</td>
</tr>
</table>

The bits of this register indicate conditions that can generate interrupts
to the main 68000 CPU. If the corresponding bit in ICR is set, an interrupt
will be generated whenever a bit of this register becomes set.

Each of the INT*X* bits becomes set whenever response message
bytes are available on the corresponding Port *X* read register.

#### ICR - Interrupt control register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ICR[7]">INTD</td>
    <td title="ICR[6]">?</td>
    <td title="ICR[5]">INTC</td>
    <td title="ICR[4]">?</td>
    <td title="ICR[3]">INTB</td>
    <td title="ICR[2]">?</td>
    <td title="ICR[1]">INTA</td>
    <td title="ICR[0]">?</td>
</tr>
</table>

The bits of this register specify the conditions for generating interrupts to
the main 68000 CPU. If a bit in this register is set, an interrupt will be
generated whenever the corresponding bit in ISR becomes set. See the description
of ISR for a description of the conditions associated with each bit.

### YCR - Y control register [sic]

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="YCR[7]">?</td>
    <td title="YCR[6]">?</td>
    <td title="YCR[5]">?</td>
    <td title="YCR[4]">?</td>
    <td title="YCR[3]">?</td>
    <td title="YCR[2]">?</td>
    <td title="YCR[1]">?</td>
    <td title="YCR[0]">?</td>
</tr>
</table>

The function of this register is not known.

NB. It is named the Y control register because ICTL was initially named
the X control register before its purpose was known.

## Commands

### Port A

TBA: port A commands

### Port B

TBA: port B commands

### Port C

#### Reset main cpu
Port | Hex | Binary | Name
- | - | - | -
PC | 88 | 1000 1000 | `Reset main cpu`

Resets the main 68000 CPU by asserting its RESET input.

**Response:** None, main CPU is reset

#### Get boot mode
Port | Hex | Binary | Name
- | - | - | -
PC | F4 | 1111 0100 | `Get boot mode`

**Response:** `Boot mode` on Port C

#### Get video standard
Port | Hex | Binary | Name
- | - | - | -
PC | F6 | 1111 0110 | `Get video standard`

**Response:** `Video standard` on Port C

TBA: more port C commands

### Port D

TBA: port D commands

## Responses

### Port A

TBA: port A responses

### Port B

#### Absolute pointer state
Port | Hex | Binary | Name
- | - | - | -
PB | 4x xx xx xx ... 7x xx xx xx | 01*ab cccc* 000*d eeee* 00*ff ffff* 10*gg gggg* | `Absolute pointer state`

Reports the current pointer state in absolute coordinates: \
*a* = set when button 2 pressed \
*b* = set when button 1 pressed \
*d* = set when pointer active \
*ccccffffff* = x position \
*eeeegggggg* = y position

[CD-i Emulator] uses this response message only when necessary;
the `Relative pointer state` response is used whenever both x and y
position delta values fit in an 8-bit signed value (i.e. are in the -128 ...
+127 range).

#### Relative pointer state
Port | Hex | Binary | Name
- | - | - | -
B | 4x xx xx xx ... 7x xx xx xx | 01*ab ccdd* 00*ee eeee* 00*ff ffff* 0000 0000 | `Relative pointer state`

Reports the current pointer state in relative coordinates: \
*a* = set when button 2 pressed \
*b* = set when button 1 pressed \
*cceeeeee* = x position delta (8-bit signed value)\
*ffgggggg* = y position delta (8-bit signed value)

[CD-i Emulator] uses this response message whenever possible;
the `Absolute pointer state` response is only used when one of the x and y
position delta values does not fit in an 8-bit signed value (i.e. is not in the -128 ...
+127 range).

TBA: more port B responses

### Port C

#### Boot mode
Port | Hex | Binary | Name
- | - | - | -
PC | A5 F4 0m | 1010 0101 1111 0100 0000 *mmmm* | `Boot mode`

Reports the boot mode as determined by service plug detection.

*m* | Mode
- | -
0 | Boot player shell
1 | Boot service shell

#### Video standard
Port | Hex | Binary | Name
- | - | - | -
PC | A5 F6 0m ?? | 1010 0101 1111 0110 0000 *mmmm* ???? ???? | `Video standard`

Reports the current video standard as taken from IKAT input pin PC7.

*m* | Standard
- | -
1 | NTSC / 60 Hz
2 | PAL / 50 Hz

[CD-i Emulator] always returns FF for the ?? byte.

TBA: more port C responses

### Port D

TBA: port D responses

[CD-i Emulator]: http://www.cdiemu.org/cdiemu/
