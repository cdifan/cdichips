# MC68HC05 8-bit Microcontroller (SERVO)

In CD-i players the SERVO microcontroller performs low-level CD drive control
functions. It contains XXX bytes of read-only factory-programmed program ROM and
XXX bytes of RAM.

The data sheet of the base MC68HC05 chip is publicly available but interface
documentation for the SERVO program is not.

The SERVO chip is used in the CD-i [Maxi-MMC], [Mini-MMC] and [Mono-I]
mainboard generations.

The SERVO is not directly connected to the 68000 main CPU. It connects to
the DSP via the X-bus and to the SLAVE via the SPI bus.

[Maxi-MMC]: https://www.cdiemu.org/players/
[Mini-MMC]: https://www.cdiemu.org/players/
[Mono-I]: https://www.cdiemu.org/players/
