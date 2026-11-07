# NOWMe Kit v1 速查

Kit 由平台托管在 `/_nowme/kit/v1/`：`nowme.css`（变量、主题、排版、组件、动效）与 `nowme.js`（`window.nowme`）。用 Kit 拼出来的页面不会难看；想超越 Kit 也完全可以，只要遵守页面契约。源码：仓库 `packages/kit/static/`。

## 字体清单

| id | family | 中文 | 字重 | 气质 |
| --- | --- | --- | --- | --- |
| `lxgw-wenkai` | LXGW WenKai（霞鹜文楷） | 是 | 300 / 400 / 500 | 温和的楷体、手写感，适合手账、文艺、长段落 |
| `smiley-sans` | Smiley Sans（得意黑） | 是 | 400 | 倾斜的窄黑体，活泼、有冲劲，适合标题、游戏感 |
| `noto-sans-sc` | Noto Sans SC（思源黑体） | 是 | 400 / 500 / 700 / 900 | 中性、清晰；900 做海报大字 |
| `noto-serif-sc` | Noto Serif SC（思源宋体） | 是 | 400 / 700 / 900 | 文气、复古；900 做杂志标题 |
| `inter` | Inter | 否 | 400 / 500 / 700 / 900 | 干净的西文无衬线，数字好看 |
| `space-grotesk` | Space Grotesk | 否 | 300 / 400 / 500 / 700 | 有个性的几何西文 |
| `jetbrains-mono` | JetBrains Mono | 否 | 400 / 500 / 700 / 800 | 等宽，终端、代码、HUD |
| `instrument-serif` | Instrument Serif | 否 | 400 | 优雅的西文衬线展示字 |

每款都有变量 `--nm-font-<id>`（family 名 + 系统回退）。中文字体按字重合计最多 2 款。`--nm-font-sans / serif / mono` 只是系统字体栈。

**角色变量**：Kit 组件只用 `--nm-font-body`（正文）、`--nm-font-display`（标题）、`--nm-font-code`（代码 / 等宽）、`--nm-font-hand`（手写感），默认都是系统字体。页面在 `:root` 里改：

```css
:root {
  --nm-font-body: var(--nm-font-lxgw-wenkai);
  --nm-font-display: var(--nm-font-smiley-sans);
  --nm-display-weight: 400;          /* 得意黑只有 400 */
  --nm-font-code: "JetBrains Mono", var(--nm-font-noto-sans-sc);
}
```

## 主题

把类名放在 `<html>`、`<body>` 或任意区块上（区块可以有自己的主题）。也可以在 `:root` 里覆盖单个颜色变量。

| 类 | 感觉 |
| --- | --- |
| `nm-theme-paper` | 暖白纸、墨色字、朱红点缀（默认） |
| `nm-theme-ink` | 墨夜：近黑底、暖白字、橙色 |
| `nm-theme-terminal` | 终端：黑底、荧光绿、琥珀 |
| `nm-theme-candy` | 糖果：浅粉、亮粉、天蓝、柠檬黄 |
| `nm-theme-forest` | 森林：浅灰绿、墨绿、砖红 |
| `nm-theme-sunset` | 七十年代：奶油、橘红、芥末黄、墨绿 |
| `nm-theme-mono` | 黑白：纯白、纯黑线条、信号橙 |
| `nm-theme-cobalt` | 克莱因蓝：钴蓝底、白字、明黄 |
| `nm-theme-night` | 星夜：深海军蓝、星光黄、紫 |

## 设计变量

- 颜色：`--nm-bg`、`--nm-surface`、`--nm-surface-2`、`--nm-fg`、`--nm-muted`、`--nm-line`、`--nm-accent`、`--nm-accent-2`、`--nm-accent-3`、`--nm-on-accent`、`--nm-focus`
- 间距：`--nm-space-1`…`--nm-space-9`（4、8、12、16、24、32、48、64、96px）、`--nm-gutter`（页面左右留白 20px）、`--nm-page-max`（560px）
- 圆角：`--nm-radius-xs / s / m / l / xl / pill`
- 字号：`--nm-text-xs / s / m / l / xl / 2xl / 3xl / 4xl / 5xl / mega`（大字号随视口放大）；`--nm-leading-tight / snug / leading`、`--nm-display-weight`
- 阴影：`--nm-shadow-s / m / l`、`--nm-shadow-hard`（硬投影）
- 动效：`--nm-ease-out / in-out / spring`、`--nm-dur-1…4`

## 组件与片段

版式：`nm-page`（居中容器）、`nm-hero`（首屏，顶部对齐）、`nm-section`、`nm-stack`（`--nm-gap`）、`nm-cluster`、`nm-grid`（`--nm-min`）、`nm-split`、`nm-bleed`（撑满到屏幕边）、`nm-scroll-x`（横滑画廊，`--nm-item-width`）。

文字：`nm-display`、`nm-mega`、`nm-h1/h2/h3`、`nm-lead`、`nm-kicker`（眉题）、`nm-quote` + `nm-quote__by`（原话）、`nm-highlight`（荧光笔）、`nm-underline`（波浪线）、`nm-outline-text`（空心字）、`nm-vertical`（竖排）、`nm-gradient-text`（`--animated`）、`nm-muted`、`nm-mono`、`nm-hand`、`nm-num`。

```html
<!-- 标签 / 可切换选项 -->
<ul class="nm-chips"><li class="nm-chip">Go</li><li class="nm-chip nm-chip--accent">小程序</li></ul>
<button class="nm-chip" aria-pressed="false">能写后端</button>

<!-- 外链按钮（平台会先让访客确认域名） -->
<button class="nm-button nm-button--accent nm-arrow" data-nm-href="https://example.com/">看作品</button>

<!-- 贴纸（白色模切边）、圆贴纸、爆炸贴纸、印章 -->
<span class="nm-sticker" style="--nm-tilt:-6deg">新人报到</span>
<span class="nm-sticker nm-sticker--round" style="--nm-sticker-bg:#3d7bff;--nm-sticker-fg:#fff">会五种方言</span>
<span class="nm-sticker nm-sticker--burst">NEW!</span>
<span class="nm-stamp nm-stamp--worn">04:00 AM</span>

<!-- 拍立得 + 纸胶带（图片用主人给的 assets/） -->
<figure class="nm-polaroid nm-tape" style="--nm-tilt:3deg">
  <div class="nm-polaroid__photo"><img src="assets/market.jpg" alt="清晨菜市场"></div>
  <figcaption class="nm-polaroid__caption">卖豆腐的阿姨</figcaption>
</figure>

<!-- 便利贴、对话气泡 -->
<div class="nm-note" style="--nm-note-color:#f6d860">想找人一起早起拍照</div>
<p class="nm-speech">新人一枚，请多指教！</p>

<!-- 终端窗口 -->
<div class="nm-terminal">
  <div class="nm-terminal__bar"><span class="nm-terminal__title">~/team</span></div>
  <div class="nm-terminal__body"><span class="nm-prompt">whoami</span>
林舟 · 全栈<span class="nm-cursor"></span></div>
</div>

<!-- demo 截图：浏览器框 / 手机框 -->
<div class="nm-browser"><div class="nm-browser__bar"><span class="nm-browser__url">bookswap.example</span></div>
  <div class="nm-browser__body"><img src="assets/demo.png" alt="首页截图"></div></div>
<div class="nm-phone"><div class="nm-phone__screen"><img src="assets/app.png" alt="小程序截图"></div></div>

<!-- 票根、时间线、大数字（只放主人给的真实数字）、进度条 -->
<div class="nm-ticket"><div class="nm-ticket__main">周末一起逛唱片店</div><div class="nm-ticket__stub">ADMIT ONE</div></div>
<ol class="nm-timeline"><li class="nm-timeline__item"><span class="nm-timeline__time">3 月</span>第一版上线</li></ol>
<div class="nm-stat"><span class="nm-stat__value" data-nm-count="300">300</span><span class="nm-stat__label">位同学用过</span></div>
<div class="nm-meter nm-meter--blocks" style="--nm-value:.75"></div>

<!-- 折叠问答、头像、分隔线 -->
<details class="nm-details"><summary>第一款桌游怎么选？</summary><div class="nm-details__body">先从半小时能打完的开始。</div></details>
<span class="nm-avatar">舟</span>
<hr class="nm-divider nm-divider--wave">
```

卡片：`nm-card`（`--hard` 硬投影、`--glass` 毛玻璃、`--outline`）。质感：`nm-grain`（加在 `<body>` 上铺满整页颗粒）、`nm-bg-dots / grid / lines / stripes`（`--nm-pattern-color`、`--nm-pattern-size`）、`nm-glow`（光晕）、`nm-blob`（变形色块）。

## 动效与交互

| 用法 | 效果 |
| --- | --- |
| `class="nm-reveal"`，可加 `data-nm-reveal="up|down|left|right|scale|fade|blur"`；父元素 `data-nm-stagger="100"` | 滚动到视口时渐显，子元素依次出现 |
| `<div class="nm-marquee"><div class="nm-marquee__track"><span>…</span></div></div>`（`--reverse`、`--fade`、`--nm-marquee-duration`） | 跑马灯，自动铺满并无缝循环 |
| `nm-float`、`nm-wiggle`、`nm-spin`（`--nm-spin-duration`）、`nm-pulse`、`nm-blink`、`nm-pop-in`、`nm-shimmer`、`nm-scroll-hint` | 循环小动效；`data-nm-motion="calm"` 时自动停掉装饰性的 |
| `<span data-nm-typewriter><span>第一句</span><span>第二句</span></span>`（值写 `once` 只打一遍；`data-nm-speed`） | 打字机；减少动效时显示第一句 |
| `<span data-nm-count="42">42</span>` | 进入视口时数字从 0 滚到 42 |
| `<canvas class="nm-particles" data-nm-particles="stars|snow|bubbles|dust|links" data-nm-color="#fff">`（父元素需定位） | 粒子背景；不可见时暂停，减少动效时静态 |
| `data-nm-confetti`（值可写数量） | 点击时在点击处撒彩纸 |
| `.nm-flip[data-nm-flip] > .nm-flip__inner > .nm-flip__front + .nm-flip__back` | 点击翻面 |
| `<div data-nm-cycle><p>一</p><p>二</p></div>`（值 `random` 随机） | 点一下换一个 |
| `data-nm-tilt="10"` | 鼠标悬停 3D 倾斜（触屏不生效） |
| `data-nm-drag` | 可拖动（贴纸拼贴；元素别太大，否则手机上不好滚动） |

所有动效都尊重 `prefers-reduced-motion`：CSS 动画和过渡被全局关闭，脚本组件给出静态画面。

## window.nowme

| 成员 | 说明 |
| --- | --- |
| `nowme.openLink(url)` | 请平台打开 https 外链（平台会先让访客确认），返回是否已发出；非 https 返回 false |
| `nowme.reducedMotion` | 是否“减少动态效果”，实时更新 |
| `nowme.onReducedMotionChange(cb)` | 订阅变化，返回取消订阅函数 |
| `nowme.motionLevel` | `<html data-nm-motion>` 的值：calm / medium / lively |
| `nowme.ready(cb)` | Kit 初始化完成后执行（页面脚本建议包在里面） |
| `nowme.refresh(el?)` | 重新初始化某个范围内的 Kit 组件（动态插入的节点会自动处理） |
| `nowme.revealAll()` | 立刻显示所有 `.nm-reveal` |
| `nowme.typewriter(el, phrases?, options?)` | 打字机，返回 `{ stop() }` |
| `nowme.particles(canvas, options?)` | 粒子背景，返回 `{ start(), stop(), redraw() }` |
| `nowme.confetti(options?)` | 撒一次彩纸（`x`、`y`、`count`、`spread`、`colors`） |
| `nowme.countUp(el, options?)` | 数字滚动 |
| `nowme.rng(seed)` | 可复现的伪随机数（生成式图形用它，封面截图才稳定） |

自己写脚本动画时：

```js
nowme.ready(function () {
  var canvas = document.getElementById('art');
  var ctx = canvas.getContext('2d');
  var rand = nowme.rng('xiaoman');
  function draw(t) { /* 用 rand() 和 t 画一帧 */ }
  if (nowme.reducedMotion) { draw(0); return; }      // 减少动效：静态画面
  function loop(t) { draw(t); if (!nowme.reducedMotion) requestAnimationFrame(loop); else draw(0); }
  requestAnimationFrame(loop);
});
```
