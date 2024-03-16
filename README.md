# CD-i chip information

[CD-i players] contain a number of chips for which a complete datasheet or
programming information is not publicly available. This presents a problem for
authors of CD-i emulators which need to emulate these chips.

When available, complete datasheets can usually be found in the [ICDIA] website
in the [CD-i Technical Documentation / System] section. This section also
contains technical summary documentation for some chips.

The main chips of various CD-i player models are described briefly below,
grouped by function. The descriptions list the CD-i player mainboard generations
that use the chip and provide links to public datasheet documents and files in
this repository.

The files in this repository contain additional information about the chips that
was determined by reverse engineering their drivers as contained in the CD-i
system ROMs and tested by emulating them in the [CD-i Emulator] program.

[CD-i Players]: https://www.cdiemu.org/players/
[CD-i Emulator]: https://www.cdiemu.org/cdiemu/
[ICDIA]: http://www.icdia.co.uk/
[CD-i Technical Documentation / System]: http://www.icdia.co.uk/docs/

## Processor chips

#### Motorola MC68HC000 Central Processing Unit

**Data sheet:** http://www.icdia.co.uk/docs/m68000.zip \
**CD-i boards:** Sony

#### Philips SCC68070 Central Processing Unit

**Data sheet:** http://www.icdia.co.uk/docs/scc68070.zip \
**CD-i boards:** JNMS, Maxi-MMC, Mini-MMC, all Mono boards

#### Toshiba TMP68305 Central Processing Unit

**Data sheet:** http://www.icdia.co.uk/docs/tmp683xx.zip \
**CD-i boards:** Kyocera

#### Motorola MC68340 Central Processing Unit

**Data sheet:** http://www.icdia.co.uk/docs/mc68340.zip \
**CD-i boards:** Goldstar

#### Motorola MC68341 Central Processing Unit

**Data sheet:** http://www.icdia.co.uk/docs/mc68341.zip \
**CD-i boards:** I2m

## Video chips

#### VSC - Philips SCC66470 Video and System Controller

CD-i players use two of these chips as front-end video controllers in
a master/slave configuration. Each chip controls 512KB of video memory and
processes image control and data information for a single plane, performing
run-length and mosaic decoding. The combined outputs of both chips
are then fed to a separate back-end video decoder/synthesizer chip.

**Data sheet:** http://www.icdia.co.uk/docs/scc66470.pdf \
**CD-i boards:** JNMS, Maxi-MMC, Mini-MMC, Sony, Kyocera

#### Dual VSC - Sony CXD8297AQ Dual Video and System Controller

CD-i players use this chip as front-end video controllers in
a master/slave configuration. The chip controls 2x512KB of video memory and
processes image control and data information for each single plane, performing
run-length and mosaic decoding. The outputs of both planes
are then fed to a separate back-end video decoder/synthesizer chip.

**Data sheet:** http://www.icdia.co.uk/docs/scc66470.pdf \
**CD-i boards:** JNMS, Maxi-MMC, Mini-MMC, Sony, Kyocera

#### VSR - Matsushita MN66460B Video Synthesizer

CD-i players use this back-end video synthesizer chip to decode CLUT, DYUV and
RGB555 formats, perform region and transparency processing and combine the
resulting RGB data of both planes and a cursor plane into analog RGB outputs.

**Data sheet:** Not available \
**Additional information**: TBA \
**CD-i boards:** JNMS, Sony

#### VSD - Motorola GSC38TG307 Video Systhesizer Decoder

CD-i players use this back-end video decoder chip to decode CLUT, DYUV and
RGB555 formats, perform region and transparency processing and combine the
resulting RGB data of both planes and a cursor plane into digital RGB888 outputs
which are then fed to a separate video DAC chip.

**Data sheet:** Not available \
**Additional information**: TBA \
**CD-i boards:** Maxi-MMC, Mini-MMC

#### VDSC - Motorola MCD212 Video Decoder and System Controller

CD-i players use this chip to perform all video decoding; it replaces the
multiple-chip video implementation used in earlier players. This chip controls up
to 2x2MB of video memory and processes image control and data information for both
planes, performing and run-length and mosaic decoding followed by decoding CLUT,
DYUV and RGB555 formats, performing region and transparency processing and then
combining the resulting RGB data of both planes and a cursor plane
into digital RGB888 outputs which are then
fed to a separate video DAC chip.

**Data sheet:** http://www.icdia.co.uk/docs/mcd212rev0.pdf \
**CD-i boards:** All Mono boards, Goldstar, I2m

## CD/Audio chips

#### CDIC - Philips IMS66490 CD-Interface Controller

The chip implements a CD drive interface suitable for decoding CD-XA and CD-i
disc sectors and directly supports the corresponding audio functions including
mixing, external audio and decoding ADPCM from CD and from memory.

**Data sheet:** Not available \
**Additional information**: [ims66490cdic.md](ims66490cdic.md) in this repository \
**CD-i boards:** JNMS, Maxi-MMC, Mini-MMC, Mono-I

#### DSP - Motorola DSP56001 Digital Signal Processor

This digital signal processor implements a CD drive interface suitable for
decoding CD-XA and CD-i disc sectors and directly supports the corresponding
audio functions including mixing, external audio and decoding ADPCM from CD and
from memory.

**Data sheet:** http://www.icdia.co.uk/docs/dsp56001.zip \
**Additional information**: TBA \
**CD-i boards:** Mono-II

#### CIAP - Motorola MCD221 CD-Interface and Audio Processor

This chip implements a CD drive interface suitable for decoding CD-XA and CD-i
disc sectors and directly supports the corresponding audio functions including
mixing, external audio and decoding ADPCM from CD and from memory.

**Data sheet:** Not available \
**Technical summary:** http://www.icdia.co.uk/docs/mcd221tsrev0.zip \
**Additional information:** [mcd221ciap.md](mcd221ciap.md) in this repository \
**CD-i boards:** Mono-III, Mono-IV, I2m

#### SERVO - Motorola MC68HC05 8-bit Microcontroller

This microcontroller performs low-level CD drive control functions.
It contains XXX bytes of read-only factory-programmed program ROM
and XXX bytes of RAM.

**Data sheet:** TBA \
**Additional information:** [mc6805servo.md](mc6805servo.md) in this repository \
**CD-i boards:** Maxi-MMC, Mini-MMC, Mono-I

#### SLAVE - Motorola MC68HC05C8 8-bit Microcontroller

This microcontroller performs high-level CD drive control, CD-i pointing
device and general system control functions.
It contains 1744 bytes of read-only factory-programmed program ROM
and 176 bytes of RAM.

**Data sheet:** http://www.icdia.co.uk/docs/mc68hc05c8rg.pdf \
**Additional information:** [mc6805slave.md](mc6805slave.md) in this repository \
**CD-i boards:** Maxi-MMC, Mini-MMC, Mono-I, Mono-II

#### IKAT - Motorola MC68HC05i8 8-bit microcontroller

This microcontroller performs high-level CD drive control, CD-i pointing
device and general system control functions.
It contains 7936 bytes of read-only factory-programmed program ROM
and 224 bytes of RAM.

**Data sheet:** http://www.icdia.co.uk/docs/mc68hc05i8ai.pdf \
**Additional information:** [mc6805ikat.md](mc6805ikat.md) in this repository \
**CD-i boards:** Mono-III, Mono-IV

## NVRAM chips

TBA

## MPEG chips

#### FMA - Motorola GSC38GG307CF50 Full Motion Audio

**Data sheet:** Not available \
**DVC boards:** GMPEG, VMPEG

#### DSP - Motorola DSP56001FC33 Digital Signal Processor

**Data sheet:** http://www.icdia.co.uk/docs/dsp56001.zip \
**DVC boards:** GMPEG, VMPEG

#### FMV - Motorola GSC02UH307MZ57 Full Motion Video

**Data sheet:** Not available \
**DVC boards:** GMPEG

#### FMV - Motorola MCD251 MPEG Full Motion Video Decoder

**Data sheet:** Not available \
**Technical summary:** http://www.icdia.co.uk/docs/mcd251ts.pdf \
**DVC boards:** VMPEG

#### IMPEG - Motorola MCD270 MPEG Integrated Video and Audio Decoder

**Data sheet:** Not available \
**Technical summary:** http://www.icdia.co.uk/docs/mcd270ts.pdf \
**DVC boards:** IMPEG
