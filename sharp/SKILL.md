---
name: sharp
description: Process images with the Sharp library for Node.js — resize, convert formats, composite, apply effects, and manage metadata. Use when the user mentions "sharp", "image processing", "resize image", "convert image", "image format", "jpeg quality", "png compression", "webp", "avif", "image thumbnail", "crop image", "watermark", "overlay image", "blur image", "sharpen image", "image metadata", "EXIF", "ICC profile", "colour space", "alpha channel", "animated gif", "image pipeline", or asks how to manipulate images in Node.js/TypeScript. Also use for "sharp constructor", "sharp cache", "sharp concurrency", "toFile", "toBuffer", or any Sharp API method.
metadata:
  author: Martin Clasen
  version: 1.0.0
  category: image-processing
  tags: [sharp, image, nodejs, resize, convert, composite, effects, metadata]
---

# Sharp

Skill for image processing with the [Sharp](https://sharp.pixelplumbing.com/) library in Node.js. Sharp is a high-performance image pipeline built on libvips.

## How Sharp Works

Sharp uses a chainable API: create an instance, chain operations, then output. All operations are lazy — nothing executes until an output method is called.

```js
const sharp = require('sharp');

sharp('input.jpg')       // input: file path, Buffer, Stream, or create options
  .resize(300, 200)      // chain operations
  .jpeg({ quality: 80 }) // set output format
  .toFile('output.jpg'); // trigger execution
```

Sharp implements `stream.Duplex`, so it can also be piped:

```js
const transformer = sharp().resize(300).on('info', ({ height }) => console.log(height));
readableStream.pipe(transformer).pipe(writableStream);
```

## Quick Reference

### Constructor

```js
sharp(input, options)
```

Input can be: file path string, Buffer, TypedArray (raw pixels), Array of inputs (joined), or omitted for stream input.

Key options: `animated: true` (read all frames), `pages: -1` (all pages), `failOn`, `limitInputPixels`, `density` (DPI for vectors), `raw` (pixel dimensions), `create` (blank image), `text` (text rendering), `join` (grid/animation).

See [references/constructor.md](references/constructor.md) for all constructor options.

### Output

```js
await sharp(input).toFile('output.png');                              // to file (format from extension)
const buffer = await sharp(input).png().toBuffer();                   // to buffer
const { data, info } = await sharp(input).toBuffer({ resolveWithObject: true }); // buffer + metadata
sharp(input).pipe(writableStream);                                    // to stream
```

By default all metadata is stripped and images convert to sRGB. Use `keepMetadata()` or `withMetadata()` to preserve.

### Resize

```js
sharp(input).resize(width, height, options)
```

Fit modes: `cover` (default, crop to fill), `contain` (letterbox), `fill` (stretch), `inside` (fit within), `outside` (cover at minimum).

Position/strategy for cover/contain: gravity keywords (`'centre'`, `'north'`, etc.), `sharp.strategy.entropy`, `sharp.strategy.attention`.

Key options: `withoutEnlargement`, `withoutReduction`, `kernel` (lanczos3 default).

See [references/resize-extract.md](references/resize-extract.md) for resize, extract, extend, trim details.

### Format Conversion

```js
sharp(input).jpeg({ quality: 80, mozjpeg: true }).toBuffer();
sharp(input).png({ palette: true }).toBuffer();
sharp(input).webp({ lossless: true }).toBuffer();
sharp(input).avif({ effort: 2 }).toBuffer();
sharp(input).gif({ dither: 0 }).toBuffer();
sharp(input).tiff({ compression: 'lzw' }).toBuffer();
sharp(input).toFormat('png').toBuffer();
```

See [references/output-formats.md](references/output-formats.md) for all format options and tile/deep-zoom output.

### Composite / Overlay

```js
await sharp(background)
  .composite([
    { input: 'logo.png', gravity: 'southeast' },
    { input: { text: { text: 'Watermark', rgba: true } }, top: 10, left: 10 },
  ])
  .toFile('output.png');
```

Overlays must be same size or smaller than the processed base. Pipeline ops (resize, rotate, etc.) apply before composition. When `top`/`left` are set they override `gravity`.

See [references/composite.md](references/composite.md) for blend modes, text overlays, tiling.

### Operations and Effects

```js
sharp(input).rotate(90).toBuffer();
sharp(input).autoOrient().toBuffer();
sharp(input).flip().toBuffer();                         // vertical mirror
sharp(input).flop().toBuffer();                         // horizontal mirror
sharp(input).blur(5).toBuffer();                        // Gaussian blur
sharp(input).sharpen({ sigma: 2 }).toBuffer();
sharp(input).modulate({ brightness: 1.5, saturation: 0.8, hue: 90 }).toBuffer();
sharp(input).normalise().toBuffer();                    // stretch contrast
sharp(input).threshold(128).toBuffer();
sharp(input).flatten({ background: '#ffffff' }).toBuffer(); // remove alpha
sharp(input).negate().toBuffer();
```

See [references/operations.md](references/operations.md) for all operations including affine, clahe, convolve, recomb, median, dilate, erode.

### Metadata

```js
const meta = await sharp(input).metadata();
// { format, width, height, space, channels, hasAlpha, orientation, pages, ... }

const stats = await sharp(input).stats();
// { channels, isOpaque, entropy, sharpness, dominant: { r, g, b } }

// EXIF-aware dimensions
const { autoOrient } = await sharp(input).metadata();
const { width, height } = autoOrient;
```

See [references/metadata.md](references/metadata.md) for all metadata fields and stats details.

### Output Metadata Control

```js
sharp(input).keepMetadata().toBuffer();          // keep everything as-is
sharp(input).withMetadata().toBuffer();          // keep most + convert to sRGB
sharp(input).keepExif().toBuffer();              // EXIF only
sharp(input).withExif({ IFD0: { Copyright: '...' } }).toBuffer();  // set EXIF
sharp(input).withExifMerge({ IFD0: { ... } }).toBuffer();          // merge EXIF
sharp(input).keepIccProfile().toBuffer();        // keep ICC
sharp(input).withIccProfile('p3').toBuffer();    // convert to profile
sharp(input).keepXmp().toBuffer();               // keep XMP
sharp(input).withXmp(xmpString).toBuffer();      // set XMP
```

### Colour Space and Channels

```js
sharp(input).greyscale().toBuffer();
sharp(input).tint({ r: 255, g: 240, b: 16 }).toBuffer();
sharp(input).toColourspace('rgb16').toBuffer();
sharp(input).pipelineColourspace('rgb16').toColourspace('srgb').toBuffer();
sharp(input).removeAlpha().toBuffer();
sharp(input).ensureAlpha(0).toBuffer();
sharp(input).extractChannel('green').toBuffer();
```

See [references/colour-channels.md](references/colour-channels.md) for colour space, metadata preservation, and channel operations.

### Global Configuration

```js
sharp.cache({ memory: 50, files: 20, items: 100 }); // configure cache
sharp.cache(false);                                   // disable cache
sharp.concurrency(2);                                 // set thread count
sharp.simd(false);                                    // disable SIMD
sharp.block({ operation: ['VipsForeignLoad'] });      // block operations
sharp.unblock({ operation: ['VipsForeignLoadWebpFile'] });
console.log(sharp.versions);                          // version info
console.log(sharp.format);                            // available formats
```

## Common Patterns

### Read metadata, then transform

```js
const image = sharp(input);
const { width } = await image.metadata();
const result = await image.resize(Math.round(width / 2)).webp().toBuffer();
```

### Multiple outputs from single input (clone)

```js
const pipeline = sharp({ failOn: 'none' });
const promises = [
  pipeline.clone().jpeg({ quality: 100 }).toFile('original.jpg'),
  pipeline.clone().resize(500).jpeg({ quality: 80 }).toFile('thumb.jpg'),
  pipeline.clone().resize(500).webp({ quality: 80 }).toFile('thumb.webp'),
];
readableStream.pipe(pipeline);
await Promise.all(promises);
```

### Animated image conversion

```js
await sharp('input.gif', { animated: true }).toFile('output.webp');
```

### Create blank image

```js
await sharp({ create: { width: 300, height: 200, channels: 4, background: '#ff0000' } })
  .png().toFile('red.png');
```

### Text rendering

```js
await sharp({ text: { text: 'Hello!', width: 400, height: 300 } }).toFile('text.png');
```

### Join images as grid

```js
const grid = await sharp([img1, img2, img3, img4], { join: { across: 2, shim: 4 } }).toBuffer();
```

### Timeout

```js
try {
  await sharp(input).blur(1000).timeout({ seconds: 3 }).toBuffer();
} catch (err) {
  if (err.message.includes('timeout')) { /* handle */ }
}
```

## Key Gotchas

- **Metadata stripped by default**: Output loses EXIF, ICC, XMP. Use `keepMetadata()` or `withMetadata()`.
- **EXIF orientation**: `metadata().width/height` are raw pixel dims, not visual. Use `metadata().autoOrient` for visual dimensions.
- **One resize per pipeline**: Subsequent `resize()` calls override previous ones.
- **Operation order matters**: `rotate(x).extract(y)` differs from `extract(y).rotate(x)`. Pipeline ops apply before `composite()`.
- **Animated images**: Must pass `{ animated: true }` or `{ pages: -1 }` in constructor, otherwise only the first frame is read.
- **Stats from original**: `stats()` reads from the original input, not after pipeline ops. Write to buffer first if you need stats on a transformed result.
- **Overlay size**: Composite overlays must be same size or smaller than the processed base image.
