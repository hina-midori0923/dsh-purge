<p align="center">
  <img src="docs/banner.svg" alt="dsh-purge" width="720">
</p>

<h1 align="center">dsh-purge — DeepSeek Harness 破甲（维护分支）</h1>

<p align="center">
  <strong>版本 1.1.73</strong> · 适配 DSH 0.2.x · 维护者
  <a href="https://github.com/hina-midori0923">hina-midori0923</a>
</p>

> **这是继续维护的 fork。** 原作者仓库 `YuJunZhiXue/dsh-purge` 和 npm 包都已不存在
> （2026-10-22 核实）。本仓库在此基础上加入 DSH 0.2.1 兼容修复（新版宿主不再注册
> `{{cwd}}` 提示词变量，会打断提示词组装），并作为后续安装与更新的唯一来源。
> 来源、备份与差异记录见 [MIRROR-NOTICE.md](MIRROR-NOTICE.md)。

---

## 这是什么

`dsh-purge` 是给官方 **DeepSeek Harness** 用的本地插件。它不改模型，改的是宿主在模型
前面塞的东西：系统提示词、身份段、拒答/审批措辞，以及 DSH 自带的一批默认策略。

点过 **「应用」** 之后，Harness 跑的是你的提示词而不是它自己的：官方身份被清掉，插件的
操作提示词在每次组装时注入，可选的代码级补丁会调整沙箱、审批、权限等默认值。宿主原有
的工具、技能、MCP 连接保持可用。

支持官方**桌面端 EXE**（已在 `0.2.1-rc.2` 上验证）和官方 **`dsh web`**。两边分开安装、
分开应用 —— Web 装完不会影响桌面端。

## 功能一览

| 区域 | 作用 |
|---|---|
| **dsh-purge 停靠栏** | 会话标题旁按钮，打开右侧停靠栏，含「清洗」和「演练台」两页 |
| **清洗** | 按组显示补丁状态、**应用** / **恢复** / **卸载**、提示词编辑器、多套规则集 |
| **提示词注入** | 每次组装都注入你的提示词（prompt-inject.md / 规则集），清掉官方身份与拒答措辞 |
| **宿主策略** | 可选的代码级补丁，调整沙箱、审批、权限与工具提示的默认行为 |
| **演练台** | 红队流程：预设、角色稿、技能与资产库，仅用于本地/离线或书面授权的目标 |
| **启动时** | 自动检查并重打补丁；可选拒答自动重试、可选自动续写 |

插件不读取宿主磁盘上的提示词文件；身份来自插件内置的加密提示词，也就是提示词框默认
显示的那一份。

## 环境要求

- DeepSeek Harness **0.2.x**（桌面端 `0.2.1-rc.2` 或官方 `dsh web`）。`0.1.x` 老宿主
  要用旧版本插件线，别拿这一版去套。
- Node.js **≥ 18**（跑 CLI / 补丁脚本用）。Windows 无闪窗需要 **Node ≥ 22**。
- `$DSH_HOME`（一般是 `~/.dsh`），不写死盘符。

## 安装（本仓库）

```sh
# 官方 dsh web
dsh plugin --profile web add github:hina-midori0923/dsh-purge

# 官方桌面端 EXE
dsh plugin --profile desktop add github:hina-midori0923/dsh-purge
```

`dsh` 不在 PATH 时用 DSH 自带的 CLI，或走手动安装：

1. 把本仓库克隆到 `$DSH_HOME/plugins/dsh-purge`（目录名必须是 `dsh-purge`，已有克隆也行）：
   ```sh
   git clone https://github.com/hina-midori0923/dsh-purge.git "$DSH_HOME/plugins/dsh-purge"
   ```
2. 在你用的 profile（`$DSH_HOME/profiles/web/package.json` 或 `.../desktop/package.json`）里：
   `dependencies` 加 `"dsh-purge": "file:../../plugins/dsh-purge"`，
   `dsh.profile.bundles` 末尾加 `"dsh-purge"`。
3. 只在该 profile 目录里跑 `pnpm install`。
4. 重启 DSH。

GitHub 拉取不稳时可用 tarball：
`https://github.com/hina-midori0923/dsh-purge/archive/refs/heads/master.tar.gz`。

## 第一次使用

1. 完全退出并重新打开 DSH（桌面端：退出托盘后再开 EXE）。
2. 打开任意会话，点会话标题旁的 **dsh-purge**。
3. 在「清洗」页点 **应用**。应用成功会自动重启宿主；补丁没打完则不重启。
4. 重启后**新开一轮**对话 —— 注入才进当前会话。

停靠栏没出现，说明插件不是从本仓库加载的，回头检查 profile 的
`dependencies` / `dsh.profile.bundles`。

## 更新

插件自带的「检查更新」看的是本 fork 的 tag（`lib/update.js` → `hina-midori0923/dsh-purge`）。

- 插件内：「清洗」页 → 检查更新。
- 命令行：`dsh plugin --profile desktop update dsh-purge`（`web` 同理），然后重启。
- 维护者发版：改 `package.json` 的 `version` → push `master` → 推 `vX.Y.Z` tag。已安装的
  副本按 git 提交解析，更新会重新解析 `master`；升版本号是为了让界面上的变化一目了然。

## 恢复与卸载

- 「清洗」页的 **恢复** 会从插件留的备份还原被改过的宿主文件。
- **卸载** 会把插件移出 profile 并重新应用原件。
- Agent 工具：`purge_status` 查看补丁/shim 状态，`purge_apply` 应用补丁集，
  `purge_revert` 执行恢复。
- `$DSH_HOME/prompt-inject.md` 是用户数据，恢复/卸载都不会动它。

## 兼容性说明

- 宿主补丁按 DSH 0.2.x 的文件布局编写。DSH 升级后，原文对不上的补丁会显示**跳过**并保持
  原样，「应用」仍算完成。
- 宿主补丁写在解包后的 `app` 目录。若安装目录里仍有 `app.asar`，状态页会提示当前进程读
  的是归档而不是补丁文件 —— 提示词注入（运行时段）照常生效，代码级补丁按状态页提示处理。
- pnpm/npm 升级时可能覆盖 `node_modules`；插件在启动时自动重打，不需要手改安装文件。
- 路径不写死：依次找 `$DSH_HOME`、启动器旁边的 `.dsh`、再退回 `~/.dsh`。
- 只处理本机官方 `@deepseek-ai` 包和本机配置；不是公网扫描器，也不带针对第三方站点的载荷。

## 来源与许可

- 原项目：**dsh-purge**，作者 **YuJunZhiXue**（原仓库 `YuJunZhiXue/dsh-purge`，已于
  2026-10-22 核实删除）。
- 备份/镜像记录以及本 fork 做过的差异：[MIRROR-NOTICE.md](MIRROR-NOTICE.md)。
- 许可：**MIT**（见 [LICENSE](LICENSE)，保持原样）。这里保留原作者署名；如果你再分发或
  基于本 fork 继续开发，请保留许可证与署名。
