# Changelog

## [0.1.0] — 2026-11-10

首个版本，fork 自 [dsh-redo 0.4.3](https://github.com/Imzl-zl/dsh-redo)（MIT）。

### Added

- **生成中也能编辑 / 重新生成**：点按钮时若当前回合仍在运行，先调官方 `Session.cancel()` 停掉，轮询至会话空闲（最多 8 秒），再走上游原有的 rewind 流程。

### Changed

- 命令由 `/redo` 改为 `/relive`，客户端模块 id、样式 id、RPC 前缀改为 `redo-live:*`，以便与上游同时安装互不干扰。
- `engines.dsh` 下界从 `>=0.2.2-rc.1` 放到 `>=0.2.2-alpha.2`：实测该版本 `Session.readAttachment(ImageAttachmentRef)` 与 rc.1 一致。

### Docs

- README 增加「和上游的唯一区别」对比表、「你是不是在找这个」场景清单、工作流程图、FAQ、非官方分支声明。
- package.json keywords 扩充到 25 个，覆盖中英文检索词（edit while generating / edit while streaming / 生成中编辑 / 边生成边改 等）。
- 新增 `catalog-entry.yml`（插件市场索引条目），描述同时标注差异与检索词。

### Attribution

上游 `dsh-redo` 的会话改写、surface 替换、图片回填、确认流程均为 [@Imzl-zl](https://github.com/Imzl-zl) 所作，本仓库仅修改生成中状态的处理。
