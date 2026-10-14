# PCB Quality Control Scanner

A command-line tool in C that inspects images of printed circuit boards (PCBs). It finds components using template matching, then maps which components are electrically connected by following the copper traces in the image.

Built as a first-year university programming project (USyd). It uses only the C standard library.

## Features

- **Template viewer (`t`)**: decodes a packed binary component library and prints any 32×32 template as ASCII art.
- **Component detection (`l`)**: thresholds a BMP image and scans it for exact matches against every template. It reports each component's type and position, sorted bottom-to-top and then left-to-right.
- **Connection mapping (`c`)**: treats the thresholded image as a graph, where bright pixels are nodes linked to their 4-connected neighbours. It runs a graph search from each component's boundary to find which components share a direct trace.

## How it works

1. **BMP parsing**: reads the file header and the DIB header to get the image dimensions and pixel data offset, then loads the RGB pixel data.
2. **Thresholding**: a pixel becomes `1` if its average RGB value is above 127, and `0` otherwise.
3. **Template decoding**: the library file holds a 1-byte count followed by 128 bytes per template. Each template is bit-packed, with 8 pixels per byte, 4 bytes per row and 32 rows. These bits are unpacked into 32×32 binary arrays.
4. **Template matching**: slides each template over every valid position in the image. A position counts as a match only if all 1,024 pixels agree.
5. **Trace connectivity**: starts from the outer edge of each component's 32×32 region and searches through connected trace pixels. Component regions themselves are excluded from the search, so connections are only counted when a trace runs directly between two components. A connection through a third component does not count.

## Project structure

```
├── include/        # Header files
├── src/
│   ├── main.c      # Argument parsing and mode dispatch
│   ├── bitmap.c    # BMP file reading/writing (provided scaffold)
│   ├── t_mode.c    # Template decoding and display
│   ├── l_mode.c    # Thresholding and template matching
│   ├── c_mode.c    # Trace graph search and connection mapping
│   └── utilities.c # Shared helpers
├── images/         # Sample PCB images
├── templates/      # Sample component library
└── makefile
```

## Build and run

```bash
make
./pcb_check <mode> <template_file> <index | image_file>
```

| Mode | Third argument | Output |
|------|----------------|--------|
| `t`  | template index | ASCII view of that template |
| `l`  | PCB image (.bmp) | List of detected components |
| `c`  | PCB image (.bmp) | Components plus their connections |

### Example

```
$ ./pcb_check l templates/temp001.bin images/pcb001.bmp
Found 3 components:
type: 4, row: 61, column: 99
type: 0, row: 206, column: 78
type: 2, row: 227, column: 189
```

The program handles invalid input with clear messages: a wrong argument count, an unknown mode, a missing or unreadable template file, or a template index that is out of range.

## Notes

- The BMP read/write functions (`bitmap.c`/`bitmap.h`) and the sample images and templates were provided as part of the course scaffold. Everything else is my own work.
- Matching is exact pixel-for-pixel, which works for clean synthetic images. Real-world images would need a tolerance-based method, such as normalised cross-correlation.
