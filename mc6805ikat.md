# MC68HC05i8 8-bit Microcontroller (IKAT)

In CD-i players the IKAT microcontroller performs CD drive control,
CD-i pointing device, front LED display and general system control functions. It
contains 7936 bytes of read-only factory-programmed user ROM and 224 bytes of
RAM.

The data sheet of the base MC68HC05i8 chip is publicly available but interface
documentation for the IKAT program is not. This document is an attempt to
describe the IKAT functions with enough level of detail so that they can be
emulated in software.

The information in this document has mostly been determined by reverse
engineering CD-i drivers for the CD-i [Mono-III] and [Mono-IV] mainboard generations that
use the IKAT microcontroller.

Because of this, the exact behaviour of many functions is unclear. In some
of these cases the emulation used by [CD-i Emulator] is described.

## Memory map

The following memory map has been derived from the service manuals and reverse
engineering.

All memory addresses are hexadecimal and relative to the base address of the
IKAT. In CD-i players this is always 00310000 so that for example the Channel A
write registers is located at main CPU address 00310001.

The IKAT has an 8-bit connection to the lower part of the main 68000 CPU data
bus and does not have an A0 input pin, so all of the memory addresses below are
odd and the registers must be accessed with byte read or write cycles.

Address | Register | Description
--- | --- | ---
01 | ADRW | Channel A Data Write Register
03 | BDRW | Channel B Data Write Register
05 | CDRW | Channel C Data Write Register
07 | DDRW | Channel D Data Write Register
09 | ADRR | Channel A Data Read Register
0B | BDRR | Channel B Data Read Register
0D | CDRR | Channel C Data Read Register
0F | DDRR | Channel D Data Read Register
11 | ASR | Channel A Status Register
13 | BSR | Channel B Status Register
15 | CSR | Channel C Status Register
17 | DSR | Channel D Status Register
19 | ISR | Interrupt Status Register
1B | IMR | Interrupt Mask Register
1D | MR | Mode Register

These register names are taken from the MC68HC05i8 Advance Information document.

Note: Channels A-D have no relation to MC68HC05i8 ports PA-PD.

## Concept of operation

Each channel accepts a number of command messages and can return a number of
response messages. Some commands will always return response messages, others
will not; response messages can also be returned aynchronously driven by
external input, e.g. from pointing devices.

The length of each message is determined by its first byte. Both commands and
responses are channel-specific, e.g. first byte 80 on channel B specifies a different
command than first byte 80 on channel D.

The IKAT will assert its level-based interrupt output whenever the interrupt
bits for any channel are set in both the ISR and IMR registers.

Some IKAT channels are read by interrupt handlers from multiple CD-i drivers; if this is the
case response messages may start with one or more A5 bytes which will select the correct
interrupt handlers.

## Register description

#### *x*DRW - Channel *x* Data Write Register (*x* = A/B/C/D)

This register is used for writing Channel *x* command bytes.

#### *x*DRR - Channel *x* Data Read Register (*x* = A/B/C/D)

This register is used for reading Channel *x* response bytes.

#### *x*SR - Channel *x* Status Register (*x* = A/B/C/D)

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="xSR[7]">ROR</td>
    <td title="xSR[6]">RRDY</td>
    <td title="xSR[5]">RFULL</td>
    <td title="xSR[4]">REMTY</td>
    <td title="xSR[3]">TOR</td>
    <td title="xSR[2]">TRDY</td>
    <td title="xSR[1]">TFULL</td>
    <td title="xSR[0]">TEMTY</td>
</tr>
</table>

This register is used for reading Channel *x* status.

ROR - Receiver Overrun

RRDY - Receiver Ready

RFULL - Receiver Full

REMTY - Receiver Empty

ROR - Receiver Overrun

RRDY - Receiver Ready

RFULL - Receiver Full

REMTY - Receiver Empty

See the datasheet for the exact functions of these bits.

[CD-i Emulator] only emulates the REMPTY bit; the other bits read as zero values except TEMTY which always reads as set since command processing
does not take emulated time; this might be different on real hardware.

#### ISR - Interrupt Status Register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ISR[7]">RD</td>
    <td title="ISR[6]">TD</td>
    <td title="ISR[5]">RC</td>
    <td title="ISR[4]">TC</td>
    <td title="ISR[3]">RB</td>
    <td title="ISR[2]">TB</td>
    <td title="ISR[1]">RA</td>
    <td title="ISR[0]">TA</td>
</tr>
</table>

R*x* - Channel *x* Receiver interrupt

T*x* - Channel *x* Transmitter interrupt

Note: the channel bits are reversed from the datasheet as determined by reverse engineering.

The bits of this register indicate conditions that can generate interrupts
to the main 68000 CPU. If the corresponding bit in IMR is set, an interrupt
will be generated whenever a bit of this register becomes set.

Each of the R*x* bits becomes set whenever response message
bytes are available on the corresponding Channel *x* read register.

[CD-i Emulator] never sets the T*x* bits.

#### IMR - Interrupt Mask Register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="IMR[7]">RMD</td>
    <td title="IMR[6]">TMD</td>
    <td title="IMR[5]">RMC</td>
    <td title="IMR[4]">TMC</td>
    <td title="IMR[3]">RMB</td>
    <td title="IMR[2]">TMB</td>
    <td title="IMR[1]">TMA</td>
    <td title="IMR[0]">TMA</td>
</tr>
</table>

RM*x* - Channel *x* Receiver interrupt Mask

TM*x* - Channel *x* Transmitter interrupt Mask

Note: the channel bits are reversed from the datasheet as determined by reverse engineering.

The bits of this register specify the conditions for generating interrupts to
the main 68000 CPU. If a bit in this register is set, an interrupt will be
generated whenever the corresponding bit in ISR becomes set. See the description
of ISR for a description of the conditions associated with each bit.

### MR - Mode Register [sic]

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="MR[7]">IMOD</td>
    <td title="MR[6]">FCLR</td>
    <td title="MR[5]">0</td>
    <td title="MR[4]">0</td>
    <td title="MR[3]">AEN</td>
    <td title="MR[2]">BEN</td>
    <td title="MR[1]">CEN</td>
    <td title="MR[0]">DEN</td>
</tr>
</table>

IMOD - Interrupt Mode

FCLR - FIFO Data Clear

*x*EN - Channel *x* Enable

See the datasheet for the exact functions of these bits.

The CD-i system bootstrap always writes 8F to this register.

[CD-i Emulator] does not emulate any of the above bits.

## Commands

### Channel A

TBA: channel A commands

### Channel B

TBA: channel B commands

### Channel C

#### Reset main cpu
Channel | Hex | Binary | Name
--- | --- | --- | ---
C | 88 | 1000 1000 | `Reset main cpu`

Resets the main 68000 CPU by asserting its RESET input.

**Response:** None, main CPU is reset

#### Get boot mode
Channel | Hex | Binary | Name
--- | --- | --- | ---
C | F4 | 1111 0100 | `Get boot mode`

**Response:** `Boot mode` on Channel C

#### Get video standard
Channel | Hex | Binary | Name
--- | --- | --- | ---
C | F6 | 1111 0110 | `Get video standard`

**Response:** `Video standard` on Channel C

TBA: more channel C commands

### Channel D

#### Get disc status
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | B0 | 1011 0000 | `Get disc status`

**Response:** `Disc status` on Channel D

#### Get disc base
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | B1 | 1011 0001 | `Get disc base`

**Response:** `Disc base` on Channel D

#### Get disc status
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | B2 | 1011 0010 | `Get disc select`

**Response:** `Disc select` on Channel D

#### Start TOC
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | C0 | 1100 0000 | `Start TOC`

Starts TOC read from the lead-in area of the disc.

#### Start CDDA
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | E0 mm ss ff | 1110 0000 *mmmm mmmm* *ssss ssss* *ffff ffff* | `Start CDDA`

Starts CD-DA playback at absolute CD address *mm:ss:ff*.

#### Start READ
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | E1 mm ss ff | 1110 0001 *mmmm mmmm* *ssss ssss* *ffff ffff* | `Start CDDA`

Starts sector reading at absolute CD address *mm:ss:ff*.

TBA: more channel D commands

## Responses

### Channel A

TBA: channel A responses

### Channel B

#### Absolute pointer state
Channel | Hex | Binary | Name
--- | --- | --- | ---
B | 4x xx xx xx ... 7x xx xx xx | 01*ab cccc* 000*d eeee* 00*ff ffff* 10*gg gggg* | `Absolute pointer state`

Reports the current pointer state in absolute coordinates: \
*a* = set when button 2 pressed \
*b* = set when button 1 pressed \
*d* = set when pointer active \
*ccccffffff* = x position \
*eeeegggggg* = y position

The x and y values use the full 0-1023 range to represent 0,0 - 767,559 (high-res PAL).

[CD-i Emulator] uses this response message only when necessary;
the `Relative pointer state` response is used whenever both x and y
position delta values fit in an 8-bit signed value (i.e. are in the -128 ...
+127 range).

#### Relative pointer state
Channel | Hex | Binary | Name
--- | --- | --- | ---
B | 4x xx xx xx ... 7x xx xx xx | 01*ab ccdd* 00*ee eeee* 00*ff ffff* 0000 0000 | `Relative pointer state`

Reports the current pointer state in relative coordinates: \
*a* = set when button 2 pressed \
*b* = set when button 1 pressed \
*cceeeeee* = x position delta (8-bit signed value)\
*ffgggggg* = y position delta (8-bit signed value)

The delta x and y values are based on the full 0-1023 range to represent 0,0 - 767,559 (high-res PAL).

[CD-i Emulator] uses this response message whenever possible;
the `Absolute pointer state` response is only used when one of the x and y
position delta values does not fit in an 8-bit signed value (i.e. is not in the -128 ...
+127 range).

TBA: more channel B responses

### Channel C


#### Boot mode
Channel | Hex | Binary | Name
--- | --- | --- | ---
C | A5 F4 0m | 1010 0101 1111 0100 0000 *mmmm* | `Boot mode`

Reports the boot mode as determined by service plug detection.

*m* | Mode
--- | ---
0 | Boot player shell
1 | Boot service shell

#### Video standard
Channel | Hex | Binary | Name
--- | --- | --- | ---
C | A5 F6 0s ?? | 1010 0101 1111 0110 0000 *ssss* ???? ???? | `Video standard`

Reports the current video standard as taken from IKAT input pin PC7.

*s* | Standard
--- | ---
1 | NTSC / 60 Hz
2 | PAL / 50 Hz

[CD-i Emulator] always returns FF for the ?? byte.

TBA: more channel C responses

### Channel D

#### Disc status
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | B0 ss ss | 1011 0000 *ssss ssss* *ssss ssss* | `Disc status`

Reports the current disc status like door {open,closing,closed,opening}, speed,
focus and wether or not the disc is multi-session (orange book).

[CD-i Emulator] always returns hexadecimal 0210.

#### Disc base
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | B1 mm ss ff | 1011 0001 *mmmm mmmm* *ssss ssss* *ffff ffff* | `Disc base`

Reports the current absolute disc base address *mm:ss:ff* which is always
00:02:00 except for multisession discs.

[CD-i Emulator] always returns 00:02:00.

#### Disc select
Channel | Hex | Binary | Name
--- | --- | --- | ---
D | B2 ss ss ss | 1011 0010 *ssss ssss* *ssss ssss* *ssss ssss* | `Disc status`

Reports the current disc select value.

[CD-i Emulator] always returns hexadecimal 200010.

TBA: more channel D responses

[CD-i Emulator]: https://www.cdiemu.org/cdiemu/
[Mono-III]: https://www.cdiemu.org/players/
[Mono-IV]: https://www.cdiemu.org/players/
