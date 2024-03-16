# MC68HC05C8 8-bit Microcontroller (SLAVE)

In CD-i players the SLAVE microcontroller performs high-level CD drive control,
CD-i pointing device, front LED display and general system control functions. It
contains 7744 bytes of read-only factory-programmed user ROM and 176 bytes of
RAM.

The data sheet of the base MC68HC05C8 chip is publicly available but interface
documentation for the SLAVE program is not. This document is an attempt to
describe the SLAVE functions with enough level of detail so that they can be
emulated in software.

The information in this document has mostly been determined by reverse
engineering CD-i drivers for the CD-i [Maxi-MMC], [Mini-MMC], [Mono-I] and [Mono-II]
mainboard generations that use the SLAVE microcontroller.

Because of this, the exact behaviour of many functions is unclear. In some
of these cases the emulation used by [CD-i Emulator] is described.

## Memory map

The following memory map has been derived from the service manuals and reverse
engineering.

All memory addresses are hexadecimal and relative to the base address of the
SLAVE. In CD-i players this is either 00200000 (Mini-MMC, Maxi-MMC) or 00310000
(Mono-I, Mono-II) so that for example on a Mini-MMC board Channel A is located at
main CPU address 00200001.

The SLAVE has an 8-bit connection to the lower part of the main 68000 CPU data
bus and does not have an A0 input pin, so all of the memory addresses below are
odd and the registers must be accessed with byte read or write cycles.

Address | Register | Description
--- | --- | ---
1 | AR | Channel A Data Register
3 | BR | Channel B Data Register
5 | CR | Channel C Data Register
7 | DR | Channel D Data Register

The register names are loosely inspired by the MC68HC05i8 register names.

Note: Channels A-D have no relation to MC68HC05 ports PA-PD.

## Concept of operation

Each channel accepts a number of command messages and can return a number of
response messages. Some commands will always return response messages, others
will not; response messages can also be returned aynchronously driven by
external input, e.g. from pointing devices.

The length of each message is determined by its first byte. Both commands and
responses are channel-specific, e.g. first byte 80 on channel B specifies a different
command than first byte 80 on channel D.

The SLAVE will assert its level-based interrupt output whenever response message
bytes are available on any channel.

The SLAVE will delay assertion of it's DTACK (Data Transfer ACKnowledge) output
to briefly halt the main 68000 CPU when needed; the exact timing dependencies
are not currently known.

## Commands

### Channel A

#### Enable pointer input
Channel | Hex | Binary | Name
--- | --- | --- | ---
A | 83 | 1000 0011 | `Enable pointer input`

Enables `Pointer state` response messages on channel A.

**Response:** `Pointer state` on channel A

#### Disable pointer input
Channel | Hex | Binary | Name
--- | --- | --- | ---
A | 84 | 1000 0100 | `Disable pointer input`

Disables `Pointer state` response messages on channel A.

**Response:** None

#### Set pointer position
Channel | Hex | Binary | Name
--- | --- | --- | ---
A | Cx xx xx ... Fx xx xx | 11*aa aaaa* 0*bbb cccc* 0*ddd dddd* | `Set pointer position`

Sets the current pointer position to x = *bbbddddddd*, y = *aaaaaacccc*.

The x and y values correspond to high-res screen positions.

**Response:** None

TBA: more channel A commands

### Channel B

TBA: channel B commands

### Channel C

#### Reset main cpu
Channel | Hex | Binary | Name
--- | --- | --- | ---
A | 8A | 1000 1010 | `Reset main cpu`

Resets the main 68000 CPU by asserting its RESET input.

**Response:** None, main CPU is reset

TBA: more channel C commands

### Channel D

TBA: channel C commands

## Responses

### Channel A

#### Pointer state
Channel | Hex | Binary | Name
--- | --- | --- | ---
A | 0x xx xx xx ... 3x xx xx xx | 00*ab cddd* 0*eee eeee* 0000 0*fff* 0*ggg gggg* | `Pointer state`

Reports the current pointer state: \
*a* = set when button 2 pressed \
*b* = set when button 1 pressed \
*c* = set when pointer active \
*dddeeeeeee* = x position \
*fffggggggg* = y position

The x and y values correspond to high-res screen positions.

TBA: more channel A responses
 
### Channel B

TBA: channel B responses

### Channel C

TBA: channel C responses

### Channel D

TBA: channel D responses

[CD-i Emulator]: https://www.cdiemu.org/cdiemu/
[Maxi-MMC]: https://www.cdiemu.org/players/
[Mini-MMC]: https://www.cdiemu.org/players/
[Mono-I]: https://www.cdiemu.org/players/
[Mono-II]: https://www.cdiemu.org/players/

