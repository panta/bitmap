# Bitmap API

A C library for manipulating bitmap graphics in memory and on disk.

```c
#include <stdio.h>
#include "bmp.h"

int main(int argc, char *argv[]) {
    Bitmap *b = bm_create(128,128);

    bm_set_color(b, bm_atoi("white"));
    bm_puts(b, 30, 60, "Hello World");

    bm_save(b, "out.gif");
    bm_free(b);
    return 0;
}
```

The code is licensed under the terms of the MIT License. See the file
[LICENSE](LICENSE) for details.

Features:
* Supported formats:
  * Windows BMP: loads 4-, 8- and 24-bit uncompressed BMP files, saves as 24-bit.
  * GIF, PCX and TGA files can be loaded and saved without
    third-party dependencies.
  * PNG through [libpng](http://www.libpng.org)
    * support for palettized images is incomplete.
  * JPEG through [libjpeg](http://www.ijg.org/)
  * Alternatively, PNG and JPEG files can be loaded through the Sean Barrett's
    [stb_image][] image loader library. [stb_image][] supports a couple of
    additional formats, such as PSD, PIC and PNM binary formats. See
    `bm_load_stb()` for more information.
    A copy of `stb_image.h` has been placed in </3rd-party/stb_image.h>.
* Supports manipulation of OpenGL textures, [SDL](https://www.libsdl.org/)
  surfaces, [GDI](https://en.wikipedia.org/wiki/Graphics_Device_Interface)
  contexts. See the `bm_bind()` function. It can also serve as a back end for
  [Cairo graphics](https://cairographics.org)
* Support for SDL2's `SDL_RWops` file manipulation routines for loading images
  in the supported formats.
* Primitives: Lines, circles, elipses, bezier curves, polygons, etc.
  * Clipping rectangles are obeyed.
* Floodfill and filled primitives.
* Image resizing: nearest neighbour, bilinear and bicubic.
* Blitting, blitting with color masks, scaled and rotated blitting.
* Text rendering: [FreeType][freetype] support and built-in _8-bit_ style
  raster fonts, and support for [SFont][sfont] and [GrafX2-style][grafx2] fonts.
* Filtering with kernels and smoothing.
* [CSS-style](http://en.wikipedia.org/wiki/Web_colors) colors.
* Loading images from [XBM](https://en.wikipedia.org/wiki/X_BitMap) and [X
  PixMap](https://en.wikipedia.org/wiki/X_PixMap) data.
* Cross platform. It's been known to compile under Windows (MinGW), Linux,
  Android and Web via [Emscripten][emscripten].

The `fonts/` directory contains some 8-bit style bitmap fonts in XBM format.

[freetype]: https://www.freetype.org/
[emscripten]: http://kripken.github.io/emscripten-site/
[stb_image]: https://github.com/nothings/stb/blob/master/stb_image.h
[sfont]: http://www.linux-games.com/sfont/
[grafx2]: https://en.wikipedia.org/wiki/GrafX2

## Getting Started

Copy `bmp.c` and `bmp.h` to your project directory, and add `bmp.c` to your
list of files to compile.

To enable PNG support you need [zlib][] and [libpng][] (the development
versions) installed. If you are using GCC, do the following:

* Use these flags ``-DUSEPNG `libpng-config --cflags` `` when compiling.
* Add `` `libpng-config --ldflags` -lz `` when linking.

Other compilers might have diffent flags. See the [libpng][] documentation for
your platform.

Likewise, to enable JPEG support, you need [libjpeg][] installed and specify
the `-DUSEJPG` command line option when compiling and add `-ljpeg` to your
linker options.

To use [stb_image][], put the `stb_image.h` file in the same directory as
`bmp.c`, and add `-DUSESTB` to your compiler flags.

Use `bm_create()` to create `Bitmap` objects and `bm_free()` to destroy them.

`bm_bind()` can be used to wrap a Bitmap object around an existing buffer of
bytes, such as OpenGL textures and SDL surfaces.

The `Makefile` generates HTML documentation from `bmp.h` through the
`[d.awk][dawk]` script. Type `make docs` to create the documentation.

[dawk]: https://github.com/wernsey/d.awk
[zlib]: https://www.zlib.net
[libpng]: http://www.libpng.org
[libjpeg]: http://www.ijg.org/

## Cross-compiling a Win32 DLL

From OSX:

```bash
$ brew install mingw-w64
$ export PREFIX=/tmp/w32
$ export PKG_CONFIG_PATH="${PREFIX}/lib/pkgconfig"
$ make -f Makefile.mingw32 dll
```

### Manual procedure to cross-compile and install libjpeg

**NOTE**: here for reference purposes only, this is automatically done by the Makefile.mingw32

```bash
$ wget http://www.ijg.org/files/jpegsrc.v9d.tar.gz
$ tar xvfz jpegsrc.v9d.tar.gz
$ cd jpeg-9d
$ export PREFIX=/tmp/w32
$ export MINGW32=/usr/local/Cellar/mingw-w64/6.0.0_2/toolchain-i686/i686-w64-mingw32
$ CC=i686-w64-mingw32-gcc DLLTOOL=i686-w64-mingw32-dlltool ./configure --prefix="${PREFIX}" --host=i686-w64-mingw32 --with-sysroot="${MINGW32}" --enable-shared=yes --enable-static=no
$ make
$ make install
```


## Additional Utilities

* The `fonts/` directory contains some 8&times;8 bitmap fonts in XBM format
  that can be loaded via the `bm_make_xbm_font()` function.
  * Most of the fonts are based on 80's home computers, drawn from the examples
    at http://damieng.com/blog/2011/02/20/typography-in-8-bits-system-fonts
  * The `INSTRUCTIONS` file contains information for using them.
  * The `tomthumb.c` file contains a 4&times;6 font based on the "Tom Thumb"
    monospace font at http://robey.lag.net/2010/01/23/tiny-monospace-font.html
* The `ftypefont/` directory contains a wrapper for
  [FreeType](http://www.freetype.org/) to allow rendering of freetype-supported
  fonts on `Bitmap` structures.
* The `misc/` directory contains
  * `gif.c` and `gif.h` - code for programmatically manipulating animated GIFs.
    It originates from the file I used to develop the GIF encoder/decoder
    originally.
  * `cairoback.c` - A demo of how the bitmap objects can be used as a back-end
    for the [Cairo graphics library](https://cairographics.org)
  * The `palette/` subdirectory contains a utility for generating palettes and
    converting images to those palettes.
  * `pbm.c` and `xpm.c`: Samples on how to use the module with PBM and XPM file
    formats. `to_xbm.c` contains a function that can output a bitmap as a XBM.
  * The `kmeans/` subdirectory contains a program that uses [K-means
    clustering](https://en.wikipedia.org/wiki/K-means_clustering) to identify
    the dominant colors in an image.
  * `imgdup.c` is a program that scans directories for duplicate images using
    the _dHash_ algorithm.
  * `nanosvg.c` is an example of how to use the library with Mikko Mononen's [NanoSVG][]
    library to load SVG files.
  * `bm_microui.c` contains functions to render `rxi`'s [microui][] graphical
    user interfaces to a `Bitmap` structure.

[NanoSVG]: https://github.com/memononen/nanosvg
[microui]: https://github.com/rxi/microui

## References

* [BMP file format](http://en.wikipedia.org/wiki/BMP_file_format) on Wikipedia
* [Bresenham's line
  algorithm](http://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm) on
  Wikipedia
* <http://members.chello.at/~easyfilter/bresenham.html>
* [Flood fill](http://en.wikipedia.org/wiki/Flood_fill) on Wikipedia
* [Midpoint circle
  algorithm](http://en.wikipedia.org/wiki/Midpoint_circle_algorithm) on
  Wikipedia
* <http://web.archive.org/web/20110706093850/http://free.pages.at/easyfilter/bresenham.html>
* [Typography in 8 bits: System
  fonts](http://damieng.com/blog/2011/02/20/typography-in-8-bits-system-fonts)
* [GIF89a specification](http://www.w3.org/Graphics/GIF/spec-gif89a.txt)
* Nelson, M.R. : "LZW Data Compression", Dr. Dobb's Journal, October 1989.
* <http://commandlinefanatic.com/cgi-bin/showarticle.cgi?article=art011>
* [What's In A
  GIF](http://www.matthewflickinger.com/lab/whatsinagif/index.html) by Matthew
  Flickinger
* <http://web.archive.org/web/20100206055706/http://www.qzx.com/pc-gpe/pcx.txt>
* <http://www.shikadi.net/moddingwiki/PCX_Format>
* [Truevision TGA](https://en.wikipedia.org/wiki/Truevision_TGA) on Wikipedia
* <http://paulbourke.net/dataformats/tga/>
* <http://www.ludorg.net/amnesia/TGA_File_Format_Spec.html>
* [X PixMap](https://en.wikipedia.org/wiki/X_PixMap) on Wikipedia
* [A simple libpng example program](http://zarb.org/~gc/html/libpng.html)
* <http://www.fileformat.info/format/xpm/egff.htm>
* "Fast Bitmap Rotation and Scaling" By Steven Mortimer, Dr Dobbs' Journal,
  July 01, 2001  \
  <http://www.drdobbs.com/architecture-and-design/fast-bitmap-rotation-and-scaling/184416337>
* <http://www.efg2.com/Lab/ImageProcessing/RotateScanline.htm>
* [Image Filtering](http://lodev.org/cgtutor/filtering.html) in _Lode's Computer Graphics Tutorial_
* [Grayscale](https://en.wikipedia.org/wiki/Grayscale) on Wikipedia
* [Xiaolin Wu's line algorithm](https://en.wikipedia.org/wiki/Xiaolin_Wu%27s_line_algorithm) on Wikipedia.
* Michael Abrash's article in the June 1992 issue of Dr Dobbs' Journal.
* The `README` file in the [SFont][sfont] distribution discusses the details.
* [This forum post](https://groups.google.com/forum/#!topic/grafx2/EQJCZDvFNfk) contains a discussion of the GrafX2 format
* Here are some SFont/GrafX2 font resources:
  * <https://opengameart.org/content/a-package-of-8-bit-fonts-for-grafx2-and-linux>
  * <https://opengameart.org/content/new-original-grafx2-font-collection>
  * <https://opengameart.org/content/the-collection-of-8-bit-fonts-for-grafx2-r2>
  * There is an outstanding bug where 8-bit PNGs cannot be read at the moment,
    so it's best to convert these to some other format first.

[sfont]: http://www.linux-games.com/sfont/