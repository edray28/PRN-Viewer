# Maintop PRN Viewer

A free, offline viewer for `.prn` files produced by **Maintop** for the **WLY3200 UV printer** (i3200 / I3200 UV" heads).

Maintop's PRN output is a raw, already-halftoned print file, not a standard image or printer-language file, so ordinary PRN viewers and converters can't open it. This tool decodes it directly and shows you what the job will print: layout, ink channels, white underbase and colour.

<img width="1919" height="916" alt="image" src="https://github.com/user-attachments/assets/d2003b5d-fe43-4216-b7fc-0b25835b2666" />


## Features

- **Opens Maintop PRN files** — drag and drop or use *Open .prn…*. Large files (45 MB+) load without trouble.
- **Correct physical size and aspect** — the print size in mm is calculated from the file's resolution, e.g. 720 × 1800 dpi.
- **Per-channel control** — turn each ink channel on or off, change its colour, and set it as *ink*, *opaque (white)* or *gloss*.
- **ICC colour preview** — built-in **3PASS, 4PASS, 6PASS and 8PASS** profiles, or load your own `.icm`. The viewer picks the matching profile from the file header.
- **Simple ink mix mode** — a quick preview with no profile, with an ink-density slider.
- **Substrate preview** — view the job on light grey, white, dark grey or black, so white ink and dark inks are easy to judge.
- **Orientation tools** — rotate 180°, mirror or flip.
- **Zoom and pan** — mouse-wheel zoom down to individual pixels.
- **Pixel readout** — hover to see the position in mm and pixels, and the drop level (0–3) of every channel at that point.
- **Save PNG** — export the preview (up to 6000 px wide).
- **Private** — everything runs in your browser. No upload, no server, no install.

## Quick start

## Using the viewer

1. Open a `.prn` file. The viewer reads the header, shows the file details in the sidebar and fits the image to the window.
2. Check the **Colour** dropdown. A profile is chosen automatically from the file header. Change it if the colours look wrong, or pick **Load .icm file…**.
3. Adjust the **Substrate** to match your material. White ink only shows against a grey or dark substrate.
4. If the print looks upside-down relative to your artwork, use **Rotate 180°** or the mirror and flip options.

Shortcut: press **F** to fit the image to the window.

## The PRN format (as understood so far)

This was worked out by analysing sample files from one printer setup. It is not official documentation.

All values are little-endian. The header is 48 bytes.

| Offset | Type | Meaning |
|-------:|------|---------|
| `0x00` | u32 | Magic, `0x5555` |
| `0x04` | u32 | Horizontal resolution (dpi), e.g. 720 |
| `0x08` | u32 | Vertical resolution (dpi), e.g. 1200 or 1800 |
| `0x0C` | u32 | Bytes per line (padded to a multiple of 4) |
| `0x10` | u32 | Number of lines |
| `0x14` | u32 | Image width in pixels |
| `0x18` | f32 | Image width in metres (width ÷ dpi × 0.0254) |
| `0x1C` | u32 | Channels per line (6) |
| `0x20` | u32 | Bits per pixel (2) |
| `0x24` | u32 | Pass value (3, 4, 6…), assumed to be the pass count |
| `0x28` | u32 | Always 1 in the samples; meaning unknown |

Pixel data starts at byte 48. For each line, the file stores one row per channel, one after another, so the total size is:

```
file size = 48 + 6 × lines × bytes-per-line
```

Each pixel is 2 bits (four per byte) holding a **drop-size level from 0 to 3**: 0 is no drop, and 1 to 3 are increasing drop sizes. The first pixel is taken from the high bits of the byte; the **LSB-first pixels** option in the viewer reverses this.

### Channel order

Inferred from the colours in the sample files:

| Channel | Ink |
|--------:|-----|
| 0 | K (black) |
| 1 | C (cyan) |
| 2 | M (magenta) |
| 3 | Y (yellow) |
| 4 | White 1 |
| 5 | White 2 (or varnish, depending on setup) |

Other presets are available in the sidebar if your printer uses a different order.

## How colour works

- **ICC mode:** the C, M, Y and K drop levels are converted to ink amounts and run through the selected CMYK→Lab profile, then converted to sRGB. White ink is treated as an opaque underbase, so colour printed over white shows as the profile intends, and colour on bare substrate is darkened to match it. The built-in profile data is the CLUT (A2B1 table) from each `.icm`, so you don't need the original files.
- **Simple mode:** each ink is multiplied onto the substrate using the channel colours you set.

The preview shows ink coverage and an estimate of colour. It isn't a substitute for a proof, because it doesn't know your media, ink lot or curing. The mapping from drop level to ink percentage is a simple curve, and the **Ink density** slider adjusts it.

## Limitations

- Built and tested against sample files from a **WLY3200 / i3200** setup only. Other printers or Maintop output modes may use a different layout. If a file is rejected, it probably isn't in this layout.
- The channel order and the meaning of the "pass value" are inferred, not confirmed.
- Bit order within a byte couldn't be confirmed from the data. It only shifts pixels within a 4-pixel group, so it's invisible except at maximum zoom.
- Maintop's `.prs`, `.mcm`, `.wpk`, `.pdv` and `.cma` files are proprietary and aren't read.
- No true `.exe` is provided. The Windows launcher is a small `.cmd` script.
- Needs a browser with `DecompressionStream` support (recent Chrome, Edge, Firefox or Safari).

## Disclaimer

This is an independent, unofficial project and is not affiliated with or endorsed by Maintop, Epson, or the printer manufacturer. Maintop, Epson and other names are trademarks of their respective owners. The file format notes come from analysing sample files and the printer's colour profiles, not from vendor documentation.

## License

MIT License

This project is licensed under the MIT License, allowing anyone to use, copy, modify, merge, publish, distribute, sublicense, and sell copies of the software, provided that the original copyright notice and permission notice are included in all copies or substantial portions of the software.
