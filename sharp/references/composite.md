# Sharp Composite Reference

## composite(images)

Layer one or more images over the processed base image. Pipeline operations (resize, rotate, flip, flop, extract) apply to the base before composition.

Overlays must be same size or smaller than the processed base.

### Per-Overlay Options

| Option | Default | Description |
|--------|---------|-------------|
| `input` | | Buffer, file path, or `{ create }` / `{ text }` object |
| `blend` | `'over'` | Blend mode |
| `gravity` | `'centre'` | Positioning gravity |
| `top` / `left` | | Pixel offsets (override gravity when both are set) |
| `tile` | `false` | Repeat overlay across entire image |
| `premultiplied` | `false` | Avoid premultiplying base image |
| `density` | 72 | DPI for vector overlays |
| `animated` | `false` | Read all frames of animated overlay |
| `autoOrient` | `false` | Apply EXIF orientation to overlay |
| `failOn` | `'warning'` | Same as constructor |
| `limitInputPixels` | 268402689 | Same as constructor |

### Blend Modes

`clear`, `source`, `over`, `in`, `out`, `atop`, `dest`, `dest-over`, `dest-in`, `dest-out`, `dest-atop`, `xor`, `add`, `saturate`, `multiply`, `screen`, `overlay`, `darken`, `lighten`, `colour-dodge`/`color-dodge`, `colour-burn`/`color-burn`, `hard-light`, `soft-light`, `difference`, `exclusion`

### Gravity Values

`'north'`, `'northeast'`, `'east'`, `'southeast'`, `'south'`, `'southwest'`, `'west'`, `'northwest'`, `'centre'`/`'center'`

### Examples

```js
// Basic multi-layer composite
await sharp(background)
  .composite([
    { input: 'layer1.png', gravity: 'northwest' },
    { input: 'layer2.png', gravity: 'southeast' },
  ])
  .toFile('combined.png');

// Tile overlay (e.g. watermark pattern)
await sharp('input.gif', { animated: true })
  .composite([{ input: 'overlay.png', tile: true, blend: 'saturate' }])
  .toBuffer();

// Text overlay
await sharp(background)
  .composite([{
    input: { text: { text: 'Copyright 2024', font: 'sans', dpi: 150, rgba: true } },
    gravity: 'southeast'
  }])
  .toFile('watermarked.jpg');

// Create overlay (solid colour box)
await sharp(background)
  .composite([{
    input: { create: { width: 100, height: 50, channels: 4, background: { r: 0, g: 0, b: 0, alpha: 0.5 } } },
    top: 20, left: 20
  }])
  .toFile('with-box.png');
```

### Text Overlay Options

| Option | Default | Description |
|--------|---------|-------------|
| `text` | _(required)_ | UTF-8 string, supports Pango markup |
| `font` | | Font name |
| `fontfile` | | Absolute path to font file |
| `width` | 0 | Word-wrap width (0 = no wrap) |
| `height` | 0 | Max height (overrides dpi) |
| `align` | `'left'` | `'left'`, `'centre'`/`'center'`, `'right'` |
| `justify` | `false` | Justify text |
| `dpi` | 72 | Render resolution |
| `rgba` | `false` | RGBA output (needed for colour) |
| `spacing` | 0 | Line height in points |

### Complex Pipeline Example

```js
sharp('input.png')
  .rotate(180)
  .resize(300)
  .flatten({ background: '#ff6600' })
  .composite([{ input: 'overlay.png', gravity: 'southeast' }])
  .sharpen()
  .withMetadata()
  .webp({ quality: 90 })
  .toBuffer();
```
