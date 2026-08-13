# Obsidian 知识库

基于 Obsidian 的个人知识库，采用"双电脑 VPS 中转同步 + GitHub 自动备份"架构。

## 目录结构

| 目录 | 用途 |
| ---- | ---- |
| `00-Inbox` | 收件箱：新笔记、未整理的临时内容先放这里 |
| `10-笔记区` | 主题笔记区，按需创建子目录（如 `日记`、`编程`、`读书`） |
| `90-Attachments` | 附件统一存放（图片、PDF 等） |
| `99-Templates` | 笔记模板（通用笔记、日记） |

## 同步与备份架构

- **实时同步**：电脑 A（本机）与电脑 B（工作机）均通过 **Remotely Save** 插件直连 VPS 上的 WebDAV 服务（HTTPS），两台电脑互不直接依赖 GitHub。
- **备份归档**：VPS 的数据目录同时是 git 仓库，由 systemd 定时器每 30 分钟自动 commit + push 到 GitHub **私有仓库**（`my_obsidian`）。
- **推送职责**：GitHub 的推送**唯一由 VPS 完成**；电脑 A 保留本地 git 历史但已停止 push，电脑 B 不装 git 插件。

## 电脑 A（本机）状态

- 已修复并启用 Remotely Save（0.5.25），插件 `data.json` 已预置忽略规则：
  - `.git`、`.trash`、`.obsidian/workspace*.json`、`.obsidian/plugins/*/data.json`
- 首次使用：设置 → Remotely Save → 远程服务选 **WebDAV**，填入 VPS 地址、账号 `obsidian`、密码（见 VPS 上 `/etc/obsidian-webdav/credentials.txt`），点击"检查连接"并做一次全量同步。
- git：本机保留 `.git` 历史用于本地回滚，已移除 GitHub remote；如需本地提交可用 Obsidian Git 插件的"Commit"命令，但**不要 push**（避免与 VPS 备份冲突）。

## 电脑 B（工作机，国内网）首次配置

1. 安装 Obsidian，新建空库，路径 `D:\Obsidian\myObsidian`。
2. 从电脑 A 复制整个 `remotely-save` 插件目录到 B 的 `.obsidian\plugins\remotely-save`（B 访问 GitHub 慢，不自行下载），并确认 `.obsidian\community-plugins.json` 含 `"remotely-save"`。
3. 打开 Obsidian 启用 Remotely Save，填与 A 相同的 WebDAV 地址、账号、密码。
4. 首次打开自动全量下载 VPS 上的文件。**不要**安装 Obsidian Git，也不要初始化 git 仓库。

## GitHub 备份（VPS 负责）

- VPS 上已运行 `deploy-git-sync.sh`：数据目录 `/srv/obsidian-vault` 内每 30 分钟自动 `git add -A && git commit && git push origin main`。
- 手动备份：在 VPS 执行 `sudo bash /usr/local/bin/obsidian-git-sync.sh`。
- 查看备份日志：`journalctl -u obsidian-git-sync -n 30 --no-pager`。

> 注意：`data.json`（含同步密码/令牌）与 `workspace*.json` 已由 `.gitignore` 和 Remotely Save 忽略规则排除，不会进入仓库。请勿把任何密码、令牌写入笔记。

## Codex 使用约定

- Codex 以 `D:\Obsidian\myObsidian` 为工作目录，只操作本库文件。
- 常规任务：批量整理/重命名、生成 MOC 与索引、按模板建笔记、统一 frontmatter、清理归档。
- 不修改 `.obsidian` 中的非必要配置；写文件遵循 `99-Templates` 模板与命名规范。
- 同步交给 Remotely Save、GitHub 备份交给 VPS 自动完成。

## 安全提醒

- VPS 只开放 HTTPS 端口，WebDAV 使用独立账号与强密码。
- GitHub 仓库必须为 private；PAT 使用 fine-grained 且仅授权该仓库。
- 定期（如每月）从 GitHub clone 一次，验证备份可恢复。
- VPS 部署脚本与详细指南由 Codex 交付（`Obsidian-VPS-Deploy` 目录），按其中说明执行。