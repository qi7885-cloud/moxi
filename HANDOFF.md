# 墨息 · BREATH OF INK — 交接文档

> 最后更新：2026-09-15 · 维护人：＿＿＿
> 一句话介绍：一座会呼吸的数字水墨园林。作品型单页网站，不是内容站——页面本身即交互装置。

---

## 1. 线上资产

| 资产 | 地址 / ID | 备注 |
|------|-----------|------|
| GitHub 仓库 | https://github.com/qi7885-cloud/moxi | 分支 `main`，交接时最新提交 `fa9ee67` |
| Netlify 站点 | https://moxi-ink.netlify.app | 已上线（HTTP 200） |
| Netlify Site ID | `1513a9c3-6162-446e-91b0-267d098ef1f6` | API 部署时使用 |
| 本地路径 | `~/Desktop/创意/moxi` | git 仓库根目录即网站根目录 |
| 本地预览 | `python3 -m http.server 8866` | 可选；直接双击 `index.html` 也能完整运行（零外部请求） |

## 2. 文件清单

```
moxi/
├── index.html   # 全部实现：HTML + CSS + JS 单文件（约 46KB），零依赖零外部资源
├── README.md    # 项目门面说明（玩法/技法）
├── HANDOFF.md   # 本文档
├── .gitignore   # .DS_Store / deploy.zip（部署产物，勿提交）
└── deploy.zip   # 部署用临时产物（已忽略，可随时删）
```

## 3. 技术架构（index.html 内部导览）

无构建、无框架、无字体/图片/JS 外链。所有系统都在 `<script>` 一个作用域里，按序排列：

| 系统 | 关键代码 | 说明 |
|------|----------|------|
| 四时色稿 | `PHASES` / `applyPhase()` / `stepPal()` | 晨(5-8点)/昼(8-17)/暮(17-20)/夜(20-5) 四套配色；`overrideKey` 由色签与 `I` 键控制；CSS 变量换肤，canvas 颜色做逐帧 RGB 插值（`PAL`），换装时画布与 DOM 同步渐变 |
| 生成远山 | `garden` | `makeNoise()`值噪声 + `fbm()` + 折脊项；6 层由远及近；鼠标 X 速度注入 `wind` 场驱动云烟；点击任意空白处生成墨晕 `drops`；滚动时 `sink`（山体下沉）+ `fade`（变淡）保证章节文字可读 |
| 静水池 | `pond` | 每帧 `clearRect` 后叠半透明水色；涟漪为噪声扰动椭圆环；墨鲤有三种行为：噪声巡游 / 循声靠近新涟漪 / 距离过近受惊窜逃；闲时自动泛涟 |
| 手卷 | `brush` | pointer capture；速度映射笔径（疾枯缓润）；`multiply` 混合 + 卫星溅墨模拟洇墨；`seal()` 钤朱印、`save()` 导出 PNG、resize 时旧画回贴保留 |
| 声音 | `sound` / `pluck()` | 双振荡器（tri + 倍频 sine）→ 低通扫频 → 反馈延迟回声；`SCALE` 为 A 起五声音阶（宫商角徵羽）；**默认静音**，点「抚水有声」时才创建 AudioContext（浏览器手势策略） |
| 惯性卷动 | `applyScroll()` | 不劫持滚轮：内容容器 `#smooth` 为 fixed，rAF 里对原生 `scrollY` 做 lerp 追随，`#spacer` 撑起真实页高。滚动条/键盘/锚点天然可用，无障碍友好 |
| 导航与光标 | `rail` / `[data-magnet]` / `#cursor` | 右侧卷轴进度线（墨填充 + 四章锚点）；磁性按钮；自定义光标三态：`link`（悬停可点）/ `ink`（落墨）/ `touch`（抚水，带文字标签）；`pointer: coarse` 自动隐藏 |
| 载入仪式 | `loaderStep()` | 墨金印章 + 衬线计数 + 进度线，完成时 `clip-path` 上揭；`prefers-reduced-motion` 时直接跳过 |
| 时辰钟 | `tickClock()` | `shichen[]` 十二时辰映射；顶栏 + 第肆章大时钟，每秒刷新、每分钟复查相位是否换装 |
| 出场动画 | `.split` / `.rv` | 逐字（blur+位移+stagger）与 IntersectionObserver 渐显 |

## 4. 设计规范

**四时配色**（改配色只动 `PHASES` 表 + 对应 CSS 默认值）：

| 相位 | 触发时段 | bg 底 | ink 墨 | panel 面板 | accent 朱 |
|------|----------|-------|--------|-----------|-----------|
| dawn 晨墨 | 5:00–7:59 | `#f1e8dd` | `#2b241e` | `#e8ddca` | `#a8502f` |
| day 昼墨 | 8:00–16:59 | `#f3eee2` | `#1c1914` | `#ece5d3` | `#9d3a28` |
| dusk 暮墨 | 17:00–19:59 | `#ebe0cf` | `#231a12` | `#e3d5bd` | `#8f3d22` |
| night 夜墨 | 20:00–4:59 | `#161310` | `#e9e2d0` | `#1e1a15` | `#c08b52` |

- 字体栈：中文 `"Songti SC" → "Noto Serif SC" → 宋体系 fallback`；西文 `Cormorant Garamond → Georgia`。**为保持零依赖未嵌 webfont**，Windows/Linux 上中文字形会有差异，属已知取舍
- 缓动统一用 `--ease: cubic-bezier(.16,1,.3,1)`；hairline 一律 `color-mix(ink 15%, transparent)`
- 章节节奏：`padding: 20vh 7vw`；诗词竖排 `writing-mode: vertical-rl + letter-spacing .42em`
- 描边数字（壹貳叁肆）用 `-webkit-text-stroke` + 透明填充

## 5. 运维手册

**日常更新**（改完 `index.html`）：

```bash
cd ~/Desktop/创意/moxi
git commit -am "更新说明" && git push        # 更新 GitHub
zip -q deploy.zip index.html README.md      # 重新部署 Netlify
curl -X POST -H "Authorization: Bearer <NETLIFY_TOKEN>" \
  -H "Content-Type: application/zip" \
  --data-binary @deploy.zip \
  "https://api.netlify.com/api/v1/sites/1513a9c3-6162-446e-91b0-267d098ef1f6/deploys"
```

**一劳永逸**：Netlify 后台 → Site configuration → Build & deploy → Link repository 绑定本仓库后，`git push` 即自动部署（也可配 GitHub Actions，用 `nwtgck/actions-netlify`，secret 存 token）。

**回滚**：Netlify 后台 Deploys 页可一键回滚任意历史部署；代码回滚则 `git revert` 后重新部署。

**凭据说明**：
- 需要两个令牌：GitHub fine-grained PAT（Contents: Read and write；要自动建仓还需 Administration）、Netlify PAT（`nfp_` 开头）
- 本机与仓库**均未存储任何令牌**（origin 为匿名 URL，History 无敏感信息）；令牌由维护者自持
- 交接时用户曾在对话中提供过令牌，如担心可自行吊销重发，吊销不影响已部署站点

## 6. 已知边界与坑（重要）

1. **`pond.step()` 每帧开头必须 `clearRect`** —— 水色是半透明叠加，漏清屏会逐帧累积把整池"灌满墨"（实际发生过的 bug）
2. 顶栏透明 + `#topfade` 顶部渐隐：章节内容上滚与顶栏相遇时靠雾化过渡，别删这个 div
3. WebAudio 默认静音是**特性**（浏览器手势策略 + 优雅），不是 bug
4. 依赖较新 CSS：`color-mix()`、`-webkit-text-stroke`；旧浏览器降级为无 hairline/实心数字，可接受
5. 墨晕用 `shadowBlur` 而非 `ctx.filter`（Safari 兼容）；DPR 钳制在 1.75 控制性能
6. `hero`/章节文字压山处用了三层 `text-shadow: var(--bg)` 光晕保证可读，改山体透明度时留意
7. 山体随滚动 `sink+fade`（`garden.step` 开头两行）是"整页皆园林"与"章节可读"的折中方案，调 scroll 参数时两章联测
8. 触屏端 `pond`/`brush` 已设 `touch-action:none`，画水/画画不会带动页面滚动

## 7. Backlog（候选优化，按价值排序）

- [ ] Netlify Link repository 或 GitHub Actions，实现 push 自动部署
- [ ] 社交分享：补 `og:image`（可截夜墨首屏 1200×630）与 `og:title/description`
- [ ] 新章节候选：雪（粒子落雪 + 山体积雪）、雨（涟漪自动密集 + 雨声白噪）、印谱集（用户钤印的画廊）
- [ ] 环境音：夜墨模式加极低音量风声（噪声 + 低通 + LFO）
- [ ] iOS Safari 实机回归（惯性卷动手感、audio 解锁路径）
- [ ] SEO：补结构化数据、`sitemap.txt`；跑一次 Lighthouse 存基线
- [ ] 英文版文案（`<html lang>` 切换或双语并列）

## 8. 验收基准（改动后过一遍）

- [ ] 四个色签各点一遍 + 按 `I` 入夜/回昼，配色过渡平滑（canvas 与 DOM 同步）
- [ ] 首屏：移动鼠标见风、点空白见墨晕；时钟冒号闪烁、时辰正确
- [ ] 声章：抚水出涟漪与音符（开声音后）、墨鲤循声而来、受惊而散
- [ ] 墨章：快笔细、慢笔润、钤印出现在右下、收卷能下载 PNG
- [ ] 时章：大时钟走字、色签高亮态、「回到此刻」复位
- [ ] 移动端 390px：首屏排版、水池/手卷可画、按钮不溢出
- [ ] 系统开启"减弱动态效果"后：无惯性卷动、无载入动画、内容直出

---

*由留白、算法与你的到访共同写就。接手愉快。*
