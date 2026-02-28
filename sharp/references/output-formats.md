# Sharp Output Formats Reference

## Output Destinations

### toFile(path)

Write to file. Format inferred from extension (JPEG, PNG, WebP, AVIF, TIFF, GIF, DZI, V). Caller must ensure directory exists.

Returns `{ format, size, width, height, channels, premultiplied }`. Crop strategies also return `cropOffsetLeft`, `cropOffsetTop`. Attention strategy adds `attentionX`, `attentionY`. Animated output includes `pageHeight`, `pages`.

### toBuffer([options])

Write to Buffer. Default format matches input (SVG becomes PNG).

`options.resolveWithObject: true` returns `{ data, info }` instead of just data.

```js
const { data, info } = await sharp(input).png().toBuffer({ resolveWithObject: true });
```

### toFormat(format, options)

Force output format by name string or `{ id: 'format' }` object.

## Format Options

### jpeg([options])

| Option | Default | Description |
|--------|---------|-------------|
| `quality` | 80 | 1-100 |
| `progressive` | `false` | Progressive scan |
| `chromaSubsampling` | `'4:2:0'` | `'4:4:4'` for max quality |
| `mozjpeg` | `false` | mozjpeg defaults (trellis, overshoot deringing, optimised scans, quant table 3) |
| `optimiseCoding` | `true` | Optimise Huffman tables |
| `trellisQuantisation` | `false` | Trellis quantisation |
| `overshootDeringing` | `false` | Overshoot deringing |
| `optimiseScans` | `false` | Optimise progressive scans (forces progressive) |
| `quantisationTable` | 0 | Quantization table 0-8 |
| `force` | `true` | Force JPEG output |

```js
await sharp(input).jpeg({ quality: 100, chromaSubsampling: '4:4:4' }).toBuffer();
await sharp(input).jpeg({ mozjpeg: true }).toBuffer();
```

### png([options])

| Option | Default | Description |
|--------|---------|-------------|
| `progressive` | `false` | Interlace scan |
| `compressionLevel` | 6 | zlib 0 (fastest) to 9 (smallest) |
| `adaptiveFiltering` | `false` | Adaptive row filtering |
| `palette` | `false` | Indexed PNG with alpha support |
| `quality` | 100 | Min colours for target quality (enables palette) |
| `effort` | 7 | CPU effort 1-10 (enables palette) |
| `colours`/`colors` | 256 | Max palette entries (enables palette) |
| `dither` | 1.0 | Floyd-Steinberg error diffusion (enables palette) |
| `force` | `true` | Force PNG output |

```js
await sharp(input).png({ palette: true }).toBuffer();
await sharp(input).toColourspace('rgb16').png().toBuffer(); // 16bpp
```

### webp([options])

| Option | Default | Description |
|--------|---------|-------------|
| `quality` | 80 | 1-100 |
| `alphaQuality` | 100 | Alpha quality 0-100 |
| `lossless` | `false` | Lossless mode |
| `nearLossless` | `false` | Near-lossless mode |
| `smartSubsample` | `false` | High quality chroma subsampling |
| `smartDeblock` | `false` | Auto deblocking filter (slow) |
| `preset` | `'default'` | `'default'`, `'photo'`, `'picture'`, `'drawing'`, `'icon'`, `'text'` |
| `effort` | 4 | CPU effort 0-6 |
| `loop` | 0 | Animation iterations (0 = infinite) |
| `delay` | | Frame delay(s) in ms |
| `minSize` | `false` | No key frames for smaller files (slow) |
| `mixed` | `false` | Mix lossy/lossless frames (slow) |
| `force` | `true` | Force WebP |

```js
await sharp(input).webp({ lossless: true }).toBuffer();
await sharp(input, { animated: true }).webp({ effort: 6 }).toBuffer();
```

### gif([options])

| Option | Default | Description |
|--------|---------|-------------|
| `reuse` | `true` | Re-use existing palette |
| `progressive` | `false` | Interlace scan |
| `colours`/`colors` | 256 | Max palette entries 2-256 (includes transparency) |
| `effort` | 7 | CPU effort 1-10 |
| `dither` | 1.0 | Floyd-Steinberg diffusion 0-1 |
| `interFrameMaxError` | 0 | Inter-frame transparency error 0-32 (0 = lossless) |
| `interPaletteMaxError` | 3 | Inter-palette reuse error 0-256 |
| `keepDuplicateFrames` | `false` | Keep duplicate frames |
| `loop` | 0 | Animation iterations |
| `delay` | | Frame delay(s) in ms |
| `force` | `true` | Force GIF |

```js
await sharp('in.gif', { animated: true }).gif({ interFrameMaxError: 8 }).toFile('optim.gif');
await sharp('in.gif', { animated: true }).resize(128, 128).gif({ dither: 0 }).toBuffer();
```

### avif([options])

No image sequences. Prebuilt binaries: 8-bit only. Experimental on Windows ARM64 (needs ARM64v8.4+).

| Option | Default | Description |
|--------|---------|-------------|
| `quality` | 50 | 1-100 |
| `lossless` | `false` | Lossless |
| `effort` | 4 | CPU effort 0-9 |
| `chromaSubsampling` | `'4:4:4'` | `'4:2:0'` for chroma subsampling |
| `bitdepth` | 8 | 8, 10, or 12 |

### heif(options)

HEIC (hevc) requires globally-installed libvips with libheif, libde265, x265.

| Option | Default | Description |
|--------|---------|-------------|
| `compression` | _(required)_ | `'av1'` or `'hevc'` |
| `quality` | 50 | 1-100 |
| `lossless` | `false` | Lossless |
| `effort` | 4 | CPU effort 0-9 |
| `chromaSubsampling` | `'4:4:4'` | Chroma subsampling |
| `bitdepth` | 8 | 8, 10, or 12 |

### tiff([options])

| Option | Default | Description |
|--------|---------|-------------|
| `quality` | 80 | 1-100 |
| `compression` | `'jpeg'` | `'none'`, `'jpeg'`, `'deflate'`, `'packbits'`, `'ccittfax4'`, `'lzw'`, `'webp'`, `'zstd'`, `'jp2k'` |
| `predictor` | `'horizontal'` | `'none'`, `'horizontal'`, `'float'` |
| `pyramid` | `false` | Image pyramid |
| `tile` | `false` | Tiled TIFF |
| `tileWidth`/`tileHeight` | 256 | Tile dimensions |
| `xres`/`yres` | 1.0 | Resolution in pixels/mm |
| `resolutionUnit` | `'inch'` | `'inch'` or `'cm'` |
| `bitdepth` | 8 | 1, 2, 4, or 8 |
| `bigtiff` | `false` | BigTIFF variant |
| `miniswhite` | `false` | 1-bit as miniswhite |
| `force` | `true` | Force TIFF |

### jxl([options]) — experimental

Requires libvips with libjxl (not in prebuilt binaries).

| Option | Default | Description |
|--------|---------|-------------|
| `distance` | 1.0 | Max encoding error 0-15 |
| `quality` | | JPEG-like quality 1-100 (overrides distance) |
| `decodingTier` | 0 | Decode speed tier 0-4 |
| `lossless` | `false` | Lossless |
| `effort` | 7 | CPU effort 1-9 |
| `loop` | 0 | Animation iterations |
| `delay` | | Frame delay(s) in ms |

### jp2([options])

Requires libvips with OpenJPEG (not in prebuilt binaries).

| Option | Default | Description |
|--------|---------|-------------|
| `quality` | 80 | 1-100 |
| `lossless` | `false` | Lossless |
| `tileWidth`/`tileHeight` | 512 | Tile dimensions |
| `chromaSubsampling` | `'4:4:4'` | Chroma subsampling |

### raw([options])

Raw uncompressed pixel data. Left-to-right, top-to-bottom, no padding. RGB/RGBA for non-greyscale.

| Option | Default | Description |
|--------|---------|-------------|
| `depth` | `'uchar'` | `'char'`, `'uchar'`, `'short'`, `'ushort'`, `'int'`, `'uint'`, `'float'`, `'complex'`, `'double'`, `'dpcomplex'` |

```js
const { data, info } = await sharp('input.jpg').raw().toBuffer({ resolveWithObject: true });
const pixelArray = new Uint8ClampedArray(data.buffer);
```

### tile([options])

Deep zoom / image pyramid. Set format via `jpeg()`, `png()`, or `webp()` first. Use `.zip`/`.szi` extension with `toFile` for zip container.

| Option | Default | Description |
|--------|---------|-------------|
| `size` | 256 | Tile size 1-8192 |
| `overlap` | 0 | Tile overlap 0-8192 |
| `angle` | 0 | Rotation (multiple of 90) |
| `background` | white | Background colour |
| `depth` | | `'onepixel'`, `'onetile'`, `'one'` |
| `skipBlanks` | -1 | Threshold to skip blank tiles |
| `container` | `'fs'` | `'fs'` or `'zip'` (Buffer/Stream default to zip) |
| `layout` | `'dz'` | `'dz'`, `'iiif'`, `'iiif3'`, `'zoomify'`, `'google'` |
| `centre`/`center` | `false` | Centre image in tile |
| `basename` | | Directory name within zip |

```js
sharp('input.tiff').png().tile({ size: 512 }).toFile('output.dz');
const zip = await sharp(input).tile({ basename: 'tiles' }).toBuffer();
```

### timeout(options)

```js
await sharp(input).blur(1000).timeout({ seconds: 3 }).toBuffer();
```

Clock starts when libvips opens the input (excludes libuv thread wait).
