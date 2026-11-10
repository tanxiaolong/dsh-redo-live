# dsh-redo-live

> 编辑已发送的消息并原地重答，**回答还在生成时也能改**（edit while streaming / edit while generating）。

<p>
  <img src="https://img.shields.io/badge/license-MIT-3da639" alt="License: MIT">
  <img src="https://img.shields.io/badge/DSH-%3E%3D0.2.2--alpha.2-4D6BFE" alt="DeepSeek Harness">
  <img src="https://img.shields.io/badge/platform-web-6B7280" alt="Platform: web">
  <img src="https://img.shields.io/badge/fork_of-dsh--redo-8B5CF6" alt="Fork of dsh-redo">
</p>

> **非官方分支声明**：本项目是个人维护的分支，fork 自 [dsh-redo](https://github.com/Imzl-zl/dsh-redo)（作者 [@Imzl-zl](https://github.com/Imzl-zl)）。与原作者无隶属关系，也不代表其立场。上游选择"生成中禁用"，本分支选择"生成中先停再改"，两者是不同的取舍，不是谁修 bug。

---

## 你是不是在找这个？

如果你有过下面任何一个念头，这个插件就是为你做的：

- 「它都已经答错了，**为什么还不能改**？非要等它答完、再点停止、再点编辑？」
- 「**边生成边改**行不行？我现在就想把那条消息换掉。」
- 「Codex / ChatGPT 里点铅笔就能改，DSH 为什么要等？」
- 「回答流到一半我发现问错了，**打断 + 改 + 重发**能不能一步到位？」
- 「我不想因为一句话打错就新开一个会话。」

上游 `dsh-redo` 能做到"改完重发"，但**要求你先自己停掉正在生成的回答**。这个分支把这一步自动化了。

## 和上游 dsh-redo 的唯一区别

| | [dsh-redo](https://github.com/Imzl-zl/dsh-redo) | **dsh-redo-live**（本包） |
|---|---|---|
| 空闲时编辑 / 重新生成 | ✅ | ✅ |
| **回答正在生成时编辑** | ❌ 按钮置灰，要你先点停止 | ✅ **按钮可用，自动替你停** |
| 停止由谁触发 | 你手动点 | 插件调官方 `Session.cancel()`，等会话空闲（最多 8 秒）后继续 |
| 命令 | `/redo` | `/relive`（可与上游同时安装） |
| 会话改写方式 | 追加式 surface replace | 完全相同（沿用上游实现） |
| 图片随行、二次确认、版本化 | ✅ | ✅ 相同 |

一句话：**上游 = 你自己先停；本包 = 点编辑时它先替你停。**

## 工作方式

点击某条用户消息上的铅笔（或点"重新生成"）时：

```
按钮 → 会话还在跑？
        ├─ 是 → Session.cancel()  ← 官方停止入口，不是 hack
        │        └→ 每 100ms 轮询，直到真正空闲（上限 8 秒）
        │             ├─ 成功 → 继续
        │             └─ 超时 → 报错，不改写任何东西
        └─ 否 → 继续
        ↓
      /relive <消息序号> 武装 → 原文回到输入框
        ↓
      你改 → 点发送 → 从那条消息原地重写对话
```

停止用的 `Session.cancel()` 是 DSH 官方客户端接口，不是绕过会话日志的写操作；改写仍然走上游的 append-only surface replace。

## 功能

- **生成中编辑**：回答还在流式输出时，直接点你那条消息的铅笔。
- **生成中重新生成**：不编辑，直接把这一轮停掉、用原话重答。
- **编辑重发**：消息连同它之后的回答离开对话与模型上下文，文字和图片回到输入框；改完发送，新内容落在原位，不多出一条。
- **图片跟着走**：存图读回字节、重新挂成草稿，发送时一起上送。
- **破坏性操作有确认**：撤回中段消息前告诉你后面有几轮会离开上下文；输入框已有内容时再问一次是否替换。
- **版本化 surface**：改写以追加式 surface replace 实现，不删原始日志、不破坏 `/compact`。

## 安装

```sh
dsh plugin add dsh-redo-live
```

或在插件市场 / Discover 里搜 `dsh-redo-live` / `生成中编辑`。装完**重启 DSH**（客户端模块需要重新加载）。

要求 DSH `>= 0.2.2-alpha.2 < 0.3.0`，Web GUI 使用。发布物已含构建好的 `lib/`，不需要自己 build。

## 命令

```
/relive <消息序号>     # 武装：下一条发出的消息替换掉那一条及其之后
/relive off           # 取消武装
```

按钮是主入口，命令给键盘流用户和脚本用。

## 常见问题

**Q：会让正在生成的回答断掉吗？**
会，而且这是刻意的——你要改的就是这一轮的输入，不停就没法改。停止走官方 `Session.cancel()`，不会留下半截写入。

**Q：会不会破坏会话 / 让 `/compact` 出问题？**
改写逻辑完全沿用上游 dsh-redo：append-only surface replace，不删日志、不新增压缩债。上游在这一点上做得比自己修一遍更稳，所以这段一行没动。

**Q：能和上游的 dsh-redo 同时装吗？**
能。命令（`/relive` vs `/redo`）、客户端模块 id、样式 id、RPC 前缀都做了隔离。但一般没必要——本包是上游的超集。

**Q：停止失败怎么办？**
超时（8 秒）会直接报错并放弃，不会在不空闲的会话上强行改写。

**Q：为什么不自带"停止后自动发送"？**
那会把"停止"和"改写"两步合并成一次不可撤销的操作。这里保留你确认的机会——原文回填后你自己决定改不改、发不发。

## 兼容性说明

上游声明 `>=0.2.2-rc.1`，理由是"第一个带完整 `ImageAttachmentRef` 读取 API 的版本"。实测 `0.2.2-alpha.2` 的 `Session.readAttachment(attachment: ImageAttachmentRef)` 签名与 rc.1 **完全一致**，`@deepseek-ai/dsh-session` 两版逐字节相同，所以本包下界放在 `>=0.2.2-alpha.2`，比上游宽松一档。

## 已知边界

- 只处理纯文本与图片；带其它文件附件的消息不能编辑或重试（上游同）。
- 会话第一条消息不能改写（protected session head）。
- 停止超时或失败会明确报错，不会静默改写。

## 致谢

- **[@Imzl-zl](https://github.com/Imzl-zl)** —— 上游 [dsh-redo](https://github.com/Imzl-zl/dsh-redo) 的作者。本项目的会话改写、surface 替换、图片回填、确认流程、版本化处理全部来自他的工作，我只改了"生成中"这一道门。
- 上游项目：https://github.com/Imzl-zl/dsh-redo

## License

MIT。原始版权归 Imzl-zl，修改部分归 tanxiaolong，详见 [LICENSE](LICENSE)。
