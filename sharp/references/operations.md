# Sharp Operations Reference

## Rotation and Orientation

### rotate([angle], [options])

Only one rotation per pipeline. Angle converts to valid positive degrees (-450 becomes 270). Multi-page images: 180 only.

Method order matters: `.rotate(x).extract(y)` differs from `.extract(y).rotate(x)`.

`options.background`: colour for non-90-degree rotations (default black).

```js
await sharp(input).rotate(90).toBuffer();
await sharp(input).rotate(45, { background: '#ff0000' }).toBuffer();
```

### autoOrient()

Auto-orient via EXIF Orientation tag, then remove the tag. Mirroring supported. Subsequent `rotate(angle)`, `flip()`, `flop()` logically occur after auto-orientation regardless of call order.

```js
await sharp(input).autoOrient().toBuffer();
```

### flip([bool]) / flop([bool])

- `flip()`: Mirror vertically (up-down, about x-axis). Always before rotation. Does not work with multi-page images.
- `flop()`: Mirror horizontally (left-right, about y-axis). Always before rotation.

### affine(matrix, [options])

Affine transform. Occurs after resizing, extraction, rotation.

Matrix: array of 4 numbers or 2x2 nested array.

| Option | Default | Description |
|--------|---------|-------------|
| `background` | black | Fill colour |
| `idx`/`idy` | 0 | Input offsets |
| `odx`/`ody` | 0 | Output offsets |
| `interpolator` | `sharp.interpolators.bicubic` | See interpolators below |

**Interpolators**: `sharp.interpolators.nearest`, `.bilinear`, `.bicubic`, `.locallyBoundedBicubic` (`'lbb'`), `.nohalo`, `.vertexSplitQuadraticBasisSpline` (`'vsqbs'`)

```js
sharp().affine([[1, 0.3], [0.1, 0.7]], { background: 'white', interpolator: sharp.interpolators.nohalo });
```

## Sharpening and Blurring

### sharpen([options])

No args: fast, mild sharpen. With `sigma`: slower, accurate sharpen on LAB L channel.

| Option | Default | Description |
|--------|---------|-------------|
| `sigma` | | Gaussian mask sigma (0.000001-10) |
| `m1` | 1.0 | Sharpening for "flat" areas (0-1000000) |
| `m2` | 2.0 | Sharpening for "jagged" areas (0-1000000) |
| `x1` | 2.0 | Flat/jagged threshold (0-1000000) |
| `y2` | 10.0 | Max brightening (0-1000000) |
| `y3` | 20.0 | Max darkening (0-1000000) |

```js
await sharp(input).sharpen().toBuffer();
await sharp(input).sharpen({ sigma: 2, m1: 0, m2: 3, x1: 3, y2: 15, y3: 15 }).toBuffer();
```

### blur([options])

No args: fast 3x3 box blur. With sigma: Gaussian blur.

| Option | Default | Description |
|--------|---------|-------------|
| `sigma` | | 0.3-1000 |
| `precision` | `'integer'` | `'integer'`, `'float'`, `'approximate'` |
| `minAmplitude` | 0.2 | Mask accuracy 0.001-1 (smaller = larger, more accurate mask) |

```js
await sharp(input).blur().toBuffer();
await sharp(input).blur(5).toBuffer();
```

### median([size])

Median filter for noise reduction. Default 3x3.

```js
await sharp(input).median(5).toBuffer();
```

## Morphology

### dilate([width]) / erode([width])

Expand/shrink foreground objects. Default width: 1 pixel.

```js
await sharp(input).dilate(3).toBuffer();
await sharp(input).erode().toBuffer();
```

## Thresholding

### threshold([value], [options])

Pixels >= value become 255, others 0. Default: 128.

| Option | Default | Description |
|--------|---------|-------------|
| `greyscale`/`grayscale` | `true` | Convert to single channel greyscale |

### boolean(operand, operator, [options])

Bitwise operation between two images: `'and'`, `'or'`, `'eor'`.

```js
await sharp(input).boolean('mask.png', 'and').toBuffer();
```

## Colour and Tone

### modulate([options])

| Option | Description |
|--------|-------------|
| `brightness` | Multiplier (1.0 = no change, 2.0 = double) |
| `saturation` | Multiplier |
| `hue` | Degrees of rotation |
| `lightness` | Additive offset (0 = no change) |

```js
await sharp(input).modulate({ brightness: 0.5, saturation: 0.5, hue: 90 }).toBuffer();
```

### linear([a], [b])

Apply `a * input + b`. Single number for all channels, array for per-channel.

```js
await sharp(rgbInput).linear([0.25, 0.5, 0.75], [150, 100, 50]).toBuffer();
```

### gamma([gamma], [gammaOut])

Gamma correction: darken pre-resize at `1/gamma`, brighten post-resize at `gamma`. Range 1.0-3.0. Disables JPEG/WebP shrink-on-load.

### negate([options])

Produce the negative. `options.alpha`: whether to negate alpha (default `true`).

### normalise([options]) / normalize([options])

Stretch luminance to full dynamic range.

| Option | Default | Description |
|--------|---------|-------------|
| `lower` | 1 | Percentile for underexposure |
| `upper` | 99 | Percentile for overexposure |

### clahe(options)

Contrast Limiting Adaptive Histogram Equalization. Enhances darker details.

| Option | Default | Description |
|--------|---------|-------------|
| `width` | _(required)_ | Search window width |
| `height` | _(required)_ | Search window height |
| `maxSlope` | 3 | Brightening level 0-100 (0 disables limiting) |

## Alpha and Flattening

### flatten([options])

Merge alpha with background, then remove alpha channel. `options.background` defaults to black.

```js
await sharp(rgbaInput).flatten({ background: '#F0A703' }).toBuffer();
```

### unflatten()

Add alpha channel, make white pixels fully transparent. Experimental.

```js
await sharp(input).threshold(128, { grayscale: false }).unflatten().toBuffer();
```

## Convolution

### convolve(kernel)

| Option | Default | Description |
|--------|---------|-------------|
| `width` | | Kernel width |
| `height` | | Kernel height |
| `kernel` | | Flat array of length width*height |
| `scale` | sum | Kernel scale |
| `offset` | 0 | Kernel offset |

```js
// Horizontal Sobel
sharp(input).convolve({ width: 3, height: 3, kernel: [-1, 0, 1, -2, 0, 2, -1, 0, 1] });
```

### recomb(matrix)

3x3 or 4x4 recombination matrix.

```js
// Sepia filter
sharp(input).recomb([
  [0.3588, 0.7044, 0.1368],
  [0.2990, 0.5870, 0.1140],
  [0.2392, 0.4696, 0.0912],
]);
```
