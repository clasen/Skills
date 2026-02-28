# Sharp Constructor Reference

## new Sharp([input], [options])

Creates a Sharp instance. Chain operations, then call an output method.

### Input Types

| Type | Description |
|------|-------------|
| `string` | Filesystem path to JPEG, PNG, WebP, AVIF, GIF, SVG, or TIFF |
| `Buffer` / `ArrayBuffer` / `Uint8Array` / `Uint8ClampedArray` | Image data in memory |
| `TypedArray` (Int8, Uint16, Int16, Uint32, etc.) | Raw pixel data (requires `raw` option) |
| `Array` | Multiple inputs to join together |
| _(omitted)_ | Stream input — pipe into the Sharp instance |

### Constructor Options

| Option | Default | Description |
|--------|---------|-------------|
| `failOn` | `'warning'` | Abort on invalid pixel data: `'none'`, `'truncated'`, `'error'`, `'warning'` |
| `limitInputPixels` | 268402689 | Max pixels (width x height). `false`/`0` to disable |
| `unlimited` | `false` | Remove memory safety features (JPEG, PNG, SVG, HEIF) |
| `autoOrient` | `false` | Rotate/flip to match EXIF Orientation on load |
| `sequentialRead` | `true` | Sequential read (faster). `false` for random access |
| `density` | 72 | DPI for vector images (1-100000) |
| `ignoreIcc` | `false` | Ignore embedded ICC profile |
| `pages` | 1 | Pages to extract for multi-page input. `-1` for all |
| `page` | 0 | Starting page (zero-based) |
| `animated` | `false` | Read all frames (equivalent to `pages: -1`) |

### Raw Pixel Input (`options.raw`)

| Option | Description |
|--------|-------------|
| `raw.width` | Pixels wide |
| `raw.height` | Pixels high |
| `raw.channels` | Number of channels (1-4) |
| `raw.premultiplied` | Input already premultiplied (default `false`) |
| `raw.pageHeight` | Height per page/frame for animated raw input |

```js
const pixels = Uint8Array.from([255, 255, 255, 0, 0, 0]);
const image = sharp(pixels, { raw: { width: 2, height: 1, channels: 3 } });
await image.toFile('two-pixels.png');
```

### Create Blank Image (`options.create`)

| Option | Description |
|--------|-------------|
| `create.width` | Pixels wide |
| `create.height` | Pixels high |
| `create.channels` | 3 (RGB) or 4 (RGBA) |
| `create.background` | Colour (parsed by the `color` module) |
| `create.pageHeight` | Height per page/frame for animated images |
| `create.noise.type` | `'gaussian'` (only option) |
| `create.noise.mean` | Mean pixel value (default 128) |
| `create.noise.sigma` | Standard deviation (default 30) |

```js
// Blank semi-transparent red image
sharp({ create: { width: 300, height: 200, channels: 4, background: { r: 255, g: 0, b: 0, alpha: 0.5 } } })
  .png().toBuffer();

// Gaussian noise
sharp({ create: { width: 300, height: 200, channels: 3, noise: { type: 'gaussian', mean: 128, sigma: 30 } } })
  .toFile('noise.png');
```

### Text Image (`options.text`)

| Option | Default | Description |
|--------|---------|-------------|
| `text.text` | _(required)_ | UTF-8 string, supports Pango markup |
| `text.font` | | Font name |
| `text.fontfile` | | Absolute path to font file |
| `text.width` | 0 | Word-wrap width (0 = no wrap) |
| `text.height` | 0 | Max height (overrides dpi). Ignored if width is 0 |
| `text.align` | `'left'` | `'left'`, `'centre'`/`'center'`, `'right'` |
| `text.justify` | `false` | Justify text |
| `text.dpi` | 72 | Render resolution |
| `text.rgba` | `false` | RGBA output (needed for colour emoji / Pango colour markup) |
| `text.spacing` | 0 | Line height in points |
| `text.wrap` | `'word'` | `'word'`, `'char'`, `'word-char'`, `'none'` |

```js
// Simple text
await sharp({ text: { text: 'Hello, world!', width: 400, height: 300 } }).toFile('text.png');

// Coloured Pango markup
await sharp({
  text: { text: '<span foreground="red">Red!</span>', font: 'sans', rgba: true, dpi: 300 }
}).toFile('colour-text.png');
```

### Join Multiple Inputs (`options.join`)

| Option | Default | Description |
|--------|---------|-------------|
| `join.across` | 1 | Images to join horizontally |
| `join.animated` | `false` | Join as animated image |
| `join.shim` | 0 | Pixels between joined images |
| `join.background` | | Background colour for gaps |
| `join.halign` | `'left'` | Horizontal alignment: `'left'`, `'centre'`, `'right'` |
| `join.valign` | `'top'` | Vertical alignment: `'top'`, `'centre'`, `'bottom'` |

```js
// 2x2 grid with 4px gutter
const data = await sharp([img1, img2, img3, img4], { join: { across: 2, shim: 4 } }).toBuffer();

// Two-frame animated image from emoji
const frames = ['😀', '😛'].map(text => ({ text: { text, width: 64, height: 64, channels: 4, rgba: true } }));
await sharp(frames, { join: { animated: true } }).toFile('animated.gif');
```

### Format-Specific Input Options

| Option | Description |
|--------|-------------|
| `tiff.subifd` | Sub IFD to extract for OME-TIFF (default -1, main image) |
| `svg.stylesheet` | Custom CSS for SVG input |
| `svg.highBitdepth` | Render SVG at 32-bit per channel (default `false`) |
| `pdf.background` | Background colour for transparent PDF areas |
| `openSlide.level` | Level for multi-level input (default 0) |
| `jp2.oneshot` | Decode tiled JP2 in single operation (default `false`) |

### clone()

Snapshot a Sharp instance to create multiple output pipelines from a single input.

```js
const pipeline = sharp().rotate();
pipeline.clone().resize(800, 600).pipe(firstStream);
pipeline.clone().extract({ left: 20, top: 20, width: 100, height: 100 }).pipe(secondStream);
readableStream.pipe(pipeline);
```

### Stream Events

- `info` event: emitted with output dimensions when using stream output
- `warning` event: non-critical problems during processing

### Global Configuration

| Method | Description |
|--------|-------------|
| `sharp.cache([options])` | Get/set libvips cache limits. `{ memory: 50, files: 20, items: 100 }`. `false` to disable |
| `sharp.concurrency([n])` | Get/set max threads (default = CPU cores). `0` to reset |
| `sharp.simd([bool])` | Get/set SIMD instructions (requires highway). Improves resize/blur/sharpen |
| `sharp.block({ operation })` | Block libvips operations at runtime |
| `sharp.unblock({ operation })` | Unblock specific operations |
| `sharp.versions` | Version info for sharp, libvips, dependencies |
| `sharp.format` | Available input/output formats |
| `sharp.queue` | EventEmitter for task queue changes |
| `sharp.counters()` | `{ queue, process }` task counts |

```js
// Allow only WebP from filesystem
sharp.block({ operation: ['VipsForeignLoad'] });
sharp.unblock({ operation: ['VipsForeignLoadWebpFile'] });
```
