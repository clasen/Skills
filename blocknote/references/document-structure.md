# BlockNote Document Structure

Reference for all data types that make up a BlockNote document.

## Table of Contents

1. [Block](#block)
2. [InlineContent](#inlinecontent)
3. [Styles](#styles)
4. [TableContent](#tablecontent)
5. [Column Blocks](#column-blocks)

---

## Block

The `Block` type describes any block in the editor.

```ts
type Block = {
  id: string;
  type: string;
  props: Record<string, boolean | number | string>;
  content: InlineContent[] | TableContent | undefined;
  children: Block[];
};
```

### Fields

| Field | Description |
|---|---|
| `id` | Unique string ID. Stays constant from creation to removal. No two blocks share the same ID. |
| `type` | Block type identifier: `"paragraph"`, `"heading"`, `"bulletListItem"`, `"numberedListItem"`, `"checkListItem"`, `"table"`, `"image"`, `"video"`, `"audio"`, `"file"`, `"codeBlock"`, etc. |
| `props` | Type-specific key/value properties. E.g. a heading has `{ level: 1 \| 2 \| 3, ... }`. See built-in block type docs for the full list per type. |
| `content` | Rich text content as `InlineContent[]` for most blocks. `TableContent` for table blocks. `undefined` for blocks that contain no inline text (images, column blocks). |
| `children` | Nested child blocks (e.g. sub-items in a list). Represented as `Block[]`. Does **not** include the inline content — that's in `content`. |

### PartialBlock

When inserting or updating blocks, use `PartialBlock` — every field is optional:

```ts
type PartialBlock = Partial<Block> & { type?: string };

// Only the fields you care about are required
editor.insertBlocks([{ type: "paragraph", content: "Hello" }], refBlock);
editor.updateBlock(id, { props: { level: 2 } });
```

### Built-in Block Props (common)

All built-in text blocks share these base props:

```ts
{
  backgroundColor: string; // "default" | named color
  textColor: string;        // "default" | named color
  textAlignment: "left" | "center" | "right" | "justify";
}
```

Additional props by type:

| Type | Extra props |
|---|---|
| `heading` | `level: 1 \| 2 \| 3`, `isToggleable: boolean` |
| `checkListItem` | `checked: boolean` |
| `image` / `video` / `audio` / `file` | `url`, `caption`, `width`, `showPreview`, etc. |
| `codeBlock` | `language: string` |

---

## InlineContent

Inline content sits inside the `content` field of a block and represents rich text.

```ts
type InlineContent = StyledText | Link | CustomInlineContent;
```

### StyledText

A run of text with a set of active styles.

```ts
type StyledText = {
  type: "text";
  text: string;
  styles: Styles;
};
```

Example:

```ts
{ type: "text", text: "Hello, ", styles: {} }
{ type: "text", text: "world",   styles: { bold: true, textColor: "red" } }
```

### Link

A hyperlink containing styled text.

```ts
type Link = {
  type: "link";
  content: StyledText[];
  href: string;
};
```

Example:

```ts
{
  type: "link",
  content: [
    { type: "text", text: "Visit ", styles: {} },
    { type: "text", text: "BlockNote", styles: { bold: true } },
  ],
  href: "https://blocknotejs.org",
}
```

### CustomInlineContent

Custom inline content types added via editor schema customization.

```ts
type CustomInlineContent = {
  type: string;
  content: StyledText[] | undefined;
  props: Record<string, boolean | number | string>;
};
```

---

## Styles

The `styles` field on `StyledText` is a map of active style attributes.

```ts
type Styles = Partial<{
  bold: boolean;
  italic: boolean;
  underline: boolean;
  strikethrough: boolean;
  code: boolean;
  textColor: string;        // named color or hex (depends on schema)
  backgroundColor: string;  // named color or hex
}>;
```

An empty object `{}` means no styles are applied to that text run.

Named colors available by default: `"gray"`, `"brown"`, `"red"`, `"orange"`, `"yellow"`, `"green"`, `"blue"`, `"purple"`, `"pink"`.

Custom styles can be added via editor schema customization.

---

## TableContent

Tables use `TableContent` instead of `InlineContent[]`. Each cell is itself an array of `InlineContent`.

```ts
type TableContent = {
  type: "tableContent";
  columnWidths: (number | undefined)[];  // pixel widths per column; undefined = auto
  headerRows?: number;                   // number of header rows (from top)
  headerCols?: number;                   // number of header columns (from left)
  rows: {
    cells: InlineContent[][];            // cells[rowIndex][colIndex]
  }[];
};
```

Example — a 2×2 table:

```ts
{
  type: "tableContent",
  columnWidths: [200, 200],
  rows: [
    { cells: [
      [{ type: "text", text: "Name", styles: { bold: true } }],
      [{ type: "text", text: "Age",  styles: { bold: true } }],
    ]},
    { cells: [
      [{ type: "text", text: "Alice", styles: {} }],
      [{ type: "text", text: "30",    styles: {} }],
    ]},
  ],
}
```

---

## Column Blocks

From the `@blocknote/xl-multi-column` package. Allows side-by-side layout.

### ColumnBlock

```ts
type ColumnBlock = {
  id: string;
  type: "column";
  props: { width: number };  // relative width (e.g. 1 = equal, 2 = twice as wide)
  content: undefined;
  children: Block[];         // regular blocks inside this column
};
```

### ColumnListBlock

```ts
type ColumnListBlock = {
  id: string;
  type: "columnList";
  props: {};
  content: undefined;
  children: ColumnBlock[];   // must be column blocks only
};
```

### Restrictions

| Rule | Detail |
|---|---|
| `columnList` children | Must be `column` blocks only |
| `column` children | Must be regular (non-column) blocks |
| Minimum columns | A `columnList` must have at least 2 `column` children |
| No inline content | Both `column` and `columnList` have `content: undefined` |

### Example Structure

```
columnList
├── column (width: 1)
│   ├── paragraph "Left content"
│   └── bulletListItem "Point A"
└── column (width: 1)
    └── paragraph "Right content"
```
