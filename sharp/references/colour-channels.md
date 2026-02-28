# Sharp Colour, Metadata Preservation, and Channel Reference

## Output Metadata Preservation

Sharp strips all metadata by default and converts to sRGB.

### Methods Comparison

| Method | Keeps EXIF | Keeps ICC | Keeps XMP | Keeps IPTC | Converts to sRGB |
|--------|-----------|-----------|-----------|------------|-------------------|
| _(default)_ | No | No | No | No | Yes |
| `keepMetadata()` | Yes | Yes | Yes | Yes | No |
| `withMetadata()` | Yes | sRGB added | Yes | Yes | Yes |
| `keepExif()` | Yes | No | No | No | - |
| `keepIccProfile()` | No | Yes | No | No | - |
| `keepXmp()` | No | No | Yes | No | - |

**keepMetadata vs withMetadata**: `keepMetadata()` preserves everything as-is including original ICC. `withMetadata()` converts to sRGB and adds web-friendly ICC — better for web delivery.

### EXIF Methods

```js
sharp(input).keepExif().toBuffer();                             // preserve input EXIF
sharp(input).withExif({ IFD0: { Copyright: '...' } }).toBuffer();    // replace EXIF
sharp(input).withExifMerge({ IFD0: { Copyright: '...' } }).toBuffer(); // merge with input EXIF
```

`keepExif()` is unsupported for TIFF output.

### ICC Profile Methods

Built-in profiles: `'srgb'`, `'p3'`, `'cmyk'`

```js
sharp(input).keepIccProfile().toBuffer();
sharp(input).withIccProfile('p3').toBuffer();
sharp(input).withIccProfile('/path/to/profile.icc', { attach: false }).toBuffer();
```

For CMYK workflows:

```js
await sharp(cmykInput).pipelineColourspace('cmyk').toColourspace('cmyk').keepIccProfile().toBuffer();
```

### XMP Methods

Supported by PNG, JPEG, WebP, TIFF output.

```js
sharp(input).keepXmp().toBuffer();
sharp(input).withXmp(xmpString).toBuffer();
```

### withMetadata([options])

Keep most metadata + convert to sRGB. Options:

| Option | Description |
|--------|-------------|
| `orientation` | Set EXIF Orientation (1-8) |
| `density` | Set DPI |

## Colour Space

### toColourspace(space) / toColorspace(space)

Set output colour space. Default: `srgb`.

Values: `'srgb'`, `'rgb'`, `'rgb16'`, `'cmyk'`, `'lab'`, `'b-w'`, `'grey16'`, `'scrgb'`

### pipelineColourspace(space) / pipelineColorspace(space)

Set internal processing colour space. Input converts at start, converts to output space at end.

```js
await sharp(input).pipelineColourspace('rgb16').toColourspace('srgb').toFile('output.png');
```

### greyscale() / grayscale()

8-bit greyscale (256 shades). Linear operation — use `gamma()` for best results on sRGB input. Output has 3 identical channels by default; use `toColourspace('b-w')` for single channel.

### tint(colour)

Tint using a colour. Alpha unchanged.

```js
await sharp(input).tint({ r: 255, g: 240, b: 16 }).toBuffer();
```

## Channel Operations

### removeAlpha()

Remove alpha channel. No-op if no alpha.

### ensureAlpha([alpha])

Add alpha if missing. Default `alpha: 1` (opaque). `alpha: 0` for transparent. No-op if already has alpha.

### extractChannel(channel)

Extract single channel. Output: `b-w` (8-bit) or `grey16` (16-bit).

Channel: `0`/`1`/`2`/`3` or `'red'`/`'green'`/`'blue'`/`'alpha'`

```js
await sharp(input).extractChannel('green').toFile('green.jpg');
```

### joinChannel(images, options)

Add channels. Meaning depends on output colourspace:
- sRGB: 0=Red, 1=Green, 2=Blue, 3=Alpha
- CMYK: 0=Magenta, 1=Cyan, 2=Yellow, 3=Black, 4=Alpha

### bandbool(boolOp)

Bitwise boolean across all channels to single-channel output: `'and'`, `'or'`, `'eor'`.

```js
sharp('rgb.png').bandbool(sharp.bool.and).toFile('single.png');
// Each pixel P = R & G & B
```
