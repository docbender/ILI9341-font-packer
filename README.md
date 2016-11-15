# ILI9341 Font Packer v1.0

## What does it do?
Using a combination of a [Teensy](https://www.pjrc.com/teensy/) microcontroller, and an
[SPI flash memory IC](http://uk.farnell.com/spansion/s25fl216k0pmfi011/memory-flash-16mb-3v-spi-8soic/dp/2327997), you
can display (almost) any font on an
[ILI9341-compatible](https://www.adafruit.com/products/2478) LCD display, using Paul Stoffregen's
[ILI9341](https://github.com/PaulStoffregen/ILI9341_t3) and [SerialFlash](https://github.com/PaulStoffregen/SerialFlash)
libraries for the Teensy.

Tested with Python 3.5. 2.x will possibly baulk at all the byte and string twiddling, but that shouldn't be hard to fix 
for those with the inclination.

Don't forget to check the license of any font you use. Use of this software grants you no rights in that regard.

## What's inside?
The main application is `font_packer.py`:

    ILI9341 Font Packer
    
    Converts TrueType fonts to a compact binary bitmap format for use with Paul Stoffregen's ILI9341 library for the Teensy.
    
    Usage:
        font_packer.py --height=<pixels> [--range=<range-string>] [--placeholder=<ascii-code>] [--packed|--code|--smoke]
                       <font-file> [<output>]
        font_packer.py -h | --help
        font_packer.py --version
    
    Options:
        -h --help                   Show this screen.
        --version                   Show version.
        --height=<pixels>           The maximum height of the glyphs.
        --range=<range-string>      A range of ASCII codes to generate glyphs for [default: 32-126].
        --placeholder=<ascii-code>  A placeholder character to use for ones not in the set. Defaults to the first one.
        --packed                    Sends the packed binary bitmap font to output or stdout.
        --code                      Generates C structs to output or stdout.
        --smoke                     Smoke proof. Displays to output or stdout the bitmaps of each character as asterisks.
        output                      Output file name or stdout if not supplied.

It generates a bitmap font from a TrueType or OpenType font (or any other font supported by
[FreeType2](https://www.freetype.org/freetype2/docs/ft2faq.html))

`--height` The height of the font in pixels. This isn't necessarily the actual height of the final bitmap due to space 
for accents and so on; the bitmap you see on your screen will be smaller. But it is what FreeType2 uses, and it is
comparable to using non-antialiased type with no hinting in GIMP.

`--range` The range, in ASCII, of the glyphs you want to produce. This can be a disjoint range, such as `32,33,65-90`.
Gaps in the range will be replaced by the first glyph (space in this example), or by the glyph specified in the
`--placeholder` parameter. These gaps won't take up any space in the actual bitmap array, they will just be an entry in 
the index array that points to the first glyph. Characters outside the minimum and maximum of the range are just ignored
by the ILI9341 font rendering functions.

`--placeholder` The glyph to use for missing characters within the minimum and maximum range specified with the 
`--range` parameter. This is in ASCII and will appear however FreeType2 decides to render it, so using 0 will generally
produce a hollow rectangle glyph.

`--packed` Flag to output the font as a Teensy ILI9341 font rendering compatible structure. This can be stored on a
flash chip (or SD card) and loaded at runtime. This is the default.

`--code` Flag to output C code in the same format as the example fonts provided by the display library. This enables the
font to be compiled directly into the executable.

`--smoke` Produces a "smoke proof" ASCII art of each glyph using asterisks and spaces, similar to the Unix `banner`
command. Useful for checking which glyphs will get output, and their size in pixels.

`output` File name to direct the output to. If this is missing the output gets sent to stdout.

`filename` The file name of a FreeType2 recognised font format. Only TTFs and OTFs have been tested, but others should
work as well.

Aside from this there is `rawfile-uploader.py` which is a modification of the same file found in the `extras` directory
of the [SerialFlash](https://github.com/PaulStoffregen/SerialFlash) repository. This is used to upload a file to a flash
chip attached to a Teensy (and probably Arduino-alikes as well), running the modified `CopyFromSerial.ino` sketch that 
can be found here.

## How do I use it?

`python font_packer.py --height=36 --range=43,45,46,48-57 --placeholder=0 --packed arial.ttf numbers.bin`

Will output just the glyphs `+-.0123456789` and replace `,` and `/` with a hollow rectangle.

Assuming the `CopyFileFromSerial.ino` sketch is running on the Teensy, you can upload the file like this:

`python font_packer.py /dev/ttyACM0 numbers.bin`

If all goes well, the Teensy's LED will go out and stay off. If it flashes after upload, then there has been an error.
The number of flashes indicate the error, which you can find in the code.

Compile and upload the `examples/test.cpp` file onto your Teensy, and you should see some black text on a yellow
background.

A few things to note. Because you're loading the font into RAM, you need to malloc() enough space before loading it.
Once loaded into memory, you need to fix up the two pointers that point to the index and the data. Also, the font
rendering code expects a reference to a `ILI9341_t3_font_t` struct, so don't forget to dereference the pointer to your 
font you just allocated.

## To do
The ILI9341 font library only supports ASCII at this time, so does this font packer. I'd like to at least support
different code pages, and ultimately some sort of unicode to single-byte mapping.

The rendering library also only supports 1-bit fonts. It would be good to provide at least 2-bit anti-aliased fonts.

---
Copyright (c) 2016 GeoSpark

Portions of this code are released under the MIT License (MIT)

See the LICENSE file, or visit http://opensource.org/licenses/MIT