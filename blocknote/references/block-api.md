# BlockNote Block API Reference

Complete reference for the `editor` instance API.

## Table of Contents

1. [Reading Blocks](#reading-blocks)
2. [Inserting Blocks](#inserting-blocks)
3. [Updating Blocks](#updating-blocks)
4. [Removing Blocks](#removing-blocks)
5. [Replacing Blocks](#replacing-blocks)
6. [Moving Blocks](#moving-blocks)
7. [Nesting Blocks](#nesting-blocks)
8. [Inserting Inline Content](#inserting-inline-content)
9. [Reading Inline Content & Styles](#reading-inline-content--styles)
10. [Styling Text](#styling-text)
11. [Working with Links](#working-with-links)
12. [Cursor & Selections](#cursor--selections)

---

## Reading Blocks

### `editor.document`

Returns a snapshot of all top-level (non-nested) blocks.

```ts
const blocks: Block[] = editor.document;
```

### `editor.getBlock(blockIdentifier)`

```ts
getBlock(blockIdentifier: BlockIdentifier): Block | undefined
```

Returns the block matching the given ID or Block reference, or `undefined` if not found.

```ts
const block = editor.getBlock("block-123");
```

### `editor.getPrevBlock(blockIdentifier)`

```ts
getPrevBlock(blockIdentifier: BlockIdentifier): Block | undefined
```

Returns the block immediately before the given block in document order, or `undefined` if there is none.

```ts
const prev = editor.getPrevBlock("block-123");
```

### `editor.getNextBlock(blockIdentifier)`

```ts
getNextBlock(blockIdentifier: BlockIdentifier): Block | undefined
```

Returns the block immediately after the given block in document order.

```ts
const next = editor.getNextBlock("block-123");
```

### `editor.getParentBlock(blockIdentifier)`

```ts
getParentBlock(blockIdentifier: BlockIdentifier): Block | undefined
```

Returns the parent block of a nested block, or `undefined` for top-level blocks.

```ts
const parent = editor.getParentBlock("nested-block-123");
```

### `editor.forEachBlock(callback, reverse?)`

```ts
forEachBlock(
  callback: (block: Block) => boolean,
  reverse?: boolean
): void
```

Traverses all blocks depth-first (including nested children) and calls `callback` for each. Return `false` from the callback to stop traversal early. Pass `reverse: true` to traverse in reverse order.

```ts
// Collect all heading blocks
const headings: Block[] = [];
editor.forEachBlock((block) => {
  if (block.type === "heading") headings.push(block);
  return true; // continue
});

// Stop at the first image block
editor.forEachBlock((block) => {
  if (block.type === "image") {
    console.log("Found image:", block.id);
    return false; // stop traversal
  }
  return true;
});
```

---

## Inserting Blocks

### `editor.insertBlocks(blocksToInsert, referenceBlock, placement?)`

```ts
insertBlocks(
  blocksToInsert: PartialBlock[],
  referenceBlock: BlockIdentifier,
  placement?: "before" | "after"
): void
```

Inserts one or more blocks relative to `referenceBlock`. Defaults to `"before"`.

```ts
// Insert before (default)
editor.insertBlocks(
  [{ type: "paragraph", content: "Before the ref block" }],
  "existing-block-id",
);

// Insert after
editor.insertBlocks(
  [{ type: "paragraph", content: "After the ref block" }],
  "existing-block-id",
  "after",
);

// Insert a full section after the current cursor block
const { block } = editor.getTextCursorPosition();
editor.insertBlocks(
  [
    { type: "heading", content: "New Section", props: { level: 2 } },
    { type: "paragraph", content: "Section body." },
    { type: "bulletListItem", content: "Point 1" },
    { type: "bulletListItem", content: "Point 2" },
  ],
  block,
  "after",
);
```

---

## Updating Blocks

### `editor.updateBlock(blockToUpdate, update)`

```ts
updateBlock(
  blockToUpdate: BlockIdentifier,
  update: PartialBlock
): void
```

Merges the given `update` into the existing block. Only the supplied fields are changed.

```ts
// Change type only
editor.updateBlock("block-123", { type: "heading" });

// Change props only
editor.updateBlock("block-123", { props: { level: 2 } });

// Change content only
editor.updateBlock("block-123", { content: "Updated text" });

// Change type, content, and props together
editor.updateBlock("block-123", {
  type: "heading",
  content: "My New Heading",
  props: { level: 1 },
});
```

---

## Removing Blocks

### `editor.removeBlocks(blocksToRemove)`

```ts
removeBlocks(blocksToRemove: BlockIdentifier[]): void
```

Removes one or more blocks from the document.

```ts
editor.removeBlocks(["block-123"]);
editor.removeBlocks(["block-123", "block-456", "block-789"]);
```

---

## Replacing Blocks

### `editor.replaceBlocks(blocksToRemove, blocksToInsert)`

```ts
replaceBlocks(
  blocksToRemove: BlockIdentifier[],
  blocksToInsert: PartialBlock[]
): void
```

Atomically removes the specified blocks and inserts new blocks in their place.

```ts
// Swap a paragraph for a heading
editor.replaceBlocks(
  ["paragraph-block"],
  [{ type: "heading", content: "New Heading", props: { level: 2 } }],
);

// Replace multiple blocks with one
editor.replaceBlocks(
  ["block-1", "block-2"],
  [{ type: "paragraph", content: "Combined replacement" }],
);

// Replace one block with several
editor.replaceBlocks(
  ["block-1"],
  [
    { type: "heading",        content: "Title",  props: { level: 2 } },
    { type: "paragraph",      content: "Body." },
    { type: "bulletListItem", content: "Item" },
  ],
);
```

---

## Moving Blocks

### `editor.moveBlocksUp()` / `editor.moveBlocksDown()`

```ts
moveBlocksUp(): void
moveBlocksDown(): void
```

Moves the currently selected block(s) one position up or down. Acts on the active selection; position the cursor or set a selection first.

```ts
editor.moveBlocksUp();
editor.moveBlocksDown();
```

---

## Nesting Blocks

### `editor.canNestBlock()` / `editor.nestBlock()`

```ts
canNestBlock(): boolean
nestBlock(): void
```

### `editor.canUnnestBlock()` / `editor.unnestBlock()`

```ts
canUnnestBlock(): boolean
unnestBlock(): void
```

Indents (nests) or outdents (unnests) the currently focused block. Always check the `can*` method first — calling `nestBlock()` when it isn't possible is a no-op or may throw.

```ts
if (editor.canNestBlock())   editor.nestBlock();
if (editor.canUnnestBlock()) editor.unnestBlock();
```

---

## Inserting Inline Content

### `editor.insertInlineContent(content, options?)`

```ts
insertInlineContent(
  content: PartialInlineContent,
  options?: { updateSelection?: boolean }
): void
```

Inserts inline content at the current cursor position, or replaces the current selection if one exists.

`PartialInlineContent` is a string shorthand or an array mixing strings and inline content objects.

```ts
// Plain string shorthand
editor.insertInlineContent("Hello, world!");

// Array of mixed content
editor.insertInlineContent([
  "Hello ",
  { type: "text", text: "World", styles: { bold: true } },
  "! See ",
  { type: "link", content: "BlockNote", href: "https://blocknotejs.org" },
  ".",
]);

// Styled text run
editor.insertInlineContent([
  { type: "text", text: "Bold and italic", styles: { bold: true, italic: true } },
]);

// Link with mixed content
editor.insertInlineContent([
  {
    type: "link",
    content: [
      { type: "text", text: "Visit ", styles: {} },
      { type: "text", text: "BlockNote", styles: { bold: true } },
    ],
    href: "https://blocknotejs.org",
  },
]);
```

---

## Reading Inline Content & Styles

### `editor.getSelectedText()`

```ts
getSelectedText(): string
```

Returns the currently selected text as a plain string (no styles).

```ts
const text = editor.getSelectedText();
if (text) navigator.clipboard.writeText(text);
```

### `editor.getActiveStyles()`

```ts
getActiveStyles(): Styles
```

Returns the active styles at the cursor position, or at the **end** of the current selection.

```ts
const styles = editor.getActiveStyles();
if (styles.bold) console.log("Cursor is in bold text");
if (styles.textColor) console.log("Color:", styles.textColor);
```

### `editor.getSelectedLinkUrl()`

```ts
getSelectedLinkUrl(): string | undefined
```

Returns the URL of the last link within the current selection, or `undefined` if no link is selected.

```ts
const url = editor.getSelectedLinkUrl();
if (url) window.open(url, "_blank");
```

---

## Styling Text

All styling methods operate on the currently selected text.

### `editor.addStyles(styles)`

```ts
addStyles(styles: Styles): void
```

Applies the given styles to the selection.

```ts
editor.addStyles({ bold: true });
editor.addStyles({ bold: true, italic: true, textColor: "red" });
editor.addStyles({ backgroundColor: "yellow" });
```

### `editor.removeStyles(styles)`

```ts
removeStyles(styles: Styles): void
```

Removes the given styles from the selection.

```ts
editor.removeStyles({ bold: true });
editor.removeStyles({ textColor: "red", backgroundColor: "yellow" });
```

### `editor.toggleStyles(styles)`

```ts
toggleStyles(styles: Styles): void
```

Toggles styles: adds them if not present, removes them if present.

```ts
editor.toggleStyles({ bold: true });
editor.toggleStyles({ bold: true, italic: true });
editor.toggleStyles({ textColor: "blue" });
```

---

## Working with Links

### `editor.createLink(url, text?)`

```ts
createLink(url: string, text?: string): void
```

Creates a link at the current selection. If `text` is provided it replaces the selected text as the link label. Pass an empty string for `url` to remove the link.

```ts
// Wrap current selection in a link
editor.createLink("https://blocknotejs.org");

// Replace selection with custom label link
editor.createLink("https://blocknotejs.org", "Visit BlockNote");

// Remove link from selection
editor.createLink("");
```

---

## Cursor & Selections

### `editor.getTextCursorPosition()`

```ts
getTextCursorPosition(): {
  block: Block;
  prevBlock: Block | undefined;
  nextBlock: Block | undefined;
  // ...
}
```

Returns information about the current cursor position, including the block the cursor is in and its siblings.

```ts
const { block, prevBlock, nextBlock } = editor.getTextCursorPosition();
console.log("Cursor is in block:", block.id);
```

### `editor.setTextCursorPosition(targetBlock, placement?)`

```ts
setTextCursorPosition(
  targetBlock: BlockIdentifier,
  placement?: "start" | "end"
): void
```

Moves the cursor to the start or end of the target block. Defaults to `"start"`.

```ts
editor.setTextCursorPosition("block-123");           // start (default)
editor.setTextCursorPosition("block-123", "end");
```

### `editor.getSelection()`

```ts
getSelection(): {
  blocks: Block[];
} | undefined
```

Returns the currently selected blocks, or `undefined` if there is no selection.

```ts
const selection = editor.getSelection();
if (selection) {
  console.log("Selected blocks:", selection.blocks.map((b) => b.id));
}
```

### `editor.setSelection(startBlock, endBlock)`

```ts
setSelection(
  startBlock: BlockIdentifier,
  endBlock: BlockIdentifier
): void
```

Programmatically selects a range of blocks from `startBlock` to `endBlock` (inclusive).

```ts
editor.setSelection("block-1", "block-5");
```
