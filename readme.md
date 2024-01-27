# Introduction

This is a module emulating the playback functionality of the Toccata sound card for the Commodore Amiga. [Toccata sound card](https://amiga.resource.cx/exp/toccata)

The module is written in SystemVerilog, and is intended to be used in the Minimig core.

It was not possible for me to get an actual Toccata sound card, or find the documentation for the sound card.
This module is implemented by reverse engineering the Toccata sound card emulation of UAE, and looking at the OpenBSD driver.
The Toccata sound card uses the AD1848 audio chip, the datasheet was used to implement the exposed registers to the zorro II bus.

## Implemented features

Because I needed 16-bit audio output, but have no use for audio input, input was not implemented in this module.

List of implemented features:

- Control of the playback features via the Zorre II tatus register.
- Status feedback via reading the Zorro II status register.
- Playback of 8-bit mono, 8-bit stereo, 16-bit mono, 16-bit stereo
- Audio volume via the AD1848 registers
- Muting of left and right audio channels via AD1848 registers
- Fake recording interrupts and data (only records scilence)
- 1kb sample buffer, with interrupt generation on half empty.
- Writing audio to the FIFO using the ZORRO II bus registers
- Configuring the playback sample rate using the AD1848 registers

Not implemented features:

- Playback of Companded Audio (Played back as normal 8-bit audio)
- Mixing in the Paula audio via the Toccata sound card
- Real audio callibration, callibration ready is faked. (Not needed for this module)

## Usage

This module needs to be hooked into the Minimig core, in the Zorro II memory space.
For this the autoconfig process needs to be configured, providing the Kickstart with the information it needs to assign a memory region (64kb wide).
In the Minimig core this means add an extra entry to handle the Toccata sound card.

The configuration I use for the Toccata auto configuration ROM is:

```verilog
// Toccata sound card

ram[sndbase+'h0] = 4'b1100; // Zorro-II card, no link, no ROM
ram[sndbase+'h2/2] = 4'b0001; // Next board not related, size 'h64k
// Inverted from here on
ram[sndbase+'h6/2] = 4'b0011; // Lower byte product number

ram[ethbase+'ha/2] = 4'b1101;   // logical size 64k

ram[sndbase+'h10/2] = 4'b1011; // Manufacturer ID: 0x4754
ram[sndbase+'h12/2] = 4'b1000;
ram[sndbase+'h14/2] = 4'b1010;
ram[sndbase+'h16/2] = 4'b1011;
```

When the Toccata sound card is the first Zorro II IO card it ends up on memory address $e90000, after autoconfiguration is completed.

For a full integration example take a look at the [OpenAARS Minimig](https://github.com/ranzbak/MinimigAGA_TC64/tree/v5.0/rtl/openaars/toccata) project.

## Testing

Because this module is part of my Minimig project, I was able to test the module with :

- AHI sound drivers
- Eagle player 2.06 both AHI and Toccata amplifier.
- Hippo player via the AHI sound driver
- Octamed sound studio using the Toccata sound 16bit ++ output.

The programs above played back audio correctly, although some weird behaviour was observed when applying effects to the audio streams, not sure if this because of software errors or bugs in this module.

## Contributing

If there are things that are missing, should be improved or fixed, please create a pull request.

## License

BSD-2-Clause "Simplified" License

- Also if you like this project, and you run into me at some nerd event buy me a coffee / Club mate :-)
