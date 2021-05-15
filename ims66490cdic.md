# IMS66490 CD-Interface Controller (CDIC)

The IMS66490 CDIC implements a CD drive interface suitable for decoding CD-XA and CD-i
disc sectors and directly supports the corresponding audio functions including
mixing, external audio and decoding ADPCM from CD and from memory.

No data sheet or technical summary of the CDIC is publicly available.
This document is an attempt to describe the CDIC
registers with enough level of detail so that they can be emulated in software.

The information in this document has been determined by reverse engineering the
CD-i CDIC drivers *cdap18x* and *cdapdriv* used by all CD-i players with a
[JNMS], [Maxi-MMC], [Mini-MMC] or [Mono-I] mainboard.

Because of this, the exact behaviour of register bits is often unclear. In some
of these cases the emulation used by [CD-i Emulator] is described.

In CD-i players the CDIC is connected to the main 68000 CPU as host CPU.

## Memory map

The following memory map has mostly been determined from reverse engineering. 

All memory addresses are hexadecimal and relative to the base address of
the CDIC. In CD-i players this is always 00300000 so that for example DATA
buffer 1 is located at main CPU address 00300A00.

The CDIC is a 16-bit wide device and is not connected to the main 68000 CPU A0 signal,
therefore all of the memory addresses below are even and all accesses must use word bus cycles.
The 68000 CPU supports reading bytes, words and long words using such cycles but
all writing must be done with aligned word or long word operations.

Address | Description
--- | ---
0000 - 09FF | DATA buffer 0
0A00 - 13FF | DATA buffer 1
2800 - 31FF | ADPCM buffer 0
3200 - 3BFF | ADPCM buffer 1
3C00 - 3FFE | Register area

Note: According to the low-level test the CDIC memory ranges from 0000 to 3C7F
which means that part of the register area is actually in CDIC memory. This
means that some registers are actually in CDIC memory and are probably
implemented by the CDIC microcode.

## Register map

The following register map is mostly based on reverse engineering.

Address | Name | Description
--- | --- | ---
3C00 | CMD | Command register
3C02 | TIME | Time register
3C06 | FILE | File select register
3C08 | CHAN | Channel mask register
3C0C | ACHAN | Audio channel mask register
3C80 | DSEL | Data select register
3FF4 | ABUF | Audio buffer register
3FF6 | XBUF | Extra buffer register
3FF8 | DMACTL | DMA control register
3FFA | AUDCTL | Audio control register
3FFC | IVEC | Interrupt vector register
3FFE | DBUF | Data buffer register

The CD-i CDIC driver uses word read and write operations for all registers except
TIME and CHAN for which it always uses long read and write operations.

[CD-i Emulator] does not implement access other then described above.

## Concept of operation

The CDIC sector decoder accepts incoming serial data from a CD drive and uses
the embedded framing information to split this into frames of 1/75th second.

Each frame is then decoded into 2340 bytes of mainchannel data and XXX bytes of
subchannel data, 12 bytes each for subchannels Q to W. The mainchannel sector
sync bytes and subchannel frame sync bits are not made available to the host
processor.

The mainchannel data part of each sector starts with four bytes of minute,
second, frame, and mode information. For mode 2 sectors this is directly followed
by two four byte sets of file, channel, submode and coding information. The CDIC can
use filtering on the file and channel bytes to select sectors for delivery to
memory and can also optionally send the ADPCM contents of mode 2 audio sectors
in specific channels directly to the CDIC audio processor for decoding.

During mode 2 read commands the CDIC will switch from DATA to ADPCM buffers
after the first four byte set of file, channel, submode and coding information
when the channel has been selected for direct ADPCM decoding.

The CDIC does not implement control functions for CD drives. For CD-i these
functions are implemented by a separate 68HC05 microcontroller called SLAVE.

The CDIC is a probably a microcoded device but if so the microcode is contained
in the device and not downloaded from the main CPU.

The CDIC can delay assertion of it's DTACK (Data Transfer ACKnowledge) output
to briefly halt the main 68000 CPU when needed; the exact timing dependencies
are not currently known.

## Register description

Most of the information below has been determined by reverse engineering.

When the function of individual bits is not known this has been indicated with a
question mark ? in the corresponding bit position. For emulation purposes these
bits should be preserved when written but otherwise not used.

TBA

[CD-i Emulator] never asserts the DONE output signal.

[JNMS]: http://www.cdiemu.org/players/
[Maxi-MMC]: http://www.cdiemu.org/players/
[Mini-MMC]: http://www.cdiemu.org/players/
[Mono-I]: http://www.cdiemu.org/players/
[CD-i Emulator]: http://www.cdiemu.org/cdiemu/
