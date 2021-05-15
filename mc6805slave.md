# MC68HC05 8-bit Microcontroller (SLAVE)

In CD-i players the SLAVE microcontroller performs high-level CD drive control,
CD-i pointing device, front LED display and general system control functions. It
contains XXX bytes of read-only factory-programmed program ROM and XXX bytes of
RAM.

The data sheet of the base MC68HC05 chip is publicly available but interface
documentation for the SLAVE program is not. This document is an attempt to
describe the SLAVE functions with enough level of detail so that they can be
emulated in software.

The information in this document has mostly been determined by reverse
engineering CD-i drivers for the CD-i Maxi-MMC, Mini-MMC, Mono-I and Mono-II
mainboard generations that use the SLAVE microcontroller.

Because of this, the exact behaviour of many functions is unclear. In some
of these cases the emulation used by [CD-i Emulator] is described.

## Memory map

The following memory map has been derived from the service manuals and reverse
engineering.

All memory addresses are hexadecimal and relative to the base address of the
SLAVE. In CD-i players this is either 00200000 (Mini-MMC, Maxi-MMC) or 00310000
(Mono-I, Mono-II) so that for example on a Mini-MMC board Port A is located at
main CPU address 00200001.

The SLAVE has an 8-bit connection to the lower part of the main 68000 CPU data
bus and does not have an A0 input pin, so all of the memory addresses below are
odd and the registers must be accessed with byte read or write cycles.

Address | Register | Description
--- | --- | ---
1 | PA | Port A
3 | PB | Port B
5 | PC | Port C
7 | PD | Port D

Note: Ports A-D as documented here are not the MC68HC05 ports of the same name.

## Concept of operation

Each port accepts a number of command messages and can return a number of
response messages. Some commands will always return response messages, others
will not; response messages can also be returned aynchronously driven by
external input, e.g. from pointing devices.

The length of each message is determined by its first byte. Both commands and
responses are port-specific, e.g. first byte 80 on port B specifies a different
command than first byte 80 on port D.

The SLAVE will assert its level-based interrupt output whenever response message
bytes are available on any port.

The SLAVE will delay assertion of it's DTACK (Data Transfer ACKnowledge) output
to briefly halt the main 68000 CPU when needed; the exact timing dependencies
are not currently known.

## Commands

### Port A

#### Enable pointer input
Port | Hex | Binary | Name
--- | --- | --- | ---
A | 83 | 1000 0011 | `Enable pointer input`

Enables `Pointer state` response messages on port A.

**Response:** `Pointer state` on port A

#### Disable pointer input
Port | Hex | Binary | Name
--- | --- | --- | ---
A | 84 | 1000 0100 | `Disable pointer input`

Disables `Pointer state` response messages on port A.

**Response:** None

#### Set pointer position
Port | Hex | Binary | Name
--- | --- | --- | ---
A | Cx xx xx ... Fx xx xx | 11*aa aaaa* 0*bbb cccc* 0*ddd dddd* | `Set pointer position`

Sets the current pointer position to x = *bbbddddddd*, y = *aaaaaacccc*.

The x and y values correspond to high-res screen positions.

**Response:** None

TBA: more port A commands

### Port B

TBA: port B commands

### Port C

#### Reset main cpu
Port | Hex | Binary | Name
--- | --- | --- | ---
A | 8A | 1000 1010 | `Reset main cpu`

Resets the main 68000 CPU by asserting its RESET input.

**Response:** None, main CPU is reset

TBA: more port C commands

### Port D

TBA: port C commands

## Responses

### Port A

#### Pointer state
Port | Hex | Binary | Name
--- | --- | --- | ---
A | 0x xx xx xx ... 3x xx xx xx | 00*ab cddd* 0*eee eeee* 0000 0*fff* 0*ggg gggg* | `Pointer state`

Reports the current pointer state: \
*a* = set when button 2 pressed \
*b* = set when button 1 pressed \
*c* = set when pointer active \
*dddeeeeeee* = x position \
*fffggggggg* = y position

The x and y values correspond to high-res screen positions.

TBA: more port A responses
 
### Port B

TBA: port B responses

### Port C

TBA: port C responses

### Port D

TBA: port D responses

[CD-i Emulator]: http://www.cdiemu.org/cdiemu/
