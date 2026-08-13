# Obsidian 知识库

基于 Obsidian 的个人知识库，采用"本地为主 + 多端实时同步 + 双云备份"架构。

## 目录结构

| 目录 | 用途 |
| ---- | ---- |
| `00-Inbox` | 收件箱：新笔记、未整理的临时内容先放这里 |
| `10-笔记区` | 主题笔记区，按需创建子目录（如 `日记`、`编程`、`读书`） |
| `90-Attachments` | 附件统一存放（图片、PDF 等） |
| `99-Templates` | 笔记模板（通用笔记、日记） |

## 同步与备份架构

- **实时同步**：本机（Windows）与 Android 设备均通过 **Remotely Save** 插件连接 VPS 上的 WebDAV 服务（HTTPS）。
- **备份归档**：本机通过 **Obsidian Git** 插件定时自动 commit + push 到 GitHub **私有仓库**（单向备份，不与同步混用）。
- **主节点约定**：git 只在本机运行，手机与 VPS 不跑 git，避免多端冲突。

## 本机初始化步骤（仅首次）

1. 用 Obsidian 打开本库（选择文件夹 `D:\Obsidian`）。
2. 确认 `社区插件` 已启用（设置 → 第三方插件 → 关闭"安全模式"），并启用 **Remotely Save** 与 **Obsidian Git**。
3. 配置 Remotely Save（设置 → Remotely Save）：
   - 远程服务选 **WebDAV**；
   - 填入 VPS 地址（如 `https://your-server/`）、用户名、密码（部署见 VPS 章节）；
   - 勾选启动时自动同步，点击"检查连接"验证。
4. 配置 Obsidian Git（设置 → Obsidian Git）：
   - 自动备份间隔设为 30 分钟（或按需），勾选"Vault 打开时自动拉取/提交"；
   - 首次需手动完成一次 push（见 GitHub 备份章节）。
5. 本库已初始化 git 仓库并完成首个提交，插件可直接使用。

## Android 设备（手机/平板）

1. 安装 Obsidian，打开本库（首次可选择从 WebDAV 恢复或复制已有文件）。
2. 安装并启用 Remotely Save，填写与电脑相同的 WebDAV 地址、账号、密码。
3. 设置同步间隔（如 15 分钟）或手动同步；将 Obsidian 加入系统"电池优化白名单"，保证后台同步稳定。
4. 手机端**不要**安装 git 相关插件，备份统一由电脑完成。

## GitHub 备份（首次）

1. 在 GitHub 创建**私有仓库**，例如 `obsidian-vault`。
2. 本机仓库内执行：

```bash
git remote add origin git@github.com:<你的用户名>/obsidian-vault.git
git push -u origin main
```

3. 配置 SSH 密钥（或 PAT）并开启 GitHub 双重认证。
4. 之后 Obsidian Git 插件会按间隔自动 commit + push。

> 注意：`data.json`（含同步密码/令牌）已被 `.gitignore` 排除，不会进入仓库。请勿把任何密码、令牌写入笔记。

## Codex 使用约定

- Codex 以 `D:\Obsidian` 为工作目录，只操作本库文件。
- 常规任务：批量整理/重命名、生成 MOC 与索引、按模板建笔记、统一 frontmatter、清理归档。
- 不修改 `.obsidian` 中的非必要配置；写文件遵循 `99-Templates` 模板与命名规范。
- 同步交给 Remotely Save、备份交给 Obsidian Git 插件自动完成。

## 安全提醒

- VPS 只开放 HTTPS 端口，WebDAV 使用独立账号与强密码。
- GitHub 仓库必须为 private，并开启双因素认证。
- 定期（如每月）从 GitHub clone 一次，验证备份可恢复。

## VPS 部署

VPS 端 WebDAV 的一键部署脚本与指南由 Codex 单独交付（位于 Codex 输出目录），按其中说明执行即可。