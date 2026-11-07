# 页面输出契约（第 4–5 步细则）

依据：`docs/contracts.md` 第 1 节与 2.2 节。平台会在提交时重新校验全部规则，不信任本地结果；页面最终在沙箱 iframe（`sandbox="allow-scripts"`）和严格 CSP（`connect-src 'none'`）里运行。

## 目录

```
page/
├── index.html      # 必需。自包含单文件，HTML / CSS / JS 都内联
├── card.json       # 必需。缩略卡
├── manifest.json   # 必需。版本与声明
├── cover.png       # 不要自己放：由 nowme preview 生成（750×1000）
└── assets/         # 可选。主人给的图片：png / jpg / jpeg / webp / gif
```

`page/` 里的其他文件（`brief.md`、草稿、`DESIGN.md`）会得到 `IGNORED_FILE` 警告，提交时不上传——但 brief 本来就不该放进来。

## index.html 骨架

```html
<!doctype html>
<html lang="zh-CN" class="nm-theme-paper" data-nm-motion="medium">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>名字 · 一句话</title>
<link rel="stylesheet" href="/_nowme/kit/v1/nowme.css">
<script src="/_nowme/kit/v1/nowme.js"></script>
<style>
  :root {
    --nm-font-body: var(--nm-font-lxgw-wenkai);      /* 用了哪款 Kit 字体，就在 manifest.fonts 里声明 */
    --nm-font-display: var(--nm-font-noto-serif-sc);
    --nm-display-weight: 900;                        /* 标题字重必须是声明过的字重 */
  }
  /* 页面自己的样式 */
</style>
</head>
<body>
<main class="nm-page">
  <header class="nm-hero" data-nm-id="hero">…</header>
  <section class="nm-section" data-nm-id="works">…</section>
  <section class="nm-section" data-nm-id="together">…</section>
</main>
<script>
  nowme.ready(function () { /* 页面自己的交互 */ });
</script>
</body>
</html>
```

- 必须有 `<!doctype html>`、`<meta charset="utf-8">`、`<meta name="viewport" content="width=device-width, ...">`；建议 `<html lang="zh-CN">`。
- `nowme.js` 放在 `<head>` 里、不加 `defer`，这样首屏不会闪；页面脚本放在 `</body>` 前，或包在 `nowme.ready(...)` 里。
- 体积 ≤ 300 KB（含内联的 base64 图片，不含 `assets/` 与字体）。大图放 `assets/`，不要内联。
- 移动端优先，以 375px 宽设计；封面是页面顶部 375×500 的截图，最重要的信息放在这里。

## 只能引用这些

| 可以 | 写法 |
| --- | --- |
| Kit | `/_nowme/kit/v1/nowme.css`、`/_nowme/kit/v1/nowme.js`（以根路径引用） |
| 图片 | `assets/<文件名>`（相对路径）、`data:image/...`、脚本运行时生成的 `blob:` |
| 页内锚点 | `<a href="#works">` |
| 外链 | `data-nm-href="https://..."`（Kit 自动绑定点击 / 回车 / 空格），或脚本里 `nowme.openLink("https://...")`；只能 `https:` |

不可以：任何 `http(s)://` 资源（图片、字体、CSS、脚本、iframe）、`<a href="https://...">`、`<form>`、`<iframe>`、`<object>`、`<embed>`、`<base>`、`<meta http-equiv="refresh">`、除 Kit 以外的 `<link>` 与 `<script src>`。内联 SVG 直接写 `<svg>`，不需要 `xmlns`；不要用 `url("data:image/svg+xml,...")` 写 SVG 背景（里面的 `xmlns` 地址可能被当成外部引用），改用内联 `<svg>` 或 Kit 的纹理类。

## 脚本里不能出现

`fetch(`、`XMLHttpRequest`、`WebSocket`、`EventSource`、`sendBeacon`、`RTCPeerConnection`、`WebTransport`、`localStorage`、`sessionStorage`、`indexedDB`、`caches.`、`document.cookie`、`cookieStore`、`window.open`、`window.top`、`top.location`、`parent.location`、`location.href =`、`location.assign`、`location.replace`、`serviceWorker`、`importScripts`、`new Worker`、`new SharedWorker`、动态 `import(`、`eval(`、`new Function`、`setTimeout("..."`、`getUserMedia`、`geolocation`。

这条检查也扫注释和 `on*` 属性，所以注释里也别写这些词；不要写 `onclick="..."`，用 `addEventListener`。变量名注意别撞上（例如 `prefetch(`、`retrieval(` 里含有 `fetch(`、`eval(`）。

## data-nm-id

- 至少 3 个主要区块带 `data-nm-id`，值匹配 `^[a-z][a-z0-9-]{0,39}$`，页面内唯一。
- 用有含义的英文短词：`hero`、`projects`、`teammate`、`collection`、`ask-me`……修改时按它定位（第 6 步）。
- `manifest.blocks` 写同样的列表、同样的顺序。

## 动效与减少动效

- 引入了 `nowme.css` 的页面，CSS 动画和过渡由 Kit 的全局规则处理（减少动效时全部关闭）。所以动画的**终点必须是元素的正常状态**：关闭动画后内容要完整显示（例如用 `animation: x 1s both` 从透明“出现”，而不是靠动画把元素移进视野）。
- 脚本动效（`requestAnimationFrame`、`.animate(`、Canvas 循环）必须检查 `nowme.reducedMotion`，为真时给一张静态画面；最好用 Kit 现成的 `nowme.particles` / `nowme.confetti` / `data-nm-typewriter`，它们已经处理好了。
- `<html data-nm-motion="calm|medium|lively">` 与 `manifest.motionLevel` 一致。

## 字体

- 只用 Kit 字体清单（`context.json` 的 `kit.fonts`，或 `reference/kit.md`）。用 `var(--nm-font-<id>)` 或直接写 family 名；不写 `@font-face`、不内联字体、不把字体放进 `assets/`。
- 每用一款，就在 `manifest.fonts` 里声明 `{ "id": "...", "weights": [...] }`，字重只能是清单里有的。
- **中文字体每个字重算 1 款，合计 ≤ 2**。例如 `noto-sans-sc` 400 + 900，或 `smiley-sans` 400 + `lxgw-wenkai` 400。西文字体不计入。
- 页面里实际用到的字重必须声明过：`font-weight: 700` 而只声明了 400，浏览器会伪粗体，很难看。Kit 的标题字重由 `--nm-display-weight` 控制（默认 700），换成只有 400 的字体（得意黑）时设为 400。
- 中西混排：`font-family: "Inter", var(--nm-font-noto-sans-sc);`（西文用 Inter，中文落到思源黑体）。
- 平台按页面实际用到的字生成字体子集，取字范围是：`index.html` 源码里出现的**全部**字符（含属性、脚本、注释）+ 首屏渲染后的文字 + `manifest.extraChars`。所以写在源码任何地方的字（包括脚本字符串、`data-*` 属性）都会进子集；`extraChars` 只需要补**源码里完全没有出现**、又要在页面上显示的字（例如脚本按码点算出来的字）。`nowme validate` 会自动把脚本字符串里的字补进 `extraChars`（不影响子集，只是消除警告）。把交互后才显示的文字写成隐藏的 HTML 元素、用脚本切换 `hidden` 仍然是最稳妥的做法（示例页都这么做）。`nowme preview` 用同样的规则在本地生成子集，预览看到的就是提交后的效果。

## card.json

```json
{
  "schemaVersion": 1,
  "displayName": "阿柚",
  "tagline": "做过 3 个校园小程序的前端，想找能一起熬夜的设计",
  "hooks": [
    { "id": "help-frontend", "kind": "can-help", "text": "我能帮你把原型变成能点的页面", "opener": "你好！我在墙上看到你说能把原型变成能点的页面，我们队正好缺前端，想聊聊吗？" },
    { "id": "find-designer", "kind": "looking-for", "text": "找会做视觉的队友", "opener": "嗨，我是做设计的，看到你在找会做视觉的队友～" }
  ]
}
```

| 字段 | 规则 |
| --- | --- |
| `displayName` | 1–24 字；按边界用真名或昵称 |
| `tagline` | 1–40 字，一行介绍，不换行 |
| `hooks` | 2–3 个；`id` 小写短词且唯一；`text` 2–32 字；`opener` 1–80 字，**访客**发给主人的第一句话；`kind` 取场景指南 `hookKinds` |

文本长度按字（码点）算；不能只有空白，不能有换行和制表符；不能有多余字段。

## manifest.json

```json
{
  "schemaVersion": 1,
  "sceneType": "hackathon",
  "kitVersion": "1",
  "motionLevel": "medium",
  "fonts": [{ "id": "noto-sans-sc", "weights": [400, 700] }, { "id": "jetbrains-mono", "weights": [400] }],
  "extraChars": "",
  "blocks": ["hero", "teammate", "projects"],
  "generator": { "agent": "claude-code", "skillVersion": "0.1.0" }
}
```

`sceneType` 取 `context.json` 的 `wall.sceneType`；`fonts` 与 `extraChars` 必填（可以为空）；除 `generator` 内部外不能有多余字段。

## assets/

只放主人给的图片或你用代码生成的位图：`png` / `jpg` / `jpeg` / `webp` / `gif`；文件名小写字母数字与 `._-`，最多一层子目录；单个 ≤ 2 MB，合计 ≤ 8 MB，最多 20 个。SVG、字体、HTML、脚本一律不能放。用 `<img src="assets/xxx.jpg" alt="...">` 引用；没被引用的文件会得到警告。

## 校验错误怎么修

| 错误码 | 常见原因 | 修法 |
| --- | --- | --- |
| `MISSING_FILE` | 少了 `index.html` / `card.json` / `manifest.json` | 补上 |
| `HTML_TOO_LARGE` | 内联了大图或大段 base64 | 图片放 `assets/`；删掉没用的 SVG 路径 |
| `HTML_MISSING_DOCTYPE` / `_CHARSET` / `_VIEWPORT` / `_LANG` | 骨架缺项 | 照上面的骨架补 |
| `EXTERNAL_RESOURCE` | 引用了网上的图片、字体、CSS | 换成 `assets/`、内联 SVG 或 Kit |
| `FORBIDDEN_LINK_HREF` | `<a href="https://...">` | 改成 `<a data-nm-href="https://...">` 或按钮 |
| `INSECURE_OPEN_LINK` | 外链不是 https | 换成 https 地址，或删掉 |
| `EXTERNAL_URL_IN_SCRIPT` | 脚本字符串里有网址 | 网址只放在 `nowme.openLink("...")` 参数或 `data-nm-href` 里 |
| `FORBIDDEN_ELEMENT` | 用了 form / iframe / object / embed / base / 外部 link 或 script | 删掉；表单改成普通按钮 + 脚本；嵌入内容改成截图 + 外链 |
| `FORBIDDEN_API` | 见上面的禁用列表 | 删掉；需要“记住状态”的交互只在本次浏览内有效 |
| `JS_SYNTAX_ERROR` | 内联脚本写错 | 按行号修 |
| `NM_ID_TOO_FEW` / `_INVALID` / `_DUPLICATE` | 区块标识不够、格式不对或重复 | 至少 3 个、小写短词、唯一 |
| `NM_BLOCKS_MISMATCH` | `manifest.blocks` 与页面不一致 | 改成页面里的 `data-nm-id` 列表 |
| `MOTION_WITHOUT_REDUCED` | 没引 `nowme.css` 却写了 CSS 动画；或脚本动画没检查减少动效 | 引入 `nowme.css`；脚本里 `if (nowme.reducedMotion) { 画静态帧; return; }` |
| `FONT_FACE_FORBIDDEN` | 写了 `@font-face` 或字体 data URI | 删掉，改用 Kit 字体 |
| `FONT_UNDECLARED` | 用了 Kit 字体但没在 `manifest.fonts` 声明 | 补声明 |
| `FONT_NOT_IN_KIT` | 写了清单外的字体名 | 换成 Kit 字体或通用族名（`sans-serif` 等） |
| `FONT_TOO_MANY_CJK` | 中文字体字重合计 > 2 | 减少字重或换成一款字体 |
| `FONT_UNKNOWN` | 字体 id 或字重不在清单里 | 照 `reference/kit.md` 的清单改 |
| `EXTRA_CHARS_MISSING` | 脚本里的中文不在静态文本和 `extraChars` 里 | `nowme validate` 默认自动补全；仍有警告时手动加 |
| `CARD_INVALID` / `MANIFEST_INVALID` | JSON 写错、字段超长或多了字段 | 按报告里的字段名修 |
| `COVER_INVALID` | 自己放了不合格的 `cover.png` | 删掉，让 `nowme preview` 生成 |
| `ASSET_*` | 图片格式、大小、数量、文件名不合规，或引用了不存在的图片 | 转成 jpg / webp、压缩、改名、删掉没用的 |
| `CREDENTIAL_LEAK` | 文件里出现了 `nmc_` / `nmt_` 开头的字符串或 `credentials` 文件 | 立刻删掉；凭证只由 `nowme` 管理 |
| `IGNORED_FILE` | `page/` 里有白名单外的文件 | 移出 `page/`（brief 放在工作目录根下） |

渲染检查（`nowme preview` 与平台提交时）：`RENDER_SCRIPT_ERROR`（脚本抛异常）、`RENDER_CSP_VIOLATION`（用了被禁止的资源或 API）、`RENDER_CONSOLE_ERROR`、`RENDER_SLOW`（375×667 下加载超过 3 秒：减少 Canvas 粒子数量、压缩图片）、`RENDER_BLANK`（渲染后没有文字：检查是不是内容全靠脚本生成且脚本出错）。
