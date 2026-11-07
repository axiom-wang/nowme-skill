---
name: nowme-page
description: 为 NOWMe 社区墙做一页“有个性的自己”。当用户输入 `/nowme-page <wallId> <token> --server <url>`、粘贴 NOWMe 应用里给出的 Agent 指令（wal_ 开头的墙 ID 加 nmt_ 开头的一次性 token），或说要做 / 修改自己在 NOWMe 墙上的页面时使用。流程：拉取墙的情境 → 按场景做一次很短的采访 → 主人确认 brief → 用 NOWMe Kit 生成页面 → 校验与截图自查 → 按主人意见修改 → 提交（发布由主人在应用内完成）。
---

# NOWMe 页面 Skill

你在帮**主人**（正在和你对话的这个人）做一页个人页面，挂在他刚加入的一个圈子（黑客松、兴趣社群、社团……）的共同的墙上。别人在墙上看到缩略卡，点进来看这一页，再点平台提供的“想聊这个”去找主人。

调用方式：

```
/nowme-page <wallId> <token> --server <url>
```

例如 `/nowme-page wal_xxxx nmt_xxxx --server http://localhost:3000`。token 只能换一次；换过以后，同一个工作目录里直接用 `nowme fetch`（不带参数）刷新。

Skill 版本：0.1.0。最新版本与安装说明在 https://github.com/axiom-wang/nowme-skill （可跨电脑安装）；`nowme fetch` 发现有新版本时会提示，请转告主人更新。

本文件是操作手册，按第 0–7 步顺序做。细节放在 `reference/` 里，用到时再读：

| 文件 | 什么时候读 |
| --- | --- |
| `reference/interview.md` | 第 2–3 步：采访话术、编号选择题的格式、brief 确认、画像写回 |
| `reference/page-contract.md` | 第 4–5 步：页面输出契约全文、HTML 骨架、card / manifest、校验错误怎么修 |
| `reference/kit.md` | 第 4 步：NOWMe Kit 的变量、主题、组件、`window.nowme` API 与代码片段 |
| `reference/design-principles.md` | 第 4 步：设计构想怎么写、按场景怎么做、怎样避免 AI 模板感 |
| `reference/self-check.md` | 第 5 步：截图自查清单 |
| `reference/editing.md` | 第 6 步：按区块修改、不支持的请求与替代方案 |
| `scenes/<sceneType>.json` | 第 1 步起：这面墙的场景指南（采访题目、必须有的信息、接话点类型、页面目标） |
| `examples/index.json`、`examples/<目录>/` | 第 2 步给主人选风格；第 4 步参考结构与 `DESIGN.md` |
| `templates/brief.md`、`templates/profile.md` | 第 3 步：brief 与长期画像的格式 |

下文的路径如果没有特别说明，`scenes/`、`reference/` 等都相对于本文件所在的目录。

## 始终遵守

1. **只用主人给的信息**：本次采访的回答、主人主动交给你的文件和图片、长期画像里“主人明确表达 / 主人确认”的记录。其余一概不用（见“隐私规则”）。
2. **不编造**：经历、数字、引语、奖项、学校、链接都不能编；不知道的写进 brief 的“未知与不要猜”，页面上就不出现。
3. **主人确认 brief 之后才生成**；页面上的每一句话都能在 brief 里找到出处。
4. **发布只能由主人在应用内完成**。你能做的最后一步是 `nowme submit`。
5. **凭证不外露**：不打印、不读取、不复制 `~/.nowme/credentials`；token 和凭证不写进任何文件（尤其是 `page/`）。
6. **说真话**：做不到的效果直接说做不到，并给替代方案；不要声称做了没做的事。

## 第 0 步：准备

### 0.1 确认 `nowme` 命令

按顺序找，用第一个能输出版本号的：

- **方式 A（推荐，随 Skill 一起安装）**：本 Skill 目录里自带打包好的命令行 `scripts/nowme.mjs`，只需要 Node ≥ 22.13：
  ```bash
  node "<本 Skill 目录>/scripts/nowme.mjs" --version
  ```
  输出形如 `nowme 0.1.0 · NOWMe Skill 0.1.0`。它的 `preview` 由 NOWMe 服务端渲染截图（本机不需要浏览器），版本与本 Skill 一致。`node` 不存在或版本太低时，请主人先安装 Node 22（https://nodejs.org）。
- **方式 B（已全局安装）**：`nowme --version` 能输出版本号。
- **方式 C（NOWMe 开发者在本地仓库里运行）**：主人会告诉你 NOWMe 仓库的路径 `<仓库路径>`，用
  ```bash
  pnpm --dir <仓库路径> nowme <命令> [参数]
  ```
  pnpm 会在仓库根目录里运行命令，所以：**所有路径参数写绝对路径**（例如 `"$PWD/page"`），并且始终在第 0.2 步的工作目录里执行。CLI 通过 pnpm 设置的 `INIT_CWD` 知道你执行命令时所在的目录，`nowme.json`、`context.json` 会写在那里。第一次先运行 `pnpm --dir <仓库路径> nowme status`，确认它显示的工作目录就是当前目录；如果不是，改用等价的直接调用（保留当前目录）：
  ```bash
  "<仓库路径>/node_modules/.bin/tsx" --conditions=source "<仓库路径>/packages/cli/src/bin.ts" <命令> [参数]
  ```

以下统一写作 `nowme <命令>`，替换成你确定的那种调用方式。

### 0.2 新建工作目录

为这面墙新建一个**空目录**并进入，例如 `~/nowme/<墙名或 wallId>/`。不要在主人的其他项目目录里工作（避免读到无关文件）。目录结构（contracts 1.5）：

```
<工作目录>/
├── nowme.json      # nowme fetch 写入：{ server, wallId, pageId }，不含密钥
├── context.json    # nowme fetch 写入：墙的情境
├── brief.md        # 第 3 步你写的 brief，只在本地，永不上传
└── page/           # 第 4 步生成的页面（只有这里的白名单文件会被提交）
~/.nowme/profile.md     # 长期画像，只在本地，永不上传
~/.nowme/credentials    # 页面凭证，由 nowme 管理；你不要读、不要打印
```

已经有 `nowme.json` 的目录说明之前做过：直接 `nowme fetch` 刷新情境，读已有的 `brief.md` 和 `page/`，问主人这次是“修改”还是“重新做”。

## 第 1 步：换取凭证，拉取墙的情境

```bash
nowme fetch <wallId> <token> --server <url>
```

- 成功后读 `context.json`：`wall.name`、`wall.description`、`wall.sceneType`、`wall.organizerNote`、`kit.fonts`（可用字体）、`limits`（体积等限制）。
- 读场景指南 `scenes/<wall.sceneType>.json`。之后的采访题目、必须有的信息、接话点类型、页面目标都以它为准。
- 如果 `~/.nowme/profile.md`（或 `$NOWME_HOME/profile.md`）存在，读它。只采用“主人明确表达 / 主人确认”的记录；“模型暂定”的只能在采访里向主人求证，不能当事实用。只读这一个文件，不要去找别的。
- 失败时：`TOKEN_INVALID` 说明 token 已用过、过期或不属于这面墙——请主人在应用内这面墙的“我的页面 → 创作工作区”重新生成指令；`CREDENTIAL_INVALID` 说明连接被断开或墙已关闭，同样需要重新生成指令。网络错误可以重试一次，再失败就把命令输出原文告诉主人。

## 第 2 步：采访（详见 `reference/interview.md`）

目标：用最少的来回拿到场景指南 `mustHave` 里的信息，加上 1–2 个只属于主人的具体细节。按场景的 `interviewBudgetSeconds` 控制节奏（黑客松约 90 秒，兴趣社群约 2 分钟，社团约 3 分钟）。

1. **先收素材**：用场景指南的 `assetPrompt` 请主人贴现成的东西（自我介绍、作品链接、截图、照片）。从中抽取第一批事实候选，后面就不再问已经知道的。
2. **编号选择题**，一条消息发完，主人回复编号即可（例如 `1a 2bd 3a`）：
   - 场景指南 `choiceQuestions` 里的题目，按顺序；标“必答 / 重要”的在题目里写明。
   - **风格**：列出 `examples/index.json` 里 6 个示例的一句话风格描述，请主人选 1–2 个，或“你来定”。
   - **动效**：三档——安静（calm）/ 适中（medium）/ 活泼（lively）。
   - **边界**：场景指南里 `boundaries` 那道多选题；标“默认勾选”的选项预先选中，主人只需增减。
   - 黑客松：素材提示和选择题放在**同一条消息**里，节省时间。
3. **追问**：最多 `maxFollowUps` 个（黑客松只有 1 个），只问会明显改变页面、主人又容易回答的问题；每个追问都给 2–4 个回答选项并允许“跳过”。已经知道的不要再问。
4. **判断够不够**：对照 `mustHave` 逐项检查。“够了，去生成”始终可选：主人这么说而必须有的项还缺时，说明缺哪一项、为什么需要（例如“黑客松的首屏要写你想找什么样的队友，不然别人不知道该不该来找你”），主人仍要继续就用现有信息生成，缺的部分**不出现**，绝不编造。

## 第 3 步：brief 与“AI 眼中的你”

1. 按 `templates/brief.md` 写 `<工作目录>/brief.md`（不要放进 `page/`）。
2. 给主人看**几句通顺的话**——“AI 眼中的你”（3–5 句，尽量用主人自己的说法），再列出 2–3 个**接话点候选**（编号，类型取场景指南 `hookKinds`）和要显示 / 不显示的信息。请主人回复：`确认`、`划掉 2`、`第 1 句改成……`。
3. 按主人的修改更新 brief，直到主人确认。**确认之前不生成。**
4. **写回画像**：把这次新确认的**长期**信息（技能、作品、兴趣、经历、原话、主人说明了范围的边界）按 `templates/profile.md` 的格式写回 `~/.nowme/profile.md`，每条带来源、确认状态、适用范围、更新时间。写之前在同一条消息里告诉主人会记哪几条，主人说“不记”就不写。**本页意图不写回**；表达偏好只有主人明确说“以后都这样”才写。

## 第 4 步：设计构想与生成

1. **设计构想**：先用一段话写下这页想给人什么感觉、核心视觉是什么（通常是 brief 的“重点细节”）、区块怎么安排（列出 `data-nm-id`）、用哪些 Kit 组件、动效档位。写在回复里，不必等主人批准。参考 `reference/design-principles.md`。
2. **选起点**：主人选了示例就读对应的 `examples/<目录>/DESIGN.md` 与 `index.html`，借它的**结构和手法**，内容全部换成主人的；也可以完全自由创作。不要把示例里虚构人物的任何文字留在页面上。
3. **生成 `<工作目录>/page/`**（完整规则见 `reference/page-contract.md`）：
   - `index.html`：自包含单文件；`<head>` 里引用 `/_nowme/kit/v1/nowme.css` 与 `/_nowme/kit/v1/nowme.js`；`<html lang="zh-CN" data-nm-motion="<动效档位>">`；至少 3 个 `data-nm-id` 区块；外链只用 `data-nm-href="https://..."` 或 `nowme.openLink(...)`；不联网、不存储、不用表单、不嵌 iframe；字体只用 Kit 清单里的，中文字体（按字重计）最多 2 款；脚本动效检查 `nowme.reducedMotion`；体积 ≤ 300 KB。
   - **不要把联系方式（微信号、手机号、邮箱）写进页面**：平台在页面外提供“想聊这个”，联系方式由主人在应用内设置是否公开。
   - 图片只用主人给的（复制进 `page/assets/`，png / jpg / webp / gif，单个 ≤ 2 MB），或你用内联 SVG / Canvas 画的装饰；不要用网图。不要放 `cover.png`，它由 `nowme preview` 生成。
   - `card.json`：`displayName`（按边界用真名或昵称）、`tagline`（≤ 40 字的一行介绍）、`hooks`（2–3 个，来自主人确认的接话点候选，`kind` 取场景指南的类型，`opener` 是访客口吻的开场白）。
   - `manifest.json`：`sceneType` 取 `context.json`、`kitVersion: "1"`、`motionLevel`、`fonts`、`extraChars: ""`、`blocks`（与 `data-nm-id` 一致）、`generator: { "agent": "<你的名字>", "skillVersion": "0.1.0" }`。
4. 写完后上报：`nowme event generate`（只上报事件，不带内容；失败不影响后续）。

## 第 5 步：校验与截图自查

1. `nowme validate page`：逐条按报告里的 `hint` 修复，重复直到 0 个错误。警告也尽量处理；`EXTRA_CHARS_MISSING` 会被自动补全。错误码对照见 `reference/page-contract.md`。
2. `nowme preview page`：在和平台一样的沙箱里渲染（方式 A 的命令行把页面发给 NOWMe 服务端渲染，服务端不保存、不生成版本；方式 B、C 在本机渲染，也可以加 `--remote` 改由服务端渲染），报告渲染问题（脚本报错、CSP 违规、加载慢、空白），并做与提交后一致的字体子集化。`page/` 里还没有 `cover.png` 时会把封面（页面顶部 375×500，存成 750×1000）写进去；之后改了首屏要重新生成封面就加 `--cover`。其他截图（封面副本、整页、减少动态效果的整页、按 `data-nm-id` 切开的区块图）和 `report.json` 写在工作目录的 `.nowme-preview/` 里（不会进 `page/`），命令会打印每张图的绝对路径。
3. **看截图**，按 `reference/self-check.md` 逐项检查，至少做一轮“截图 → 审视 → 修改 → 再截图”。你看不了图片时，如实告诉主人，请他打开 `.nowme-preview/` 里的截图（方式 B、C 也可以用 `nowme preview page --serve` 在浏览器里看），再按他的反馈改。

## 第 6 步：交给主人，按意见修改（详见 `reference/editing.md`）

1. 把 `.nowme-preview/` 里的截图给主人看（方式 B、C 可以运行 `nowme preview page --serve`，默认 `http://localhost:4100/`，端口被占用时加 `--port <端口>`，把地址给主人；方式 A 也可以先 `nowme submit`，让主人在工作区里预览这个版本）；用两三句话说明你的设计构想，以及缩略卡上的一行介绍和接话点。
2. 主人提修改时，先确定改哪个区块（`data-nm-id`），**只改那个区块**，其他部分保持不变。范围不清时先问（例如列出区块编号让主人选）。
3. 主人在修改时补充的新信息属于“主人明确表达”，可以直接用；其中的长期信息按第 3 步写回画像。
4. 每次修改后重新 `validate` 与 `preview`。做不到的请求如实说明，并给替代方案。

## 第 7 步：提交

```bash
nowme submit page
```

- 命令会先在本地校验，再只上传白名单文件（`index.html`、`card.json`、`manifest.json`、`cover.png`、`assets/`），并打印版本号、校验结果和创作工作区的链接。
- 告诉主人：**“去应用里的创作工作区确认缩略卡（一行介绍、接话点和开场白）、联系方式和公开范围，然后点发布。我没有办法、也不应该替你发布。”**
- 被服务端拒绝（422）时，按返回报告里的 `hint` 修复后重新提交；之前通过的版本和已发布的页面不受影响。
- 以后再改：修改 → `validate` → `preview` → `submit`，每次提交都是一个新版本，需要主人在应用内重新确认发布。
- 提交后可以上报 `nowme event submit`（CLI 已自动上报时可省略）。

## 隐私规则（brief-collection 第 6 节）

- **不主动翻找**主人的文件、聊天记录、其他项目、浏览器、通讯录；**不使用**你自己已有上下文里关于主人的任何信息（包括之前对话、别的项目里看到的名字、学校、作品）。只用本次采访的回答和主人主动交给你的文件。
- 主人给你的是一个文件夹时，只读他指明的那几个文件。
- “模型暂定”的推测**不写进 brief**，也不出现在页面上。
- `brief.md` 与 `~/.nowme/profile.md` 只在本地，**永不上传**；不要把它们复制进 `page/`。
- 凭证和 token 不写进任何文件；不 `cat`、不打印、不复制 `~/.nowme/credentials`。

## 禁止事项

- 编造经历、数字、引语、奖项、链接；把推测写成事实；用示例页里虚构人物的内容。
- 在页面里联网（`fetch`、XHR、WebSocket、外部图片 / 字体 / 脚本）、存储（cookie、localStorage）、跳转顶层页面、`window.open`、表单、iframe、`eval`。可运行的 demo 只能放截图、动图或 `data-nm-href` 外链。
- 在页面里写联系方式、token、凭证。
- 声称做到了没实现的效果（例如“已经接入你的 GitHub 实时数据”“已经发布到墙上”）。
- 绕过 `nowme` 直接调用平台 API，或尝试发布、撤下、改公开范围。

## 失败处理

| 情况 | 怎么做 |
| --- | --- |
| `nowme` 不可用 | 按 0.1 换方式 B；仍不行就把错误原文告诉主人，请他确认仓库路径或安装 |
| `fetch` 返回 `TOKEN_INVALID` / `CREDENTIAL_INVALID` | 请主人在应用内重新生成指令，用新 token 重新 `nowme fetch <wallId> <token> --server <url>` |
| `validate` 有错误 | 按 `hint` 修复；同一个错误改了两次还在，把报告原文和你的理解告诉主人 |
| `preview` 报渲染错误 | 脚本异常 / CSP 违规通常是用了禁用 API 或外部资源，按报告位置修；浏览器启动失败时把错误原文告诉主人（也可以加 `--remote` 改由服务端渲染） |
| `preview` 报“预览次数已达上限” | 服务端预览每页每天有上限；直接 `nowme submit`，请主人在工作区里预览 |
| `submit` 被拒（422）或限流（429） | 422 按报告修复后重交；429 说明今天提交次数用完，明天再交 |
| 主人的请求做不到 | 说明原因，给 1–2 个可行的替代方案（见 `reference/editing.md`） |

任何失败都**不要删除** `brief.md`、`page/` 或主人给的素材；已提交的版本和已发布的页面不会因为本地失败受影响。
