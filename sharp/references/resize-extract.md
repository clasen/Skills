# Sharp Resize, Extract, Extend, Trim Reference

## resize([width], [height], [options])

Only one resize per pipeline — subsequent calls override previous ones.

### Fit Modes

| Fit | Behaviour |
|-----|-----------|
| `'cover'` (default) | Preserve aspect ratio, crop to fill both dimensions |
| `'contain'` | Preserve aspect ratio, letterbox with background |
| `'fill'` | Stretch to both dimensions (ignores aspect ratio) |
| `'inside'` | As large as possible while both dimensions are within target |
| `'outside'` | As small as possible while both dimensions exceed target |

### Position (for cover/contain)

Default: `'centre'`

- **Position keywords**: `'top'`, `'right top'`, `'right'`, `'right bottom'`, `'bottom'`, `'left bottom'`, `'left'`, `'left top'`
- **Gravity keywords**: `'north'`, `'northeast'`, `'east'`, `'southeast'`, `'south'`, `'southwest'`, `'west'`, `'northwest'`, `'center'`/`'centre'`
- **Strategy (cover only)**: `sharp.strategy.entropy` (highest Shannon entropy), `sharp.strategy.attention` (luminance frequency, colour saturation, skin tones)

### All Options

| Option | Default | Description |
|--------|---------|-------------|
| `width` | | Pixels wide (alternative to positional arg) |
| `height` | | Pixels high (alternative to positional arg) |
| `fit` | `'cover'` | How to fit target dimensions |
| `position` | `'centre'` | Position, gravity, or strategy |
| `background` | `{r:0,g:0,b:0,alpha:1}` | Background for `contain` |
| `kernel` | `'lanczos3'` | Downsizing kernel |
| `withoutEnlargement` | `false` | Don't upscale if already smaller |
| `withoutReduction` | `false` | Don't downscale if already larger |
| `fastShrinkOnLoad` | `true` | JPEG/WebP shrink-on-load (may cause moiré) |

### Downsizing Kernels

`nearest`, `linear`, `cubic`, `mitchell`, `lanczos2`, `lanczos3` (default), `mks2013`, `mks2021`

When upsampling, these map to `nearest`, `linear`, or `cubic` interpolators.

```js
// Width only (auto height)
await sharp(input).resize({ width: 100 }).toBuffer();

// Contain with background
sharp(input).resize(200, 300, { fit: 'contain', position: 'right top', background: { r: 255, g: 255, b: 255, alpha: 0.5 } });

// Entropy-based smart crop
sharp().resize({ width: 200, height: 200, fit: sharp.fit.cover, position: sharp.strategy.entropy });

// No enlargement
sharp(input).resize(200, 200, { fit: sharp.fit.inside, withoutEnlargement: true });

// Scale by percentage
const { width } = await sharp(input).metadata();
await sharp(input).resize(Math.round(width * 0.5)).toBuffer();
```

## extract(options)

Crop a region using pixel coordinates.

| Option | Description |
|--------|-------------|
| `left` | Zero-indexed offset from left |
| `top` | Zero-indexed offset from top |
| `width` | Width of region |
| `height` | Height of region |

### Operation Order

- `extract` before `resize` = pre-resize crop
- `extract` after `resize` = post-resize crop
- Two `extract` + one `resize` = extract-resize-extract

```js
sharp(input)
  .extract({ left: 0, top: 0, width: 500, height: 500 })
  .resize(200, 200)
  .extract({ left: 10, top: 10, width: 180, height: 180 })
  .toFile('result.jpg');
```

## extend(extend)

Add padding to edges. Always occurs after resizing and extraction.

| Option | Default | Description |
|--------|---------|-------------|
| `top`/`left`/`bottom`/`right` | 0 | Pixels to add per edge |
| `extendWith` | `'background'` | `'background'`, `'copy'`, `'repeat'`, `'mirror'` |
| `background` | black | Fill colour |

Can also pass a single number to add equally to all edges.

```js
sharp(input).resize(140).extend({ top: 10, bottom: 20, left: 10, right: 10, background: { r: 0, g: 0, b: 0, alpha: 0 } });
sharp(input).extend({ right: 8, extendWith: 'mirror' });
```

## trim([options])

Remove edges similar to a reference colour (default: top-left pixel).

| Option | Default | Description |
|--------|---------|-------------|
| `background` | top-left pixel | Reference colour |
| `threshold` | 10 | Allowed colour difference |
| `lineArt` | `false` | Better for vector/line art input |

Output info includes `trimOffsetLeft` and `trimOffsetTop`. If trimming would produce an empty image, no change is made.

```js
await sharp(input).trim().toFile(output);                               // default
await sharp(input).trim({ threshold: 0 }).toFile(output);              // exact match only
await sharp(input).trim({ background: '#FF0000', lineArt: true }).toBuffer(); // line art
await sharp(input).trim({ background: 'yellow', threshold: 42 }).toBuffer();  // lenient
```
