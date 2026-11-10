# Changelog

## Unreleased — image attachments no longer disappear from the transcript

Forked from [`XylaAlyx/dsh-plugin-dex-style-edit@1299ac4`](https://github.com/XylaAlyx/dsh-plugin-dex-style-edit/commit/1299ac4).

### Fixed

**Sending a message with an attached image made the image vanish from the transcript.**

The image was present the whole time: the composer rail displayed it before sending, the
Host stored it in the attachment object store, and the model received it. Only the
transcript bubble stopped rendering it — silently, with no error tile.

Root cause: this plugin shadows the shipped `key: 'user'` renderer
(`conversation.chat.node`, priority `-1`), and its replacement gated the attachment row on
`props.renderMessageImages`. The Chat plugin does not pass that prop. The owner object
`ChatNodeSeat` passes to a keyed node renderer is:

```js
{ ...groupPart, cwd, openFile, inspectCall, forkAt, loadImage, fileMentions, turnProcess, node }
```

`renderMessageImages` is not among them, so `typeof renderMessageImages === 'function'` was
always `false` and the attachment `<div>` was always `null`.

### Changed

- `lib/client.js` now renders message images itself from `props.loadImage`, the
  session-authorized loader (with a synchronous `peek`) that the Chat plugin does pass.
  The shipped `MessageImage` component is private to
  `@deepseek-ai/dsh-client-ui-attachment` and reachable only through the
  `conversation.chat.user.images` slot, which a plugin cannot render for a node renderer
  it replaces — hence a local component instead of a call into the shipped one.
- A lone image uses the shipped gallery's sizing rule (long edge 240px, aspect ratio
  clamped to [0.25, 4], cropped by `object-fit: cover`, never upscaled). Two or more
  images render as 64px tiles.
- A failed read now renders a clickable retry tile rather than collapsing to nothing.
- Added a `data-message-attachments` marker on the attachment row, matching the shipped
  renderer.

### Unchanged

- Editing, forking, branching, the session fork badge, and the model-selection handoff are
  untouched.
- Text-only messages render exactly as before.

### Verified

Rendered under a React double with the buggy prop absent (as in the real Chat owner):

- one 501×102 image → one frame sized 240×60 with the loaded URL in `src`;
- two images → two 64px tiles;
- a text-only message → no attachment row;
- the message text still flows through `conversation.user.text`.

Verified in a live DSH `0.2.1-rc.5` session: a sent image renders above the bubble instead
of disappearing.
