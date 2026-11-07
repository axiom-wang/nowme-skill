# 设计思路：hackathon-terminal（终端控制台）

> 虚构人物：林舟，大三计算机，全栈偏后端，在黑客松找一位会做视觉和交互的设计。页面里的人物、学校、项目、数字都是虚构的。

## 想给人的感觉
“这个人是真的会写代码的，而且很清楚自己缺什么。”像打开一个正在运行的终端：冷静、利落、有一点程序员的自嘲。

## 核心视觉
首屏的 **“Open #1 help wanted” 招募卡**：用 GitHub issue 的样子写“找 1 位会做视觉和交互的设计”，荧光绿描边发光，是整页最亮的东西。黑客松的封面（顶部 375×500）必须一眼看出**角色**和**想找的人**——名字下面就是角色标签，再往下就是这张卡。

## 区块安排（data-nm-id）
| 区块 | 内容 | 为什么 |
| --- | --- | --- |
| `hero` | `whoami` 命令行 + 名字 + 角色标签 + 打字机轮播三句事实 + 招募卡 + 技能标签 | 封面就把“我是谁、我要谁”说完 |
| `teammate` | `git diff` 写分工：+ 我负责什么 / + 你负责什么；引用主人原话 | 让设计同学知道来了做什么，不是来打杂 |
| `projects` | 手机框里的小程序界面示意 + 解决什么 / 我负责 / 现在 + 一个真实数字 + `git log` 时间线 + 外链 | 一个具体项目胜过十个技能标签 |
| `skills` | `skills.json`，“不擅长：配色和排版” | 坦白短板，正好回扣“为什么找你” |
| `idea` | 群通知 → 待办的动画示意 + 主人原话 | 黑客松要说清这次想做什么 |
| `ping` | 引导去点平台的“想聊这个” | 页面里不放联系方式 |

## 用到的 Kit
`nm-theme-terminal`、`nm-terminal`（diff 与 json 两个窗口）、`nm-phone`、`nm-chip`、`nm-button nm-arrow` + `data-nm-href`、`data-nm-typewriter`、`data-nm-count`、`canvas.nm-particles[data-nm-particles=links]`（首屏星座连线背景）、`nm-reveal`、`nm-grain`、`nm-pulse`、`nm-scroll-hint`。字体：JetBrains Mono（西文、数字）+ Noto Sans SC 400/700（中文），通过 `--nm-font-body: "JetBrains Mono", var(--nm-font-noto-sans-sc)` 混排。

## 动效（medium）
粒子连线、打字机、数字滚动、群消息→待办的循环动画都由 Kit 或 CSS 实现，减少动效时全部变成静态画面（打字机显示第一句、数字显示最终值）。

## 借鉴时注意
- 借的是**结构**：命令行做区块标题、issue 卡做招募、diff 写分工。内容必须换成主人自己的。
- 数字（“300 多位同学用过”）只能写主人亲口给的；没有就不放数字块。
- 项目截图要用主人给的图片（放 `assets/`），这里的手机界面是 SVG 画的示意，并在页脚注明了“截图是示意”。
- 可运行的 demo 不能嵌进页面（不能联网），只能截图 + `data-nm-href` 外链。
