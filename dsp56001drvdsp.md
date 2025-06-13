# DSP56001 Digital Signal Processor (DRVDSP)

The DSP56001 CD-Interface program implements a CD drive interface suitable for
decoding CD-XA and CD-i disc sectors and directly supports the corresponding
audio functions including mixing, external audio and decoding ADPCM from CD and
from memory.

The data sheet of the base DSP56001 chip is publicly available but interface
documentation for the DRVDSP program is not. This document is an attempt to
describe the DRVDSP functions with enough level of detail so that they can be
emulated in software.

The information in this document has mostly been determined by reverse
engineering the CD-i *dspdriv* driver for the CD-i [Mono-II] mainboard
generation that uses the DSP56001 digital signal processor.

Because of this, the exact behaviour of many functions is unclear. In some of
these cases the emulation used by [CD-i Emulator] is described.

In CD-i players the DSP is connected to the main 68000 CPU as host CPU.
It is always coupled with a Philips LEMM CD interface chip;
the LEMM chip is not connected to the main CPU bus.

## Memory map

The following memory map has been taken from the service manual.

All memory addresses are hexadecimal and relative to the base address of the
DSP. In CD-i players this is always 00300000 so that for example register 1 is
located at main CPU address 00300001.

The DSP is an 8-bit wide device and is not connected to the main 68000 CPU A0 signal,
therefore all of the memory addresses below are odd and all accesses must use byte bus cycles.

Address | Description
--- | ---
0-F | Register area

## Register map

The following register map has been taken from the DSP56001 data sheet.


Address | Name | Description
--- | --- | ---
1 | ICR | Interrupt control register
3 | CVR | Command vector register
5 | ISR | Interrupt status register
7 | IVR | Interrupt vector register
B | RXH/TXH | High byte receive/transmit register
D | RXM/TXM | Middle byte receive/transmit register
F | RXL/TXL | Low byte receive/transmit register

The CD-i DSP driver uses byte read and write operations for all registers.

[CD-i Emulator] does not implement word or long word access operations.

## Concept of operation

The DRVDSP sector decoder accepts incoming serial data from a CD drive and uses
the embedded framing information to split this into frames of 1/75th second.

Each frame is then decoded into 2340 bytes of mainchannel data and 84 bytes of
subchannel data, 12 bytes each for subchannels Q to W. The mainchannel sector
sync bytes and subchannel frame sync bits are not made available to the host
processor.

The mainchannel data part of each sector starts with four bytes of minute,
second, frame, and mode information. For mode 2 sectors this is directly followed
by two four byte sets of file, channel, submode and coding information. The DRVDSP can
use filtering on the file and channel bytes to select sectors for delivery to
memory and can also optionally send the ADPCM contents of mode 2 audio sectors
in specific channels directly to the DRVDSP audio processor for decoding.

The DRVDSP program does not implement control functions for CD drives. For CD-i these
functions are implemented by a separate 68HC05 microcontroller called SLAVE.

The DSP56001 is a microcoded device; the CD-i DSP driver downloads the microcode
from the OS-9 data module *dspcode* into the DSP Program areas on device
initialization (INIT entry point).

## Register description

Most of the information below has been taken from the data sheet but the specific
meanings of the HF (Host Flag) and HV (Host Vector) are depend on the DRVDSP program
and have been determined by reverse engineering.

When the function of individual bits is not known this has been indicated with a
question mark ? in the corresponding bit position. For emulation purposes these
bits should be preserved when written but otherwise not used.

TBA

#### ICR - Interrupt control register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ICR[7]">INIT</td>
    <td title="ICR[6]">HM1</td>
    <td title="ICR[5]">HM0</td>
    <td title="ICR[4]">HF1</td>
    <td title="ICR[3]">HF0</td>
    <td title="ICR[2]">0</td>
    <td title="ICR[1]">TREQ</td>
    <td title="ICR[0]">RREQ</td>
</tr>
</table>

The bits of the this register control host interrupts.

The INIT bit ...

The HM1/HM0 (Host Mode) bits ...

The HF1/HF0 (Host Flag) bits are general purpose mode bits for host-to-DSP communication.

The TREQ bit ...

The RREQ bit ...

TBA

#### CVR - Command vector register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="CVR[7]">HC</td>
    <td title="CVR[6-5]" align="center" colspan="2">0</td>
    <td title="CVR[4-0]" align="center" colspan="5">HV</td>
</tr>
</table>

The HC (Host Command) bit is set to trigger a host command exception inside the
DSP and will be cleared when the exception is acknowledged by the DSP.

The HV (Host Vector) bits specify the host command exception address inside the
DSP.

The following bytes are written by the CD-i DSP driver:

Byte | Command
---|---
80 | Run program
88 | Submit buffer 4
89 | Submit buffer 5
8E | Set read mode
92 | Read audio status
93 | Read status
94 | Audio something 1
95 | Audio something 2
96 | Start sector
97 | Start DMA read
98 | Start DMA write
99 | Stop DMA
9A | Read memory
9B | Write memory
9C | Get unknown status
9D | Select sectors
9F | Set volume

Some commands require that a 24-bit parameter value is written to the
TXH/TXM/TXL registers before they are issued; commands can also return a number
of 24-bit result values that must be read from the RXH/RXM/RXL register.

#### ISR - Interrupt status register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ISR[7]">HREQ</td>
    <td title="ISR[6]">DMA</td>
    <td title="ISR[5]">0</td>
    <td title="ISR[4]">HF3</td>
    <td title="ISR[3]">HF2</td>
    <td title="ISR[2]">TRDY</td>
    <td title="ISR[1]">TXDE</td>
    <td title="ISR[0]">RXDF</td>
</tr>
</table>

The bits of the this register indicate DSP status.

The HREQ bit indicates if a host request is pending (DMA or interrupt).

The DMA bit indicates if DMA mode is enabled.

The HF3/HF2 (Host Flag) bits are general purpose mode bits for DSP-to-host communication.

The TRDY bit indicates that the TXH/TXM/TXL register combination is empty
as well as the DSP HRX register.

The TXDE bit indicates that the TXH/TXM/TXL register combination is empty, i.e.
does not contain host data for the DSP to read. Writing of TXL resets this bit
which why it is usually written last.

The RXDF bit indicates that the RXH/RXM/RXL register combination is full, i.e.
contains DSP data for the host to read. Reading of RXL resets this bit which why
it is usually read last.

The CD-i DSP driver only checks the TXDE and RXDF bits.

[CD-i Emulator] only emulates the TXDE and RXDF bits.

#### IVR - Interrupt vector register

<table>
<tr>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="IVR[7-0]" align="center" colspan="8">Vector</td>
</tr>
</table>

This register controls the host interrupt vector of the DSP.

The DSP driver writes this register at device initialization (INIT entry point)
using the values from the device descriptor.

#### RXH/RXM/RXL - Byte receive registers

Bytes read from these registers retrieve the corresponding byte from the DSP HTX
register. Reading RXL clears the DSP RXDF status bit and is therefore normally
the last register read.

#### TXH/TXM/TXL - Byte transmit registers

Bytes written to these registers set the corresponding byte of the DSP HRX
register. Writing TXL transfers all three bytes to the DSP HRX register and clears
the DSP TXDE status bit and is therefore normally the last register written.

## DRVDSP commands

This section documents the commands implemented by the DRVDSP program.

The host executes a DSP command by performing the following steps:
- write a 24-bit prefix parameter value to the TXH/TXM/TXL registers (optional)
- write a command byte to the CVR register
- write a specific number of 24-bit argument values to the TXH/TXM/TXL registers
- read a specific number of 24-bit result values from the RXH/RXM/RXL registers

In the command descriptions below, the command byte is shown in **bold**.

#### BUF4 - Submit buffer 4

Command | Name
---|---
**88** | Submit buffer 4

#### BUF6 - Submit buffer 6

Command | Name
---|---
**89** | Submit buffer 6

#### Set read mode

Command | Name
---|---
mode **8E** | Set read mode

Mode | Name | Description
---|---|---
0000 | IDLE | 
#### 92 - Read audio status
#### 93 - Read status
#### 94 - Audio something 1
#### 95 - Audio something 2
#### 96 - Start sector
#### 97 - Start DMA read
#### 98 - Start DMA write
#### 99 - Stop DMA
#### 9A - Read memory
#### 9B - Write memory
#### 9C - Get unknown status
#### 9D - Select sectors
#### 9F - Set volume



[Mono-II]: https://www.cdiemu.org/players/
[CD-i Emulator]: https://www.cdiemu.org/cdiemu/
