# MCD221 CD-Interface and Audio Processor (CIAP)

The MCD221 CIAP implements a CD drive interface suitable for decoding CD-XA and CD-i
disc sectors and directly supports the corresponding audio functions including
mixing, external audio and decoding ADPCM from CD and from memory.

A technical summary of the CIAP is publicly available ([MCD221ts]) but a
complete data sheet is not. This document is an attempt to describe the CIAP
registers with enough level of detail so that they can be emulated in software.

The information in this document beyond what is described in the technical
summary has been determined by reverse engineering the CD-i CIAP driver
*ciapdriv* used by all CD-i players with a [Mono-III] and later mainboard
generation. This includes an additional register and all of functions of the
register bits.

Because of this, the exact behaviour of register bits is often unclear. In some
of these cases the emulation used by [CD-i Emulator] is described.

In CD-i players the CIAP is directly connected to the main 68000 CPU as host CPU.
The chip also supports a serial host CPU interface; no information about that option
is included in this document.

## Memory map

The following memory map has mostly been taken from the technical summary but
has been augmented with information from reverse engineering. 

All memory addresses are hexadecimal and relative to the base address of
the CIAP. In CD-i players this is always 00300000 so that for example ADPCM
buffer 1 is located at main CPU address 00300900.

The CIAP is a 16-bit wide device and does not have an A0 input pin, therefore all
of the memory addresses below are even and all accesses must use word bus cycles.
The 68000 CPU supports reading bytes, words and long words using such cycles but
all writing must be done with aligned words or long word operations.

Address | Description
- | -
0000 - 08FE | ADPCM buffer 0
0900 - 11FE | ADPCM buffer 1
1200 - 1B22 | Mainchannel DATA buffer 0
1B24 - 1B2C | Subchannel Q buffer 0
1B2E - 1B8C | Subchannel R-W buffer 0
1BC2 - 24E4 | Mainchannel DATA buffer 1
24E6 - 24EE | Subchannel Q buffer 1
24F0 - 254E | Subchannel R-W buffer 1
2550 - 257E | Unknown
2580 - 25FE | Register area
2600 - 27FE | Program area 1
2800 - 2FFE | Program area 2
3000 - 3FFF | Program area 3

## Register map

The following register map is based on the technical summary but has been
augmented with information obtained by reverse engineering. Registers not in the
technical summary are marked with a * symbol.

Address | Name | Description
- | - | -
2584 | IER | Interrupt enable register
2586 | ISR | Interrupt status register
2588 | TACS | Temporal audio channel select register
258A | AACS | Actual audio channel select register
258C | TCM1 | Temporal channel mask register
258E | ACM1 | Actual channel mask register 1
2590 | ACM2 | Actual channel mask register 2
2592 | FILE | File selection register
2594 | BMAN | Buffer management register
2596 | CCR | CIAP control register
259A | A_SHDW | ADPCM shadow register
25A0 | AP_Left | Audio processor unit left register
25A2 | AP_Right | Audio processor unit right register
25A4 | AP_Vol | Audio processor unit volume register
25A6 | APCR | Audio processor control register
25A8 | ACONF | Audio configuration register
25AA | ASTAT | Audio processor status register
25C0 | ICR | Interrupt control register
25C2 | DMACTL | DMA control register
25C4 | ID* | Identification register
25FE | DLOAD | Download register

The CD-i CIAP driver uses word read and write operations for all registers. However,
the CD-i low-level tests use byte reads for the ID register.

[CD-i Emulator] does not implement word access for any register except the ID register.

## Concept of operation

The CIAP sector decoder accepts incoming serial data from a CD drive and uses
the embedded framing information to split this into frames of 1/75th second.
Each frame is then decoded into 2340 bytes of mainchannel data and 150 bytes of
subchannel data, 10 bytes for subchannel Q and 12 bytes each for subchannels R to W.
The mainchannel sector sync bytes and subchannel frame sync bits
are not made available to the host processor and neither are the
subchannel Q CRC bytes.

The mainchannel data part of each sector starts with four bytes of minute,
second, frame, andmode information. For mode 2 sectors this is directly followed
by two sets of four file, channel, submode and coding information. The CIAP can
use filtering on the file and channel bytes to select sectors for delivery to
memory and can also optionally send the ADPCM contents of mode 2 audio sectors
in specific channels directly to the CIAP audio processor for decoding.

The CIAP does not implement control functions for CD drives. For CD-i these
functions are implemented by a separate 68HC05 microcontroller called IKAT.

The CIAP is a microcoded device; the CD-i CIAP driver downloads the microcode
from the main CPU ROM into CIAP Program areas 1 to 3 on device initialization
(INIT entry point).

## Register description

Except for most of the register names, all of the information below has been
determined by reverse engineering.

When the function of individual bits is not known this has been indicated with a
question mark ? in the corresponding bit position. For emulation purposes these
bits should be preserved when written but otherwise not used.

#### IER - Interrupt enable register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="IER[15]">?</td>
    <td title="IER[14]">?</td>
    <td title="IER[13]">?</td>
    <td title="IER[12]">?</td>
    <td title="IER[11]">QERROR</td>
    <td title="IER[10]">?</td>
    <td title="IER[9]">?</td>
    <td title="IER[8]">?</td>
    <td title="IER[7]">?</td>
    <td title="IER[6]">?</td>
    <td title="IER[5]">?</td>
    <td title="IER[4]">?</td>
    <td title="IER[3]">AUDIO</td>
    <td title="IER[2]">SUBCODE</td>
    <td title="IER[1]">?</td>
    <td title="IER[0]">DATA</td>
</tr>
</table>

The bits of this register specify the conditions for generating interrupts to
the host CPU. If a bit in this register is set, an interrupt will be
generated whenever the corresponding bit in ISR becomes set. See the description
of ISR for a description of the conditions associated with each bit.

#### ISR - Interrupt status register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ISR[15]">?</td>
    <td title="ISR[14]">?</td>
    <td title="ISR[13]">?</td>
    <td title="ISR[12]">?</td>
    <td title="ISR[11]">QERROR</td>
    <td title="ISR[10]">?</td>
    <td title="ISR[9]">?</td>
    <td title="ISR[8]">?</td>
    <td title="ISR[7]">?</td>
    <td title="ISR[6]">?</td>
    <td title="ISR[5]">?</td>
    <td title="ISR[4]">?</td>
    <td title="ISR[3]">AUDIO</td>
    <td title="ISR[2]">SUBCODE</td>
    <td title="ISR[1]">?</td>
    <td title="ISR[0]">DATA</td>
</tr>
</table>

The bits of the this register indicate conditions that can generate interrupts
to the host CPU. If the corresponding bit in IER is set, an interrupt
will be generated whenever a bit becomes set.

Reading this register will clear all the bits and any corresponding interrupt.

The DATA bit indicates that new sector data has been read from the CD drive and
is available in the current Mainchannel DATA buffer.

The SUBCODE bit indicates that new subcode data has been read from the CD drive
and is available in the current Subchannel Q and R-W buffers.

The AUDIO bit indicates that an interrupt condition was generated by the audio
decoder.

The QERROR bit indicates that the Subchannel Q buffer is not valid,
probably because it has a bad CRC in the last bytes.

[CD-i Emulator] never sets the QERROR bit.

#### TACS - Temporal audio channel select register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="TACS[15]">Ch 15</td>
    <td title="TACS[14]">Ch 14</td>
    <td title="TACS[13]">Ch 13</td>
    <td title="TACS[12]">Ch 12</td>
    <td title="TACS[11]">Ch 11</td>
    <td title="TACS[10]">Ch 10</td>
    <td title="TACS[9]">Ch 9</td>
    <td title="TACS[8]">Ch 8</td>
    <td title="TACS[7]">Ch 7</td>
    <td title="TACS[6]">Ch 6</td>
    <td title="TACS[5]">Ch 5</td>
    <td title="TACS[4]">Ch 4</td>
    <td title="TACS[3]">Ch 3</td>
    <td title="TACS[2]">Ch 2</td>
    <td title="TACS[1]">Ch 1</td>
    <td title="TACS[0]">Ch 0</td>
</tr>
</table>

For Mode 2 sectors the bit of this register corresponding to the submode channel byte
must be set for ADPCM data from audio sectors to be directly decoded by the audio processor
without being passed to the host processor.

The CD-i CIAP driver always writes the FILE, TACS, TCM1, ACM2 registers
together and in that order. The AACS and ACM1 registers are never written.

#### AACS - Actual audio channel select register

The CD-i CIAP driver never accesses this register.

#### TCM1 - Temporal channel mask register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="TCM1[15]">Ch 15</td>
    <td title="TCM1[14]">Ch 14</td>
    <td title="TCM1[13]">Ch 13</td>
    <td title="TCM1[12]">Ch 12</td>
    <td title="TCM1[11]">Ch 11</td>
    <td title="TCM1[10]">Ch 10</td>
    <td title="TCM1[9]">Ch 9</td>
    <td title="TCM1[8]">Ch 8</td>
    <td title="TCM1[7]">Ch 7</td>
    <td title="TCM1[6]">Ch 6</td>
    <td title="TCM1[5]">Ch 5</td>
    <td title="TCM1[4]">Ch 4</td>
    <td title="TCM1[3]">Ch 3</td>
    <td title="TCM1[2]">Ch 2</td>
    <td title="TCM1[1]">Ch 1</td>
    <td title="TCM1[0]">Ch 0</td>
</tr>
</table>

For Mode 2 sectors the bit of this register corresponding to the submode channel byte
must be set for sector sectors to be selected for delivery to memory.

The CD-i CIAP driver always writes the FILE, TACS, TCM1, ACM2 registers
together and in that order. The AACS and ACM1 registers are never written.

#### ACM1 - Actual channel mask register 1

The CD-i CIAP driver never accesses this register.

#### ACM2 - Actual channel mask register 2

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ACM2[15]">Ch 31</td>
    <td title="ACM2[14]">Ch 30</td>
    <td title="ACM2[13]">Ch 29</td>
    <td title="ACM2[12]">Ch 28</td>
    <td title="ACM2[11]">Ch 27</td>
    <td title="ACM2[10]">Ch 26</td>
    <td title="ACM2[9]">Ch 25</td>
    <td title="ACM2[8]">Ch 24</td>
    <td title="ACM2[7]">Ch 23</td>
    <td title="ACM2[6]">Ch 22</td>
    <td title="ACM2[5]">Ch 21</td>
    <td title="ACM2[4]">Ch 20</td>
    <td title="ACM2[3]">Ch 19</td>
    <td title="ACM2[2]">Ch 18</td>
    <td title="ACM2[1]">Ch 17</td>
    <td title="ACM2[0]">Ch 16</td>
</tr>
</table>

For Mode 2 sectors the bit of this register corresponding to the submode channel byte
must be set for sector sectors to be selected for delivery to memory.

The CD-i CIAP driver always writes the FILE, TACS, TCM1, ACM2 registers
together and in that order. The AACS and ACM1 registers are never written.

#### FILE - File selection register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="FILE[15]">?</td>
    <td title="FILE[14]">?</td>
    <td title="FILE[13]">?</td>
    <td title="FILE[12]">?</td>
    <td title="FILE[11]">?</td>
    <td title="FILE[10]">?</td>
    <td title="FILE[9]">?</td>
    <td title="FILE[8]">?</td>
    <td title="FILE[7-0]" align="center" colspan="8">File number</td>
</tr>
</table>

For Mode 2 sectors the submode file byte must match the file number in this register
for that sector the be selected for delivery to memory.

The CD-i CIAP driver always writes the FILE, TACS, TCM1, ACM2 registers
together and in that order. The AACS and ACM1 registers are never accessed.

#### BMAN - Buffer management register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="BMAN[15]">?</td>
    <td title="BMAN[14]">?</td>
    <td title="BMAN[13]">?</td>
    <td title="BMAN[12]">?</td>
    <td title="BMAN[11]">?</td>
    <td title="BMAN[10]">?</td>
    <td title="BMAN[9]">?</td>
    <td title="BMAN[8]">?</td>
    <td title="BMAN[7]">XXX1?</td>
    <td title="BMAN[6]">XXX0?</td>
    <td title="BMAN[5]">SUBCH1</td>
    <td title="BMAN[4]">SUBCH0</td>
    <td title="BMAN[3]">DATA1</td>
    <td title="BMAN[2]">DATA0</td>
    <td title="BMAN[1]">ADPCM1</td>
    <td title="BMAN[0]">ADPCM0</td>
</tr>
</table>

This register controls buffer usage during disc reading and audio playback.

Each of the ADPCM*n* bits indicates if ADPCM buffer *n* contains ADPCM audio data
to be decoded by the audio processor.

Each of the DATA*n* bits indicates if Mainchannel DATA buffer *n* contains
sector data that has been decoded from CD.

Each of the SUBCH*n* bits indicates if Subchannel Q buffer *n* contains
subchannel Q data that has been decoded from CD.

Each of the XXX*n* bits indicates if XXX buffer *n* contains XXX.

The XXX buffers are located inside ADPCM buffer 1 at offset 00 and C0,
respectively.

Writing to this register results in the buffer usage bits being XORred
with the bit values written, i.e. writing of a set bit toggles the corresponding
register bit, writing of a clear bit does nothing.

During regular CIAP operation, each bit pair is toggled when the corresponding
buffer has been processed.

#### CCR - CIAP control register

This register accepts control commands for the CIAP.

The following words are written by the CD-i CIAP driver.

Word | Name | Description
- | - | -
0008 | ASEL | Activate selection on next sector
0040 | ??? | ?
0094 | STARTA | ?
00C4 | STARTD | ?
0100 | RESET | Reset CIAP
3000 | PREPA | ?
7000 | PREPD | ?

#### A_SHDW - ADPCM shadow register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="A_SHDW[15]">?</td>
    <td title="A_SHDW[14]">?</td>
    <td title="A_SHDW[13]">BITS8</td>
    <td title="A_SHDW[12]">FREQ18K9</td>
    <td title="A_SHDW[11]">STEREO</td>
    <td title="A_SHDW[10]">?</td>
    <td title="A_SHDW[9]">?</td>
    <td title="A_SHDW[8]">?</td>
    <td title="A_SHDW[7]">?</td>
    <td title="A_SHDW[6]">?</td>
    <td title="A_SHDW[5]">?</td>
    <td title="A_SHDW[4]">?</td>
    <td title="A_SHDW[3]">EMPH</td>
    <td title="A_SHDW[2]">?</td>
    <td title="A_SHDW[1]">?</td>
    <td title="A_SHDW[0]">?</td>
</tr>
</table>

The bits in this register specify the format of the ADPCM data in the ADPCM
buffers. The format information is copied to an internal register for each ADPCM
buffer when the corresponding bit of the BMAN register is set by the host CPU.

The EMPH, STEREO, FREQ18K9 and BITS8 bit correspond directly to bits of the
submode coding byte for audio sectors as follows:

7 | 6 | 5 | 4 | 3 | 2 | 1 | 0
- | - | - | - | - | - | - | -
0 | EMPH | 0 | BITS8 | 0 | FREQ18K9 | 0 | STEREO
 
#### AP_Left - Audio processor unit left register

This register controls audio mixing.

The CD-i CIAP driver always writes the AP_Right and AP_Left registers
together and in that order.

[CD-i Emulator] does not currently implement this register.

#### AP_Right - Audio processor unit right register

This register controls audio mixing.

The CD-i CIAP driver always writes the AP_Right and AP_Left registers
together and in that order.

[CD-i Emulator] does not currently implement this register.

#### AP_Vol - Audio processor unit volume register

The CD-i CIAP driver never accesses this register.

#### APCR - Audio processor control register

This register accepts control commands for the audio processor.

The following words are written by the CD-i CIAP driver.

Word | Name | Description
- | - | -
0020 | INTDONE | Generate interrupt when audio playback is done
00A0 | INTNOW | Generate an immediate interrupt
0140 | PLAY0 | Start audio playback with ADPCM buffer 0

#### ACONF - Audio configuration register

The CD-i CIAP driver writes the following words to this register:

Word | Name | Description
- | - | -
0002 | ? | ?
0003 | ? | ?

#### ASTAT - Audio processor status register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ASTAT[15]">?</td>
    <td title="ASTAT[14]">?</td>
    <td title="ASTAT[13]">?</td>
    <td title="ASTAT[12]">?</td>
    <td title="ASTAT[11]">?</td>
    <td title="ASTAT[10]">?</td>
    <td title="ASTAT[9]">?</td>
    <td title="ASTAT[8]">?</td>
    <td title="ASTAT[7]">INT</td>
    <td title="ASTAT[6]">?</td>
    <td title="ASTAT[5]">?</td>
    <td title="ASTAT[4]">?</td>
    <td title="ASTAT[3]">?</td>
    <td title="ASTAT[2]">?</td>
    <td title="ASTAT[1]">?</td>
    <td title="ASTAT[0]">?</td>
</tr>
</table>

The bits in this register indicate the status of the audio processor.

The INT bit indicates that the AUDIO bit in ISR has been set by the audio processor.
It is unclear when this bit should be cleared; [CD-i Emulator] never clears it.

On reset, this register is initialized to hexadecimal 0400.

The CD-i CIAP driver never writes to this register.

#### ICR - Interrupt control register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ASTAT[15]">?</td>
    <td title="ASTAT[14]">?</td>
    <td title="ASTAT[13]">?</td>
    <td title="ASTAT[12]">?</td>
    <td title="ASTAT[11]">?</td>
    <td title="ASTAT[10-3]" align="center" colspan="8">Vector</td>
    <td title="ICR[2-0]" align="center" colspan="3">Level</td>
</tr>
</table>

This register controls the CIAP interrupt vector and level.

The CD-i CIAP driver writes this register at device initialization (INIT entry
point) using the values from the device descriptor.

#### ID* - Identification register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="ID[15-12]" colspan="4" align="center">C</td>
    <td title="ID[11-7]" colspan="4" align="center">D</td>
    <td title="ID[8-4]" colspan="4" align="center">0</td>
    <td title="ID[4-0]" colspan="4" align="center">2</td>
</tr>
</table>

The ID register identifies the CIAP version and is read-only.
All read operations will return the hexadecimal value CD02.

Note: The CD-i low level tests use byte operations to read this register.

#### DMACTL - DMA control register

<table>
<tr>
    <th>15</th><th>14</th><th>13</th><th>12</th>
    <th>11</th><th>10</th><th>9</th><th>8</th>
    <th>7</th><th>6</th><th>5</th><th>4</th>
    <th>3</th><th>2</th><th>1</th><th>0</th>
</tr>
<tr>
    <td title="DMACTL[15]">?</td>
    <td title="DMACTL[14]">START</td>
    <td title="DMACTL[13]">DIR</td>
    <td title="DMACTL[12..0]" align="center" colspan="13">Word offset</td>
</tr>
</table>

This register controls DMA transfers.

The word offset specifies the buffer address for DMA access.
This is automatically incremented by one after each transfer.

The DIR bit controls the direction of DMA operation.

Setting the START bit starts DMA operation by setting the REQ output signal.
This will cause the system DMA controller (when appropriately programmed)
to perform back-to-back word DMA transfers.

[CD-i Emulator] never asserts the DONE output signal.

#### DLOAD - Download register

The DLOAD register controls downloading of microcode into Program areas 1 to 3.

The CD-i CIAP driver writes 0001 to this register before downloading the
microcode and writes 0000 after. As configured by the CIAP device
descriptor modules, microcode from OS-9 data modules is downloaded as follows:

Module | Area
- | -
*ciap15_mc0* | Program area 1
*ciap15_mc1* | Program area 2
*ciap15_mc2* | Program area 3


[MCD221ts]: http://www.icdia.co.uk/docs/mcd221tsrev0.zip
[Mono-III]: http://www.cdiemu.org/players/
[CD-i Emulator]: http://www.cdiemu.org/cdiemu/
