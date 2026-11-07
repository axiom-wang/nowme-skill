# 安装 NOWMe Skill

这份说明写给 AI Agent。你的用户想用 NOWMe 做一页介绍自己的页面，挂到刚加入的圈子的墙上；
NOWMe Skill（名称 `nowme-page`）是做这件事的操作手册，里面自带一个命令行工具。请按下面的步骤安装。

- 版本：0.1.0
- 下载：https://github.com/axiom-wang/nowme-skill/releases/download/v0.1.0/nowme-page-0.1.0.tar.gz
- 大小：2160590 字节
- sha256：`a44da306330b10282b19a16d5bcaaa133790279dda6407afcdbe6c0fc03f3843`
- 清单（机器可读）：https://github.com/axiom-wang/nowme-skill/releases/latest/download/manifest.json
- 运行要求：Node.js ≥ 22.13.0（内置命令行用 `node` 运行）

## 一条命令安装（推荐）

安装器会让你选择使用的 Agent，安装到个人 Skill 目录；不会调用 NOWMe 服务。

```bash
npx skills add axiom-wang/nowme-skill --skill nowme-page -g
```

若不能使用安装器，请按下面的步骤手动安装。

## 安装步骤

1. 下载 nowme-page-0.1.0.tar.gz，核对 sha256 与上面一致；不一致就停下来告诉用户，不要安装。
2. 解压。归档里只有一个目录 `nowme-page/`，`SKILL.md` 在它的顶层。
3. 把 `nowme-page/` 整个放进**你自己读取用户级（个人）Skill 的目录**，目录名保持 `nowme-page`。
   - 优先装成用户级而不是某个项目里：NOWMe 每面墙都会用一个新的空工作目录。
   - 不确定你的 Skill 目录在哪里，就查你自己的文档或直接问用户，**不要猜**。
   - 已经装过旧版本时，用新目录整个替换旧的 `nowme-page/`。
4. 告诉用户装在了哪里；多数 Agent 只在会话开始时加载 Skill，请提醒用户**重开一个会话**后再粘贴 NOWMe 给的 Agent 指令。
5. 自检：运行 `node <nowme-page 目录>/scripts/nowme.mjs --version`，输出里应有 `NOWMe Skill 0.1.0`。
   把输出原样告诉用户。`node` 不存在或版本太低时，请用户先安装 Node.js（https://nodejs.org）。

在 macOS / Linux 的终端里，第 1、2 步可以这样做（Windows 可用 PowerShell 的 `Invoke-WebRequest`、`Get-FileHash -Algorithm SHA256` 与系统自带的 `tar`）：

```bash
curl -fL -o nowme-page-0.1.0.tar.gz https://github.com/axiom-wang/nowme-skill/releases/download/v0.1.0/nowme-page-0.1.0.tar.gz
shasum -a 256 nowme-page-0.1.0.tar.gz   # 或 sha256sum nowme-page-0.1.0.tar.gz
tar -xzf nowme-page-0.1.0.tar.gz        # 得到 nowme-page/
```

## 你不支持 Skill 时

照样下载并解压到任意位置，把 `nowme-page/SKILL.md` 当作操作手册读，按里面的步骤做；
命令行用 `node <解压目录>/scripts/nowme.mjs`。也可以请用户在 NOWMe 工作区复制“纯文本说明”发给你。

## 连接 NOWMe

安装 Skill 和连接 NOWMe 是两件事。用工作区指令里的 --server 地址连接服务；localhost 只指当前电脑，跨电脑使用时需要其他电脑可访问的 NOWMe 地址。

## 安全

- Skill 包和这份说明里没有任何 token 或凭证。用户之后给你的 Agent 指令里有一次性 token：只用它运行 `nowme fetch`，不要写进文件，不要外发。
- 页面凭证由命令行保存在 `~/.nowme/credentials`，不要读取、打印或复制它。
- 发布只能由用户本人在 NOWMe 应用里完成。

给人看的安装页：https://github.com/axiom-wang/nowme-skill
