# 个人导航页 · 制作手册

> 单文件、零依赖、仿 Google 首页的私人起始页。36 个站点 / 6 个分类，带真实 favicon、农历时钟、站点可达性圆点。
>
> 位置：`nav-page/index.html` · 配套：`nav-page/icon/`（35 个图标 + manifest）
> 版本：v11 · 最后更新 2026-10-27 · 全文约 4200 字，正常查阅 6 分钟

---

## 0. 速查卡

先看这张表，90% 的日常改动都在这几行里。

| 我想…… | 改哪里 | 说明 |
|---|---|---|
| 加 / 删 / 改站点 | `index.html` 的 `GROUPS` 数组 | 唯一数据源，改完刷新即可 |
| 让某张卡片占两列 | 该项加 `wide: true` | 3 列网格下占 2 列 |
| 指定某个自定义图标 | 该项加 `f: '完整地址'` | 优先级最高，跳过所有自动逻辑 |
| 换图标图片 | 直接替换 `icon/<host>.png` | 本地图标不进缓存，换完立刻生效 |
| 改分类名 / 关键词 | `GROUPS[i].name` / `.kw` | `kw` 是标题右侧那行淡灰小字 |
| 加第 7 个分类 | `GROUPS` 末尾追加对象 | 配色自动按 `h(序号 % 6)` 循环 |
| 换配色 | CSS 里 `.h0`–`.h5` 两组变量 | 见 §3.4 |
| 换搜索引擎 | `<form id="sf" action="...">` | 改 action + `input name` 即可 |
| 改格言 | `QUOTES` 数组 | 每次刷新随机取一条 |
| 改问候语时段 | `renderGreet()` 里的三元链 | 目前 5 档 |
| 调整可达性灵敏度 | `LIVE_TIMEOUT` / `LIVE_TTL` | 见 §6 参数表 |

---

## 1. 成品概览

### 1.1 文件清单

```
nav-page/
├── index.html          42 KB   全部 HTML + CSS + JS，双击即用
├── icon/              908 KB   35 个站点图标
│   ├── <host>.<ext>             文件名 = 站点域名，扩展名按真实文件头定
│   └── manifest.json            {host: 'icon/<host>.<ext>'} 扁平映射
└── README.md                   本手册
```

**关键约束：`index.html` 不引用任何外网资源。** 字体用系统字体栈，图标用本地 `icon/`，农历用浏览器内置 ICU。断网状态下除了「实时站点可达性」会全部转红，其余功能（含全部图标）完全正常。

`icon/` 是**可选增强**：单独把 `index.html` 拷到别处，图标会自动回退到在线源（favicon.im 等），不会空白，只是多了网络请求。

### 1.2 三条设计基线

做任何改动前，先确认没有破坏这三条——它们是这个页面「像 Google」的全部原因。

1. **极简到接近空**：白底、无边框面板、无装饰背景、无毛玻璃。视觉重量全部压在一个 wordmark、一个搜索框和一层薄薄的卡片上。
2. **每个像素都对齐 Google 的尺度**：584px 搜索框、46px 高、24px 圆角、8px 卡片圆角、14px 正文、`#202124` 文字色。这些数字不是审美，是 Google 的真实规格。
3. **不做本地筛选**：这是用户明确要求（两轮确认）。搜索框只负责「把词丢给必应」，页面没有任何搜索/过滤/切换逻辑。曾经做过的文本筛选、分类切换按钮、最近访问行都已彻底删除——**不要因为「看起来更强大」再加回来**。

### 1.3 打开方式

直接双击 `index.html`，或拖进浏览器。建议在浏览器里「固定标签页」，日常当新标签页用。

> 若想让它接管新标签页，需要装扩展（如 New Tab Redirect）指向本地文件路径。`file://` 协议下的 localStorage 按文件路径隔离，移动文件后主题/缓存会重置，属正常现象。

---

## 2. 数据结构：唯一需要懂的地方

整个页面只有一个数据源，位于 `<script>` 开头的 `GROUPS` 常量。

### 2.1 字段说明

| 字段 | 必填 | 含义 | 备注 |
|---|---|---|---|
| `n` | ✅ | 站点名称 | 卡片主标题，过长会省略号 |
| `d` | ✅ | 描述 | 卡片副标题，建议「域名 · 一句话」，≤14 字 |
| `u` | ✅ | 链接 | 完整 URL，含协议 |
| `i` | ⬜ | 字母占位 | 取不到 favicon 时显示，单字最佳；不填取 `n[0]` |
| `f` | ⬜ | 自定义图标地址 | 填了就走它，跳过本地清单与三级源 |
| `wide` | ⬜ | 占两列 | 3 列布局下 `grid-column: span 2` |

分组对象本身：

| 字段 | 含义 |
|---|---|
| `id` | 仅作标识，当前未参与逻辑 |
| `name` | 分类名，显示在胶囊里 |
| `kw` | 关键词，胶囊右侧的淡灰小字，≤640px 隐藏 |
| `items` | 站点数组 |

### 2.2 一条真实数据长什么样

```js
{
  id: 'ai', name: 'AI 与本地模型', kw: '推理 · 下载 · 路由',
  items: [
    { n:'LM Studio', d:'localhost:1234 · 本地推理', u:'http://localhost:1234', i:'L' },
    { n:'Hugging Face', d:'huggingface.co', u:'https://huggingface.co', i:'H' }
  ]
}
```

### 2.3 增删站点的完整流程

1. 在对应分组 `items` 里加一行，只填 `n` / `d` / `u` / `i`。
2. 刷新页面，看图标是否取到：
   - **取到了** → 想本地化的话继续第 3 步。
   - **显示字母** → 该站没有可用 favicon（如 localhost 服务），属正常，或手动补 `f`。
3. 想要离线可用，把图标存成 `icon/<host>.<ext>`，并在 `FAV_LOCAL` 里加一行：
   ```js
   'example.com': 'icon/example.com.png',
   ```
   同时建议更新 `icon/manifest.json`（页面不读它，纯粹给脚本/人看）。

> **注意**：`FAV_LOCAL` 的键必须是 `new URL(u).hostname` 的结果。带 `www.` 的链接键就要带 `www.`，写错不会报错，只是退回在线源。

### 2.4 为什么每组固定 6 个

3 列网格 + 每组 6 项 = 正好两排、右边缘齐平。改成 5 个或 7 个都会在第二排留下空洞。要加第 7 个，要么把整组扩到 9 个（三排），要么接受空洞。

---

## 3. 视觉系统

### 3.1 颜色变量（亮色 / 深色）

全部定义在 `:root` 与 `[data-theme="dark"]`，改主题只改这两处。

| 变量 | 亮色 | 深色 | 用途 |
|---|---|---|---|
| `--bg` | `#ffffff` | `#202124` | 页面底色 |
| `--txt` | `#202124` | `#e8eaed` | 主文字 |
| `--muted` | `#5f6368` | `#9aa0a6` | 次文字 |
| `--dim` | `#70757a` | `#9aa0a6` | 描述、页脚 |
| `--faint` | `#9aa0a6` | `#80868b` | 分隔点、提示 |
| `--border` | `#dfe1e5` | `#5f6368` | 搜索框描边 |
| `--border-2` | `#dadce0` | `#3c4043` | 卡片描边、页脚上边 |
| `--card` | `#ffffff` | `#202124` | 卡片底 |
| `--hover` | `#f8f9fa` | `#303134` | 按钮 hover |
| `--search-bg` | `#ffffff` | `#303134` | 搜索框底 |
| `--footer` | `#f2f2f2` | `#171717` | 页脚底 |
| `--ic-bg` | `#eef1f4` | `#dde3ea` | 图标托底 |
| `--ic-line` | `rgba(60,64,67,.09)` | `rgba(0,0,0,.05)` | 图标托底内描边 |
| `--link` | `#1a73e8` | `#8ab4f8` | 链接（当前未大量使用） |

> **深色下 `--ic-bg` 必须是浅色**（`#dde3ea`）。GitHub、少数派这类「深色描线」favicon 放在黑卡片上会直接糊掉，浅底是它们的唯一出路。这是本页色彩系统里最容易搞错的一处。

### 3.2 字体

```css
--sans: Arial,"Helvetica Neue","PingFang SC","Hiragino Sans GB","Microsoft YaHei",sans-serif;
--logo: "Product Sans","Google Sans",Arial,"PingFang SC",sans-serif;
```

`Product Sans` 是 Google 商标字体，本机通常没有，会优雅回落到 Arial。想要更接近原版可自行安装 Google Sans，但别把字体文件打进项目——会破坏「零外部资源」。

数字统一用 `font-variant-numeric: tabular-nums`，时钟跳动时不会左右抖。

### 3.3 尺度与阴影

| 项 | 值 |
|---|---|
| 搜索框 | 宽 `584px` / 高 `46px` / 圆角 `24px` / 字号 `16px` |
| 卡片 | 圆角 `8px`（`--radius`）/ 内距 `10px 12px` / 间距 `10px` |
| 图标块 | `42×42` / 圆角 `11px` / 内部图 `28×28` |
| 分类胶囊 | 高 `34px` / 圆角 `999px` |
| 页面上边距 | hero `38px` / main `40px` / 分组间 `36px` |
| 容器 | `max-width: 1120px` + 左右 `24px` |

两级阴影，全是 Google Material 的原值：

```css
--shadow-hover: 0 1px 6px rgba(32,33,36,.28);                          /* 搜索框浮起 */
--shadow-card: 0 1px 2px 0 rgba(60,64,67,.30), 0 1px 3px 1px rgba(60,64,67,.15);  /* 卡片 hover */
```

### 3.4 分组六色板

每个分组挂一套变量在 `.h0`–`.h5` 上，卡片、胶囊、圆点全部继承。

| 类 | 色名 | `--g-fg` 主色 | `--g-tint` 浅底 | `--g-tint-2` 深一档 | `--g-glow` 柔光 |
|---|---|---|---|---|---|
| `.h0` | 蓝 | `#1967d2` | `#e8f0fe` | `#d2e3fc` | `rgba(25,103,210,.34)` |
| `.h1` | 红 | `#c5221f` | `#fce8e6` | `#fad2cf` | `rgba(197,34,31,.30)` |
| `.h2` | 琥珀 | `#a8620a` | `#fef3d7` | `#fbe3a8` | `rgba(168,98,10,.30)` |
| `.h3` | 绿 | `#137333` | `#e6f4ea` | `#ceead6` | `rgba(19,115,51,.30)` |
| `.h4` | 紫 | `#8430ce` | `#f3e8fd` | `#e9d2fd` | `rgba(132,48,206,.30)` |
| `.h5` | 青 | `#00707a` | `#e2f6f7` | `#c6ecef` | `rgba(0,112,122,.30)` |

深色主题换成亮一档的同色（`#8ab4f8` / `#f28b82` / `#fdd663` / `#81c995` / `#c58af9` / `#78dce8`），`--g-tint` 与 `--g-tint-2` 改为同色不同透明度的 `rgba`，`--g-on` 统一为 `#12161c`（胶囊内文字反色）。

**`--g-deep` 只在深色主题里用**，出现在字母徽标回退场景：深色下图标托底是浅色，字母若还用亮色 `--g-fg` 就看不清，必须换成深色。

---

## 4. 六个模块的实现原理

### 4.1 主题：先于渲染的 3 行

深色用户打开页面时，如果等 JS 跑完才切主题，会先看到一帧白屏闪光。解法是把读 localStorage 的脚本放在 `<head>` 里、样式表之前：

```html
<script>try{var t=localStorage.getItem('nav-theme');if(t)document.documentElement.dataset.theme=t}catch(e){}</script>
```

主题通过 `<html data-theme="light|dark">` 一个属性驱动全部 CSS 变量，没有任何 class 切换逻辑。切换按钮只改这个属性 + 写 localStorage + 换图标（深色显示太阳、亮色显示月亮）。

### 4.2 英雄区：三段

**① wordmark「my567」**

五个字母各自包一层 `<span class="c1">`–`<span class="c4">`，用 Google 四色循环（m 蓝 / y 红 / 5 黄 / 6 绿 / 7 红）。字号 `clamp(46px, 8.4vw, 76px)`，字距 `-.045em`——负字距是关键，Google 的字标是收紧的。

彩蛋：hover 时五个字母依次弹起，靠 `nth-child` 递增 45ms 延迟：

```css
.logo:hover span{transform:translateY(-5px)}
.logo:hover span:nth-child(2){transition-delay:.045s}
/* …3/4/5 分别 .09 / .135 / .18s */
```

**② 搜索框**

原生表单提交，没有任何 JS 参与搜索逻辑：

```html
<form class="search" id="sf" action="https://www.bing.com/search" method="get"
      target="_blank" rel="noopener noreferrer" role="search">
  <span class="bing-mark" aria-hidden="true">b</span>
  <input id="q" name="q" type="search" placeholder="在 Bing 中搜索" autocomplete="off">
  <span class="hint">/</span>
</form>
```

这行 `<form>` 解决了全部问题：URL 编码由浏览器做、新标签打开由 `target` 做、空查询由一行 `submit` 拦截。左侧 `.bing-mark` 是纯 CSS 画的必应方块（`linear-gradient(135deg,#00bcf2,#008373)` + 白色小写 b）。右侧 `.hint` 是 `/` 快捷键提示。

**③ 问候行：公历 + 农历 + 时钟**

格式：`晚上好 · 10 月 27 日 星期二 · 农历九月十八 · 20:02`

农历**不查任何历法表**，直接用浏览器内置 ICU：

```js
const parts = new Intl.DateTimeFormat('zh-CN-u-ca-chinese', {
  month: 'long', day: 'numeric'
}).formatToParts(d);
// month → 「九月」「闰六月」「正月」「腊月」
// day   → 18（数字）
```

再用 30 个中文日名数组（初一…初十、十一…十九、二十、廿一…廿九、三十）拼成「农历九月十八」。已验证锚点：2026-02-17 = 正月初一、2025-07-25 = 闰六月初一、2027-02-01 = 腊月廿五。

不支持 ICU 的浏览器 `try/catch` 返回空串，这一段自动省略，其余照常显示。

时钟每 10 秒只更新 `#gtime` 的文本（不重建 DOM），只有跨小时才整行重建问候语——否则「下午好」会在 18:00 卡到下次刷新。

### 4.3 分组渲染与立体胶囊

`renderGroups()` 用字符串模板一次性拼出全部卡片，`main.innerHTML` 清空后逐组插入。每组设置 `animationDelay = gi * 55ms` 形成错落入场。

**卡片结构**（这是改样式的入口）：

```html
<a class="card" href="…" target="_blank" rel="noopener noreferrer">
  <span class="ic"><span class="lt">G</span></span>          <!-- 图标块 + 字母占位 -->
  <span class="mi">
    <span class="nm">GitHub</span>                            <!-- 标题 -->
    <span class="ds">github.com</span>                         <!-- 描述 -->
  </span>
  <span class="st" data-st="pending"><i class="dt"></i><b class="lb">离线</b></span>
  <svg class="arw">…</svg>                                    <!-- hover 浮出的箭头 -->
</a>
```

**立体胶囊的六层叠法**——这是全页最讲究的一段 CSS，缺任何一层都会「塌」成贴纸：

| # | 层 | 代码 | 作用 |
|---|---|---|---|
| ① | 三段渐变 | `linear-gradient(180deg,#fff 0%,var(--g-tint) 42%,var(--g-tint-2) 100%)` | 模拟圆柱受光，上白下深 |
| ② | 同色描边 | `0 0 0 1px var(--g-glow)` | 边缘不生硬 |
| ③ | 同色实心底边 | `0 2px 0 0 var(--g-tint-2)` | **药片侧壁厚度**，立体感的主要来源 |
| ④ | 近距软阴影 | `0 3px 3px -1px rgba(32,33,36,.06)` | 与纸面脱离 |
| ⑤ | 远距同色投影 | `0 7px 14px -5px var(--g-glow)` | 彩色环境光 |
| ⑥ | 内高光 + 内暗边 | `inset 0 1px 0 #fff` + `inset 0 -2px 4px -1px var(--g-glow)` | 顶部反光、底部凹陷 |

> **③ 必须用 `--g-tint-2`，不能用黑色。** 黑色底边会让胶囊像一张贴纸，这是最初版本失败的原因。

胶囊内的圆图标（26px）用「实色底 + 球面高光」做成立体小球：

```css
background-color: var(--g-fg);
background-image: radial-gradient(circle at 34% 28%, rgba(255,255,255,.52), transparent 64%);
box-shadow: inset 0 -1px 2px rgba(0,0,0,.22), 0 1px 3px var(--g-glow);
```

高光点偏移到 `34% 28%`（左上），这是球体受光的自然位置；放正中间会变成「塑料按钮」。

深色主题把胶囊换成「彩色薄雾」：顶部高光降到 `rgba(255,255,255,.16)`，投影全部换黑，否则彩色阴影在暗底上会发灰发脏。

### 4.4 favicon 系统

**取图优先级**（`favPlan()`）：

```
命中 FAV_LOCAL  →  icon/<host>.<ext>              ← 只走这一条，零请求
否则            →  ① https://favicon.im/<host>?larger=true
                   ② <origin>/favicon.ico
                   ③ https://favicon.yandex.net/favicon/<host>
```

localhost / 内网地址（`192.168.` / `10.` / `127.` 等）只用 ②，公共 CDN 拿不到内网图标。

**竞速而非串行**：每一级等 **900ms** 未返回，就并行启发下一级，谁先到用谁。这解决了「某个源挂住导致整批图标不显示」——串行的话 36 个站点会被一个慢源拖死。

**缓存**（`nav-fav-v2`）：成功存 30 天，失败存 3 天。`{u: url, t: 时间戳}`，`u: ''` 表示确认取不到，下次直接显示字母，不再重试。**本地图标 `icon/` 前缀的地址一律不写缓存**——否则换了图片文件，30 天内都不会变。

**本地化**（`icon/` 目录）：35 个图标，命名 `<host>.<ext>`，扩展名按真实文件头嗅探（不是按 Content-Type，站点经常给错）。抓取时按「SVG > 像素面积 > 字节数」择优，两轮升级后大部分站点拿到了 100px 以上的高清图；剩下豆瓣 32px、阮一峰 32px、雪球 48px、Stack Overflow 48px 是站点本身只提供这么大。`hf-mirror.com` 复用 `huggingface.co` 的高分 ico（同一品牌图形）。

**动画**：图标取到后不是简单淡入，而是带旋转的回弹：

```css
@keyframes favIn{
  0%   {opacity:0; transform:scale(.45) rotate(-16deg)}
  62%  {opacity:1}
  100% {opacity:1; transform:scale(1) rotate(0deg)}
}
/* .ic.has-fav img{animation:favIn .5s cubic-bezier(.34,1.56,.64,1) both} */
```

hover 时图标块 `scale(1.16) rotate(-5deg) translateY(-1px)`，内部 `img` 再叠一层 `scale:1.07`，形成「块+图」双层回弹。

> **为什么 img 用 `scale` 而不是 `transform`**：`animation ... both` 的 fill 会把 `transform` 锁在结束态（`none`），给 img 写 `transform` 完全不生效。`scale` 是独立属性（CSS Transforms L2），能与块级 `transform` 相乘叠加。

**加载时机**：`IntersectionObserver`（rootMargin 320px）——卡片靠近视口才发请求，首屏之外不抢带宽。

### 4.5 可达性圆点

**判断原理**（这是整个功能的核心，一句话）：

```js
await fetch(url, { mode:'no-cors', cache:'no-store', credentials:'omit',
                   referrerPolicy:'no-referrer', signal: ctrl.signal });
```

`no-cors` 下拿不到响应内容，但**只要服务器返回了任何响应**（含 404 / 500）Promise 就会 resolve；只有网络层失败（DNS 失败、拒绝连接、超时、证书错误）才 reject。这正好等于「站点是否可达」，不需要读响应体、不受 CORS 限制。

顺带解决了一个语义问题：**服务器活着但没有 favicon，不算离线**。

**四个参数决定体验**：

| 参数 | 值 | 原因 |
|---|---|---|
| `LIVE_TIMEOUT` | 4000ms | 太短误报多，太长整屏停在灰点 |
| 重试 | 失败后 900ms 再试一次 | 单次失败多半是抖动；本环境 GitHub / Hugging Face 时通时不通，单次误报率很高 |
| `LIVE_MAX` | 4 并发 | 一次打满连接会拖慢首屏 |
| `LIVE_TTL` | 5 分钟 | 刷新页面不重复打；够短，网络恢复后能及时变色 |

**并发闸门**是一个很短的队列：`liveJobs` 数组 + `liveRunning` 计数，`job().finally()` 里自减并递归续跑。

**状态渲染**：三态并存于一个 `.st` 元素上，只切 `data-st`：

| `data-st` | 圆点 | 附加 |
|---|---|---|
| `pending` | 灰点 + `livePulse` 呼吸（1.1s） | title「正在检测可达性」 |
| `on` | 绿点 `#1e8e3e` + 同色 2px 淡光 | title「可访问」 |
| `off` | 红点 `#d93025` + 「离线」二字淡入 | 卡片加 `.offline`：图标 `grayscale(.45) opacity(.8)`、标题转灰 |

「离线」二个字**预置在 HTML 里**，靠 CSS `display` 显隐，不由 JS 改文本——避免文本变化引起卡片布局抖动。离线卡片**链接依然可点**、hover 完全恢复，因为探测可能误判，不能真的锁死入口。

> **最隐蔽的坑：缓存键必须用 `card.href`。** `a.href` 会把 `http://localhost:1234` 规范化成带结尾斜杠的形式。如果用数据里的 `it.u` 作键存、用 `card.href` 作键读，会永远读不到缓存，每个站点每次刷新都重探。

### 4.6 键盘

| 键 | 行为 |
|---|---|
| `/` | 聚焦搜索框并全选（焦点在输入框时不触发） |
| `← →` | 在卡片间左右移动焦点 |
| `↑ ↓` | 上下移动一整行 |
| `Esc` | 清空搜索框 |
| `Enter` | 提交必应搜索 |

方向键的列数是运行时读出来的，换断点自动适配：

```js
const tpl = getComputedStyle(g).gridTemplateColumns;
return tpl.split(' ').filter(Boolean).length;
```

焦点不在卡片上时**不拦截**方向键，把默认滚动还给浏览器。焦点样式用 `:focus-visible` + 分组主色描边，鼠标点击不会留下框。

---

## 5. 响应式

只有两个断点，够用：

| 断点 | 变化 |
|---|---|
| `≤ 900px` | 卡片 3 列 → 2 列，`wide` 仍占 2 列（等于整行） |
| `≤ 640px` | 卡片 1 列；容器内距 24 → 16px；隐藏顶栏「本地模型」状态、搜索框 `/` 提示、分类关键词；胶囊 34 → 31px（图标 26 → 24、徽章 20 → 18）；图标块 42 → 38px（图 28 → 25）；页脚纵向堆叠左对齐 |

另有 `@media (prefers-reduced-motion: reduce)`：全局关掉 transition 与 animation。动画是锦上添花，不该让晕动症用户难受。

---

## 6. 参数总表

调体验只动这些常量，全部在 `<script>` 里集中声明。

| 常量 | 默认值 | 位置 | 作用 |
|---|---|---|---|
| `FAV_KEY` | `nav-fav-v2` | 图标模块 | 图标缓存键；改版本号 = 强制全体重探 |
| `FAV_TTL` | `30 * 864e5` | 图标模块 | 成功缓存 30 天 |
| `FAV_FAIL_TTL` | `3 * 864e5` | 图标模块 | 失败缓存 3 天 |
| 竞速超时 | `900` ms | `probe()` 内联 | 单源等待上限 |
| 图标 rootMargin | `320px` | `favIO` | 提前多少像素开始取图 |
| `LIVE_KEY` | `nav-live-v1` | 可达性模块 | 可达性缓存键 |
| `LIVE_TTL` | `5 * 60e3` | 可达性模块 | 结果有效期 5 分钟 |
| `LIVE_TIMEOUT` | `4000` | 可达性模块 | 单次探测超时 |
| `LIVE_MAX` | `4` | 可达性模块 | 最大并发 |
| 重试间隔 | `900` ms | `pingSite()` | 首次失败后等待 |
| 可达性 rootMargin | `220px` | `liveIO` | 提前多少像素开始探测 |
| `probeLM` 超时 | `1800` ms | 顶栏 | 本地模型探测 |
| `probeLM` 周期 | `60` s | `init` | 顶栏状态轮询 |
| 时钟周期 | `10` s | 底部 | 只更新时分 |
| 分组入场 | `55` ms/组 | `renderGroups()` | stagger |

---

## 7. 踩坑档案

按「现象 → 根因 → 正解」记录，都是实际调试中撞过的。**改这个页面前建议先扫一遍。**

| # | 现象 | 根因 | 正解 |
|---|---|---|---|
| 1 | 无头截图里图标全是字母 | `--virtual-time-budget` 把 10s 定时的虚拟时间瞬间跑完，网络请求还没回 | 改用 CDP 真实等待后再截图 |
| 2 | 注入 `.__h` 类模拟 hover，样式和真实效果不一样 | 复制规则和真实层叠顺序不同 | CDP `CSS.forcePseudoState({forcedPseudoClasses:['hover']})` |
| 3 | hover 时图标不放大 | `img` 上 `animation ... both` 把 `transform` 锁成 `none` | 改用独立属性 `scale:1.07` |
| 4 | 深色 hover 时图标块像被按下 | 用半透明 `--g-tint-2` 叠在暗底上，结果更暗 | 深色 hover 直接提亮到 `#f8fafd` |
| 5 | 胶囊底边发黑像贴纸 | 底边用了黑色 | 换成 `--g-tint-2` 同色 |
| 6 | 可达性结果从不复用，每次刷新全重探 | 存用 `it.u`、读用 `card.href`，`localhost` 被规范化加了斜杠 | 键与请求地址统一取 `card.href` |
| 7 | 顶栏「本地模型」状态永远停在「探测中」 | `probeLM()` 在 `renderGroups()` 之前跑，元素还不存在 | 挪到渲染之后（init 末尾） |
| 8 | 深色字母徽标看不清 | 浅底 + 亮色字母 | 新增 `--g-deep`，深色下用深色字母 |
| 9 | 大量站点误报离线 | 单次探测 + 网络抖动 | 4000ms 超时 + 900ms 后重试一次 |
| 10 | 深色下 GitHub / 少数派图标糊成一团 | 深色描线 favicon 落在黑卡片上 | 深色 `--ic-bg` 用浅色 `#dde3ea` |
| 11 | 搜索框右侧 `/` 提示被原生清除按钮压住 | `type=search` 的原生控件 | `::-webkit-search-cancel-button{display:none}` |
| 12 | 换了图标文件但页面不变 | 图标走了 localStorage 30 天缓存 | 本地 `icon/` 地址不写缓存（`if (!/^icon\//.test(src))`） |
| 13 | 窄屏截图看起来溢出，其实没溢出 | 无头模式最小视口约 504px，`--window-size=390` 是按 504 布局后裁切 | 别用无头窗宽判断窄屏溢出 |
| 14 | `--screenshot` 用相对路径报「拒绝访问」 | 无头 Edge 要求绝对路径 | 传绝对路径 |
| 15 | 图标全空白但不报错 | 某些站点返回 1×1 透明占位图 | `p.naturalWidth < 8` 视为失败，继续下一级 |

---

## 8. 验证方法

改完别只看一眼「好像对」，按这张单子过一遍。这套流程是前 11 轮迭代沉淀下来的，能抓住 90% 的回归。

### 8.1 第一层：语法

把主 `<script>` 抽成临时文件再交给 Node 解析（`<head>` 里那行内联主题脚本是单行闭合的，不会被这段匹配到）：

```bash
cd nav-page
sed -n '/^<script>$/,/^<\/script>$/p' index.html | sed '1d;$d' > /tmp/nav.js
node --check /tmp/nav.js && echo "语法 OK"
```

`node --check` 只做语法解析、不执行代码，能挡住漏括号、逗号、未闭合模板字符串这类改数据时最常见的错误。

### 8.2 第二层：截图清单

| 场景 | 检查点 |
|---|---|
| 亮色 1440px | 3 列整齐、图标全部渲染、胶囊有立体感 |
| 深色 | 图标托底为浅色、胶囊不发脏、文字对比够 |
| 520px | 单列、胶囊缩小、隐藏项确实隐藏 |
| 880px | 2 列、`wide` 占满整行 |
| hover 一张卡片 | 图标放大旋转、左缘竖线、标题变色、箭头浮出 |
| 断网 | 36/36 全红、无 pending 残留、图标照常显示（来自 `icon/`） |

### 8.3 第三层：CDP 真实链路

关键点是**必须真实等待**，不能用虚拟时间：

```
Target.createTarget          → 开空白页
Target.attachToTarget{flatten:true}
Page.navigate                → 打开 index.html
（sleep 真实 3–5 秒）         → 等图标与探测回来
Runtime.evaluate             → 统计 .ic.has-fav / .fallback / .st[data-st]
Page.captureScreenshot       → 出图
```

常用注入：

```js
// 切深色
document.documentElement.dataset.theme='dark'

// 断网（CDP 域）
Network.emulateNetworkConditions({offline:true, latency:0, downloadThroughput:0, uploadThroughput:0})

// 强制 hover
CSS.forcePseudoState({nodeId, forcedPseudoClasses:['hover']})
```

Node 22 自带全局 `WebSocket`，不需要装 puppeteer。临时脚本用完即删，别留在项目里。

### 8.4 回归基线（v11 实测）

- favicon：35/36 命中，仅 LM Studio（localhost 无 favicon）显示字母 L
- 可达性：36 站 24s 内全部出结果，31 绿 5 红（随网络变化）
- 断网模拟：36/36 全红，9s 内无 pending 残留
- 文件体积：`index.html` 42.1KB，`icon/` 908KB

---

## 9. 维护作业单

### 9.1 新增一个分类

```js
// 追加到 GROUPS 末尾
{
  id: 'new', name: '新分类', kw: '关键词 · 关键词',
  items: [
    { n:'站点', d:'example.com · 说明', u:'https://example.com', i:'示' },
    /* …建议凑满 6 个 */
  ]
}
```

配色自动取 `h(6 % 6)` = `h0`，即蓝色。**如果新增分组导致配色重复得难看**，就在 `.h6` 里再挂一套变量，并把 `gi % 6` 改成 `gi % 7`。

### 9.2 换一个站点图标

1. 下载目标图标（建议 ≥ 96px，SVG 最佳）；
2. 存为 `icon/<host>.<ext>`，**扩展名要与真实格式一致**；
3. `FAV_LOCAL` 里对应项改成新文件名（路径变了才需要改）；
4. 刷新——本地图标不进缓存，立刻生效。

### 9.3 换搜索引擎

```html
<form class="search" id="sf" action="https://www.google.com/search" method="get" target="_blank">
  <input id="q" name="q" type="search" placeholder="在 Google 中搜索">
```

把左侧 `.bing-mark` 一起换掉（否则视觉与引擎不符）。百度是 `https://www.baidu.com/s` + `name="wd"`。

### 9.4 备份与迁移

- **备份**：整个 `nav-page/` 目录复制走即可，无构建、无 node_modules、无隐藏依赖。
- **迁移到别的机器**：拷目录 → 双击。`file://` 下 localStorage 会重新开始（主题变默认亮色、图标重新探测一次），不影响功能。
- **换域名/路径**：`FAV_LOCAL` 里全是相对路径 `icon/...`，只要 `icon/` 与 `index.html` 同级就无需改动。

---

## 10. 演进时间线

11 轮需求，每一轮都记在这里，方便回溯「为什么现在是这个样子」。

| 轮次 | 需求 | 落地 |
|---|---|---|
| 1 | 完全不做本地筛选 | 删掉输入监听、文本匹配、空状态、`escapeHtml` |
| 2 | 去掉分类导航按钮 | 删掉 `.chips` / `renderChips()` / 数字快捷键 |
| 3 | 分类样式太单调 | 每组一套主色 + 组内同色图标 + hover 左缘竖线 |
| 4 | 调研优秀起始页 | 采纳：问候行、方向键导航、`focus-visible`、字母弹跳；**刻意不抄**毛玻璃/粒子/命令面板 |
| 5 | 精简 | 删「最近访问」行、删统计、删页脚按键说明、删南阳时间 |
| 6 | 日期加农历/星期/时钟 | `Intl` 农历 + 单行 `tabular-nums`；页脚合并为单行左右分布 |
| 7 | 用真实 favicon | 三级源 + 900ms 竞速 + localStorage 30/3 天缓存 |
| 8 | 图标落地本地 | `icon/` 35 个 + `FAV_LOCAL` 映射，命中即零请求 |
| 9 | 图标更醒目、有动画、更大 | 42px 块 + `favIn` 回弹 + hover 双层放大 |
| 10 | 分类标题改立体胶囊 | 六层阴影 + 球面高光图标 + 实心徽章 |
| 11 | 卡片加可达性圆点 | `no-cors` 探测 + 4 并发 + 重试 + 5 分钟缓存 |

**被否定并删除的东西**（别再提）：本地文本筛选、分类切换按钮、最近访问、入口统计、页脚按键说明、南阳时间、LM Studio 专用 badge。

---

## 11. 已知限制

| 限制 | 原因 | 影响 |
|---|---|---|
| 单个站点偶发误报离线 | 探测依赖网络往返 | 卡片仍可点，hover 恢复，无实际危害 |
| `file://` 下部分站点的 `no-cors` 探测被拦 | 浏览器对本地文件的安全策略 | 表现为常红；起个本地 http 服务即正常 |
| LM Studio 没有图标 | `localhost:1234` 不提供 favicon | 显示字母 L，属预期 |
| 农历依赖 ICU | 老浏览器可能不支持 | 该段自动省略，不报错 |
| `icon/` 与 `index.html` 必须同级 | 使用相对路径 | 分离后图标回退在线源，不空白 |
| 页面不会自动更新站点状态 | 只在加载时探测一次 | 需要最新状态就刷新（缓存 5 分钟） |

---

## 附：一句话总结

**把所有复杂度都关在 `<script>` 里，让用户看到的只有一个居中的 logo、一个搜索框，和一层安静的卡片。** 加了图标、胶囊、圆点之后仍然「像 Google」，是因为每个新元素都被压到了同一套色彩与尺度体系里——这是这个页面唯一需要守住的纪律。
