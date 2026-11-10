# dsh-plugin-dex-style-edit

> **This is a fixed build of [`XylaAlyx/dsh-plugin-dex-style-edit`](https://github.com/XylaAlyx/dsh-plugin-dex-style-edit)**, not an original plugin. All credit for the editing feature belongs to its author. The only change is the image-attachment rendering fix described below.

Codex-style message editing for DeepSeek Harness: edit an already-sent message and continue from there.

Click the pencil under any user message, change the text in place, press Enter. DSH session logs are append-only, so the plugin forks a new session at the turn *before* that message, sends your edited text as its opening message, and archives the session it replaced. Your files and working directory are left exactly as they are.

The model you pick in the composer before sending is the model that runs the edited message.

## What this build fixes

On DSH `0.2.1-rc.x`, an image you attached to a message **disappeared from the transcript** the moment the message was sent. It was still in the composer rail before sending and the model still received it — the bubble just stopped showing it.

The cause: this plugin shadows the shipped `key: 'user'` message renderer, and its replacement gated the attachment row on a `renderMessageImages` prop. The Chat plugin never supplies that prop — the node owner it passes is `{ groupPart, cwd, openFile, inspectCall, forkAt, loadImage, fileMentions, turnProcess }`. The gate was therefore always false, so the attachment row was always `null`:

```js
// upstream, lib/client.js
const renderMessageImages = props.renderMessageImages
...
attachments.length > 0 && typeof renderMessageImages === 'function' ? ... : null
```

This build renders the images itself from `props.loadImage`, the session-authorized loader the Chat plugin *does* pass:

- A lone image uses the same display rule as the shipped gallery (240px long edge, aspect clamped to 0.25–4, never upscaled); several images render as 64px tiles.
- A failed read now shows a **retry tile** instead of nothing.
- Text-only messages and the edit/branch actions are unchanged.

See [CHANGELOG.md](CHANGELOG.md) for the exact commit range.

<details>
<summary>中文说明</summary>

像 Codex App 一样编辑已发送的消息：点铅笔改气泡内容，**从这条继续**。

DSH 会话日志只追加、不可改写，所以实现方式是：在此消息之前那一轮处 fork 一个新会话，把改过的文本作为新会话的第一条消息直接发出，并归档被替换的原会话。工作目录与文件保持现状。

发送前你在输入框里选的模型，就是这条改过的消息实际使用的模型。

本版本修复了「发送后图片从对话中消失」的问题：插件替换了官方的用户消息渲染器，但原先用了一个 DSH 不会传入的 `renderMessageImages` 属性来判断是否显示附件，导致附件行始终为空。现在改为直接使用官方传入的 `loadImage` 自行渲染。

</details>

## Install

This build:

```sh
dsh plugin --profile desktop add github:Chernuga/dsh-dex-style-edit-fix
```

Upstream (if it has fixed this, prefer it):

```sh
dsh plugin --profile desktop add github:XylaAlyx/dsh-plugin-dex-style-edit
```

Then **restart DSH**. A pencil appears under every user message, and forked sessions carry a branch glyph in the sidebar.

Full notes (behaviour details, implementation, known limits): [docs/README.full.zh.md](docs/README.full.zh.md).

## License

MIT — see [LICENSE](LICENSE). Copyright belongs to the upstream contributors; this build only adds the fix described above.
