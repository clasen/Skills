# Sharp Metadata and Stats Reference

## metadata([callback])

Fast access to image header info without decoding pixels. Returns a Promise.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `format` | string | Decoder name: `jpeg`, `png`, `webp`, `gif`, `svg`, etc. |
| `size` | number | Total bytes (Stream/Buffer input only) |
| `width` | number | Pixels wide (ignores EXIF orientation) |
| `height` | number | Pixels high (ignores EXIF orientation) |
| `space` | string | Colour space: `srgb`, `rgb`, `cmyk`, `lab`, `b-w`, etc. |
| `channels` | number | Band count (3 for sRGB, 4 for CMYK) |
| `depth` | string | Pixel depth: `uchar`, `char`, `ushort`, `float`, etc. |
| `density` | number | DPI if present |
| `chromaSubsampling` | string | JPEG chroma: `4:2:0` or `4:4:4` |
| `isProgressive` | boolean | Progressive/interlaced scan |
| `isPalette` | boolean | Palette-based (GIF, PNG) |
| `bitsPerSample` | number | Bits per sample per channel |
| `pages` | number | Page/frame count (TIFF, HEIF, PDF, animated GIF/WebP) |
| `pageHeight` | number | Height per page in multi-page images |
| `loop` | number | Animation loop count (0 = infinite) |
| `delay` | number[] | Delay in ms between animation frames |
| `pagePrimary` | number | Primary page in HEIF |
| `levels` | Array | Multi-level details (OpenSlide) |
| `subifds` | number | Sub IFDs in OME-TIFF |
| `background` | | Default background (PNG bKGD, GIF) |
| `compression` | string | HEIF encoder: `av1` (AVIF) or `hevc` (HEIC) |
| `resolutionUnit` | string | `'inch'` or `'cm'` |
| `hasProfile` | boolean | ICC profile present |
| `hasAlpha` | boolean | Alpha channel present |
| `orientation` | number | EXIF Orientation (1-8) |
| `exif` | Buffer | Raw EXIF data |
| `icc` | Buffer | Raw ICC profile data |
| `iptc` | Buffer | Raw IPTC data |
| `xmp` | Buffer | Raw XMP data |
| `xmpAsString` | string | XMP as UTF-8 (if valid) |
| `tifftagPhotoshop` | Buffer | Raw TIFFTAG_PHOTOSHOP |
| `formatMagick` | string | Format for *magick-loaded images |
| `comments` | Array | PNG text blocks (keyword/text pairs) |
| `autoOrient` | Object | `{ width, height }` accounting for EXIF orientation |

### EXIF Orientation Caveat

`width`/`height` are raw pixel dimensions, not visual. Use `autoOrient` for visual dimensions:

```js
const { autoOrient } = await sharp(input).metadata();
const { width, height } = autoOrient; // visually-correct dimensions
```

### Read-Then-Transform Pattern

```js
const image = sharp(input);
const { width } = await image.metadata();
const result = await image.resize(Math.round(width / 2)).webp().toBuffer();
```

## stats([callback])

Pixel-derived statistics. More expensive than `metadata()` — reads pixel data. Returns a Promise.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `channels` | Array | Per-channel stats (see below) |
| `isOpaque` | boolean | No alpha or all pixels fully opaque |
| `entropy` | number | Greyscale entropy estimate |
| `sharpness` | number | Greyscale sharpness (Laplacian std dev) |
| `dominant` | `{ r, g, b }` | Most dominant sRGB colour (4096-bin histogram) |

### Per-Channel Stats

Each entry: `min`, `max`, `sum`, `squaresSum`, `mean`, `stdev`, `minX`, `minY`, `maxX`, `maxY`

### Stats on Processed Output

Stats read from the original input. To get stats after operations, write to buffer first:

```js
const part = await sharp(input).extract(region).toBuffer();
const stats = await sharp(part).stats();
```
