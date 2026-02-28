---
name: blocknote
description: Work with BlockNote, a block-based rich text editor for React. Use when the user mentions "blocknote", "BlockNote editor", "block-based editor", "insertBlocks", "updateBlock", "removeBlocks", "replaceBlocks", "editor.document", "InlineContent", "StyledText", "nestBlock", "unnestBlock", "addStyles", "toggleStyles", "removeStyles", "createLink", "getTextCursorPosition", "setTextCursorPosition", "getSelection", "setSelection", "forEachBlock", "useCreateBlockNote", "BlockNoteView", "paragraph block", "heading block", "bulletListItem", "PartialBlock", or asks how to manipulate blocks, style text, insert content, or read document structure in BlockNote.
metadata:
  author: Martin Clasen
  version: 1.0.0
  category: rich-text-editor
  tags: [blocknote, editor, react, blocks, rich-text, inline-content]
---

# BlockNote

Skill for working with [BlockNote](https://www.blocknotejs.org/), a block-based rich text editor for React.

## Core Mental Model

Everything in a BlockNote document is a **Block**. A paragraph is a block, a heading is a block, a list item is a block. Blocks can nest (children), and each block holds its rich text as an array of **InlineContent** objects.

```
Document
├── Block (paragraph)          ← top-level block
│   └── InlineContent[]        ← rich text inside
├── Block (heading)
│   └── InlineContent[]
└── Block (bulletListItem)
    ├── InlineContent[]
    └── Block (nested child)   ← indented sub-item
        └── InlineContent[]
```

## Block Type

```ts
type Block = {
  id: string;
  type: string;                                        // "paragraph" | "heading" | "bulletListItem" | ...
  props: Record<string, boolean | number | string>;    // type-specific properties
  content: InlineContent[] | TableContent | undefined; // rich text (undefined for images, columns)
  children: Block[];                                   // nested child blocks
};
```

When creating or updating blocks, use `PartialBlock` — all fields are optional:

```ts
editor.insertBlocks([{ type: "paragraph", content: "Hello!" }], refBlock);
editor.updateBlock(blockId, { props: { level: 2 } });
```

## InlineContent Types

```ts
type StyledText = { type: "text"; text: string; styles: Styles };
type Link      = { type: "link"; content: StyledText[]; href: string };
type InlineContent = StyledText | Link | CustomInlineContent;
```

`Styles` is a key/value map of active style attributes (e.g. `{ bold: true, textColor: "red" }`).

See [references/document-structure.md](references/document-structure.md) for full type definitions including `TableContent` and column blocks.

## Quick API Reference

### Reading

```ts
const all   = editor.document;                         // all top-level blocks
const block = editor.getBlock(id);                     // by id or Block ref
const prev  = editor.getPrevBlock(id);
const next  = editor.getNextBlock(id);
const parent = editor.getParentBlock(id);

editor.forEachBlock((block) => {
  console.log(block.id, block.type);
  return true;                                         // false stops traversal
});
```

### Inserting

```ts
// Insert before (default) or after a reference block
editor.insertBlocks(
  [{ type: "heading", content: "Title", props: { level: 2 } }],
  referenceBlock,
  "after",
);
```

### Updating

```ts
editor.updateBlock(blockId, { type: "heading", props: { level: 1 } });
editor.updateBlock(blockId, { content: "New text" });
```

### Removing & Replacing

```ts
editor.removeBlocks([id1, id2]);
editor.replaceBlocks([oldId], [{ type: "paragraph", content: "Replacement" }]);
```

### Moving & Nesting

```ts
editor.moveBlocksUp();
editor.moveBlocksDown();
if (editor.canNestBlock())   editor.nestBlock();
if (editor.canUnnestBlock()) editor.unnestBlock();
```

### Inline Content at Cursor

```ts
editor.insertInlineContent("plain text");
editor.insertInlineContent([
  { type: "text", text: "Bold", styles: { bold: true } },
  { type: "link", content: "BlockNote", href: "https://blocknotejs.org" },
]);
```

### Styles on Selection

```ts
editor.addStyles({ bold: true, textColor: "red" });
editor.removeStyles({ bold: true });
editor.toggleStyles({ italic: true });
const active = editor.getActiveStyles();               // styles at cursor / selection end
const text   = editor.getSelectedText();
```

### Links

```ts
editor.createLink("https://example.com");             // wraps selected text
editor.createLink("https://example.com", "Custom label");
const url = editor.getSelectedLinkUrl();
```

### Cursor & Selection

```ts
const pos = editor.getTextCursorPosition();           // { block, prevBlock, nextBlock, ... }
editor.setTextCursorPosition(blockId, "start");       // "start" | "end"
const sel = editor.getSelection();
editor.setSelection(startBlockId, endBlockId);
```

See [references/block-api.md](references/block-api.md) for the complete API reference with all parameters and examples.

## Common Patterns

### Insert block after the current cursor position

```ts
const { block } = editor.getTextCursorPosition();
editor.insertBlocks(
  [{ type: "paragraph", content: "Inserted after cursor" }],
  block,
  "after",
);
```

### Convert the current block to a heading

```ts
const { block } = editor.getTextCursorPosition();
editor.updateBlock(block, { type: "heading", props: { level: 2 } });
```

### Build a section programmatically

```ts
const anchor = editor.document[0];
editor.insertBlocks(
  [
    { type: "heading", content: "Section Title", props: { level: 2 } },
    { type: "paragraph", content: "Intro paragraph." },
    { type: "bulletListItem", content: "Item 1" },
    { type: "bulletListItem", content: "Item 2" },
  ],
  anchor,
  "after",
);
```

### Apply bold to selected text

```ts
editor.addStyles({ bold: true });
```

### Collect all blocks of a given type

```ts
const headings: Block[] = [];
editor.forEachBlock((block) => {
  if (block.type === "heading") headings.push(block);
  return true;
});
```

### Replace a block with a different type, preserving content

```ts
const block = editor.getBlock(blockId)!;
editor.replaceBlocks([block], [{ type: "heading", content: block.content, props: { level: 1 } }]);
```

## Key Gotchas

- **`content` can be `undefined`**: Image blocks, column blocks, and `columnList` blocks have no rich-text content — their `content` field is `undefined`. Always check before accessing.
- **`children` vs `content`**: Nested sub-blocks live in `children`, not `content`. `content` is only the inline rich text of the block itself.
- **`editor.document` is a snapshot**: It returns the document state at the time of the call. Subscribe to `onChange` to keep a reactive copy.
- **`PartialBlock` for writes**: When inserting or updating you never need to supply all fields. Omit anything you don't want to change.
- **`forEachBlock` is depth-first**: It traverses the entire tree including nested children, not just top-level blocks.
- **Column restrictions**: `columnList` children must be `column` blocks; `column` children must be regular blocks; a `columnList` needs at least 2 columns.
- **`moveBlocksUp/Down` acts on the selection**: Position the cursor or set a selection before calling these.
