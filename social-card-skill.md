> copyright: @op7418(歸藏)
> from: guizang-social-card-skill

# 通用社交卡片生成 Skill

## 目标

创建适用于小红书 / Rednote、微信公众号、文章封面、平台缩略图、产品更新图、教程轮播图等场景的高质量社交图片包。

这个 Skill 的目标不是把文字硬塞进海报，而是把原始内容转化为清晰、有证据、有节奏的视觉表达。

生成物通常包括：

- 小红书 / Rednote 3:4 轮播图：封面 + 多页内容卡。
- 微信公众号封面：一张 `21:9` 主封面 + 一张 `1:1` 方形封面。
- 文章封面、产品更新封面、平台缩略图。
- 截图讲解类、旅行攻略类、职场方法类、教程类、数据解读类图片。

## 适用范围

适合使用本 Skill 的任务：

- 社交媒体图文卡片 / 轮播图。
- 公众号封面组合。
- 教程、攻略、清单、路线、对比、产品更新、知识解释。
- 有文本内容，需要提炼成视觉结构。
- 有截图、照片、产品图、路线图、数据表，需要重新组织成可读社交图片。

不适合使用本 Skill 的任务：

- 完整 PPT / slide deck。
- 长视频、动态视频、复杂动画。
- 单纯图片修图且不需要版式设计。
- 纯摄影作品集，即图片本身就是唯一交付物。
- 用户明确要求某种与本 Skill 风格完全不符的强装饰风格，例如 Y2K、kawaii、哥特萝莉、梦核装饰风等。遇到这种情况，应说明本 Skill 更适合信息型、杂志型、攻略型表达，并建议切换到其他视觉系统。

## 语言规则

始终使用用户当前对话语言回复。

- 用户用中文，所有提问、进度、交付、错误说明都用中文。
- 用户用英文，所有沟通都用英文。
- 图片内文字默认使用用户提供内容的语言。

## 运行环境适配

本 Skill 应运行在多种代理和本地环境中，包括 Codex、Claude Code、本地 Node.js / Python、带浏览器自动化能力的沙箱等。

不要把流程绑定到某一个运行时。执行时根据当前环境能力选择实现方式：

- 如果有浏览器自动化能力，优先用 Playwright、Puppeteer 或环境内置浏览器截图工具渲染 HTML。
- 如果允许启动本地服务器，可在需要路由、模块打包或远程资源时使用本地服务器。
- 如果不能绑定端口，使用 `file://` 协议渲染静态 HTML。
- 如果浏览器渲染不可用，可用 Pillow / PIL 做简化静态渲染，但不要承诺与 HTML/CSS 完全一致。
- 如果环境提供 AI 生图 / 修图工具，使用当前环境可用工具；不要写死某个 MCP 工具名。
- 如果环境提供文件上传工具，可上传最终 PNG；否则返回绝对路径或环境支持的图片预览。
- 如果内存有限，超过 6 页的长轮播应分批渲染。

### 依赖策略

优先使用环境已有依赖。缺失时，只安装完成任务所必需的依赖。

Node 渲染常用依赖：

```bash
npm install playwright
npx playwright install chromium
```

Python 兜底渲染常用依赖：

```bash
python -m pip install pillow
```

不要默认污染用户 HOME 目录；依赖应安装在项目目录、任务目录、虚拟环境或当前运行时推荐位置。

## 字体策略

优先使用高质量本地字体，但不要假设固定系统路径一定存在。

推荐字体优先级：

- 西文无衬线：Inter、Helvetica Neue、Arial、system sans-serif。
- 中文无衬线：Noto Sans CJK SC、Source Han Sans SC、PingFang SC、Microsoft YaHei。
- 中文衬线：Noto Serif CJK SC、Source Han Serif SC、Songti SC。
- 等宽字体：IBM Plex Mono、JetBrains Mono、SF Mono、DejaVu Sans Mono。

CSS 字体栈示例：

```css
--sans: Inter, "Helvetica Neue", Arial, "PingFang SC", "Noto Sans CJK SC", sans-serif;
--serif: "Noto Serif CJK SC", "Source Han Serif SC", "Songti SC", serif;
--mono: "IBM Plex Mono", "SF Mono", Menlo, Consolas, monospace;
```

如果使用 Pillow 并且系统里存在 Noto Sans CJK `.ttc` 文件，简体中文通常使用 `index=2`：

```python
font = ImageFont.truetype(
    "/usr/share/fonts/opentype/noto/NotoSansCJK-Regular.ttc",
    32,
    index=2
)
```

如果字体不存在，应探测系统字体并选择最接近的替代，不要因为某个字体缺失而停止任务。

## 平台规格

默认规格：

- 小红书 / Rednote 3:4：`1080 × 1440`。
- 小红书 / Rednote 1:1：`1080 × 1080`。
- 微信公众号主封面 21:9：建议 `1260 × 540` 或按用户平台要求输出。
- 微信公众号方形封面 1:1：`1080 × 1080`。
- 通用文章封面：按用户指定比例；未指定时优先 `16:9` 或平台推荐尺寸。

文件命名建议：

- 小红书轮播：`page-01.png`、`page-02.png`、...
- 微信主封面：`wechat-wide.png`
- 微信方形封面：`wechat-square.png`
- 总览检查图：`contact-sheet.png` 或 `preview.png`

## 核心原则

每一页都只解决一个视觉问题。

为每一页决定：

- 观众一眼要理解什么？
- 哪张图、截图、数据或结构能支撑这个判断？
- 哪些字必须变大？
- 哪些字应该变成注释、角标、表格项或正文外内容？
- 哪些细节应该留给发布正文，而不是塞进图片？

图片不是长文排版。卡片应承载钩子、结论、路线、对比、清单、证据和关键步骤。

## 风格模式

一次任务选择一种主风格，不要在同一套图里随意混用。

### 1. Swiss International / 信息型

特点：

- 严格网格、左对齐、细线、数字、表格、路线、清单。
- 大标题使用轻字重；小标签、编号、元信息可以使用等宽字体。
- 适合攻略、路线、产品更新、数据解读、职场方法、教程步骤。
- 气质应清晰、克制、系统、可扫描。

常用元素：

- 时间轴。
- 路线表。
- KPI 数字。
- 对比矩阵。
- 横向条形图。
- 编号陈述。
- 截图框 / 产品图框。

### 2. Editorial Magazine / 杂志型

特点：

- 更强叙事感、纸感、图片氛围、留白、引文、注释、边栏。
- 可使用中文衬线或宋体做标题，正文使用安静的无衬线。
- 适合旅行故事、人物/品牌文章、深度观点、生活方式、影像化主题。
- 气质应像一页被认真编辑过的杂志，而不是普通海报。

常用元素：

- 大图井。
- 引文。
- 边注。
- 章节编号。
- 图注。
- 纸纹 / 轻微颗粒。
- 非对称版式。

### 3. 选择方式

不要只按内容分类选风格，应按表达目的选择：

- 需要快速看懂、保存、照着执行：选信息型。
- 需要情绪、氛围、叙事、阅读感：选杂志型。
- 两者都可行时，问自己：这是“操作说明”还是“特写文章”？

## 工作流

### 1. 信息收集

只询问会真正改变输出的信息。

必须明确：

- 目标平台和比例。
- 主题、标题、原文、要点或素材。
- 页数要求；如果用户不确定，由你决定。
- 图片来源。
- 风格偏好；如果用户不确定，由你判断。
- 必须保留或禁止出现的内容。

如果用户只提供文字、没有任何图片或截图，设计前只问一次：

```text
这篇我需要 1-2 张图。三种走法：
A. 你自己有照片 / 截图，传给我（推荐，最不像 AI 图）
B. 我帮你找可用的网络图片
C. 用 AI 生成
```

用户选择后继续执行，不要反复追问。

对于新闻、产品发布、政策、价格、型号、榜单、实时数据等可能变化的信息，必须先核实事实，再进入设计。

### 2. 内容提炼

把原文转成页面计划。

小红书 / Rednote：

- 第 1 页是封面钩子。
- 第 2 页开始每页一个主题。
- 大多数内容使用 5-9 页。
- 信息过少时减少页数，不要硬凑。
- 信息过多时压缩文案，把细节留给正文。

微信公众号封面：

- 必须同时做 `21:9` 主封面和 `1:1` 方形封面。
- 两个比例应分别构图，不要直接裁切。
- 主封面可以保留完整标题、短副标题和视觉主体。
- 方形封面应使用更短标题，通常不放复杂副标题。

页面计划格式：

```text
Page 01 / 封面 / 核心钩子 / 图片来源 / 版式意图
Page 02 / 主题 / 关键文案 / 视觉证据 / 版式意图
Page 03 / 主题 / 关键文案 / 视觉证据 / 版式意图
```

如果用户要求先确认计划，先展示页面计划再制作。否则内部规划后直接执行。

### 3. 文案压缩

图片内文案应短、准、能扫读。

原则：

- 标题优先表达结论，不要写成目录。
- 每页正文尽量不超过 3-5 个信息块。
- 每个信息块优先使用一句话。
- 复杂背景、注意事项、个人感受放到发布正文。
- 不要把表格原文逐字搬进图片。

标题长度建议：

- 3:4 封面主标题：8-18 个中文字符最稳。
- 内容页标题：8-22 个中文字符。
- 方形封面标题：4-12 个中文字符。
- 超过长度时，拆行或改写，不要缩到难读。

### 4. 页面密度

3:4 社交卡必须吃满画布。

要求：

- 内容覆盖至少约 75% 的画布高度。
- 不允许把内容挤在中间，上下留下大片空白。
- 任何超过 15% 画布高度的空白带都需要明确理由，例如大图呼吸、单句宣言式页面、段落上下留白。
- 页脚应固定在自然文流底部，避免与内容碰撞。
- 每张图单独看都应完整，不依赖前后页才能成立。

### 5. 模板与 HTML

优先使用当前 Skill 或项目中已有的 seed template。

如果存在模板：

- 信息型使用信息型模板。
- 杂志型使用杂志型模板。
- 只替换 poster 区域和任务级自定义样式。
- 不要混用两个模板的类名系统。

如果模板不存在：

- 创建一个自包含 `index.html`。
- 明确写入尺寸、字体栈、颜色 token、基础组件和页面结构。
- 保持 CSS 可读，任务级样式集中放置。
- 重置默认样式，例如 `figure { margin: 0; }`，避免浏览器默认间距造成漂移。

通用目录结构：

```text
social-card-<slug>/
  index.html
  render.cjs
  assets/
    SOURCES.md
  output/
    page-01.png
    page-02.png
```

## 图片与截图处理

### 用户提供图片

用户提供的照片、截图、产品图优先使用。

处理原则：

- 不拉伸图片。
- 不裁掉人脸、关键 UI、产品主体、硬件细节。
- 对截图优先保证可读性。
- 截图信息重要时，放大截图区域，减少附近装饰和说明文字。
- 每张图片都要设置合理裁切位置，例如 CSS `object-position`。

### 文字压在图片上

当文字位于照片或大图上时：

- 先判断图片是否有安静区域。
- 标题应避开人脸、主体、产品、关键 UI。
- 先尝试不用遮罩。
- 如果缩略图不可读，再增加局部色块、局部渐变或图片同色系遮罩。
- 不要默认给整张图加厚重黑色遮罩。
- 渲染后缩到约 360px 宽检查标题是否仍然清楚。

建议在 HTML 中用注释记录主体位置：

```html
<!-- Subject map: face at center-right, safe title zone in top-left. -->
```

### 没有图片时

只有在用户选择 B 或 C 后才进入对应流程。

B. 网络图片：

- 优先选择来源清晰、许可明确或平台允许使用的图片。
- 下载到 `assets/`。
- 用用途命名，不用随机 hash，例如 `hero-mountain.jpg`。
- 在 `assets/SOURCES.md` 记录图片来源 URL、作者、站点、许可说明。
- 交付时说明来源和未确认风险。

C. AI 生图：

- 使用当前环境可用的图片生成能力。
- 生成图只用于确实增强表达的页面，通常 1-2 张足够。
- 提示词描述画面角色，不要要求生成标题、页码、logo、水印或假 UI。
- 不用 AI 图冒充真实截图、真实产品发布或真实地点证据。

### 网络图片来源建议

可考虑：

- Unsplash：户外、生活方式、氛围图。
- Pexels：生活方式、本地化关键词、通用场景。
- Flickr Creative Commons：纪实、真实场景、街景、室内。
- 官方媒体包 / 品牌素材库：产品、应用、公司发布图。
- 直接网页搜索：具体地点、产品、游戏、新闻事件。

不要从付费图库或版权明显不明的平台直接拿图当作最终可商用素材。

## 渲染

优先使用浏览器渲染 HTML 并截图 `.poster` 或 `.cover` 节点。

通用 `render.cjs` 示例：

```js
const path = require("path");
const fs = require("fs");

let playwright;
try {
  playwright = require("playwright");
} catch {
  throw new Error("Playwright is not available. Install it or use another browser screenshot tool.");
}

const { chromium } = playwright;

(async () => {
  const taskDir = process.argv[2] ? path.resolve(process.argv[2]) : process.cwd();
  const htmlPath = path.resolve(taskDir, "index.html");
  const outputDir = path.resolve(taskDir, "output");
  fs.mkdirSync(outputDir, { recursive: true });

  const browser = await chromium.launch();
  const page = await browser.newPage({
    viewport: { width: 1280, height: 1800 },
    deviceScaleFactor: 1,
  });

  await page.goto("file://" + htmlPath);
  await page.waitForLoadState("networkidle");
  await page.evaluate(async () => {
    if (document.fonts) await document.fonts.ready;
    await Promise.all(
      Array.from(document.images).map((img) =>
        img.decode ? img.decode().catch(() => undefined) : undefined
      )
    );
  });
  await page.waitForTimeout(500);

  const posters = await page.$$(".poster, .cover");
  for (let i = 0; i < posters.length; i++) {
    const el = posters[i];
    const id = (await el.getAttribute("id")) || `page-${String(i + 1).padStart(2, "0")}`;
    await el.screenshot({ path: path.join(outputDir, `${id}.png`) });
  }

  await browser.close();
  console.log(`Rendered ${posters.length} images to ${outputDir}`);
})();
```

如果 Playwright 自带 Chromium 不存在，可根据当前环境尝试系统 Chrome / Edge，或使用环境内置浏览器截图工具。

如果浏览器完全不可用，使用 Pillow 兜底：

```python
from PIL import Image, ImageDraw, ImageFont

img = Image.new("RGB", (1080, 1440), (246, 244, 237))
draw = ImageDraw.Draw(img)

# CJK 文本需要手动测量和换行
for char in text:
    bbox = draw.textbbox((0, 0), current_line + char, font=font)
    if bbox[2] - bbox[0] > max_width:
        lines.append(current_line)
        current_line = char
    else:
        current_line += char

img.save("output/page-01.png")
```

注意：Pillow 兜底适合简单静态版式，不适合复刻复杂 CSS、WebGL、滤镜、响应式布局。

## QA 检查

渲染后必须做轻量检查：

- 每张 PNG 尺寸是否正确。
- 文字是否溢出。
- 页脚是否与正文或图片碰撞。
- 标题在缩略图下是否可读。
- 3:4 页面是否吃满画布。
- 图片主体是否被裁坏。
- 截图、UI、产品细节是否仍可辨认。
- 是否有浏览器默认 margin 导致布局漂移。
- 是否出现错别字、乱码、字体 fallback 异常。
- 是否使用了未说明来源的网络图片。

如果项目提供自动检查脚本，可在用户要求或最终定稿前运行，例如：

```bash
node validate-social-deck.mjs <task-dir>
```

自动检查失败时应修复后重新渲染。警告项需要阅读并判断是否影响交付。

## 交付流程

默认先展示结果，再询问是否自动核查。

流程：

1. 渲染完成后，展示输出图片或输出路径。
2. 说明图片尺寸和已做检查。
3. 问用户：`先你自己看，还是我先自动核查一遍？`
4. 如果用户要自动核查，运行检查脚本，修复失败项，重新渲染。
5. 最终交付时说明输出目录、图片列表、尺寸、验证状态、图片来源和未解决风险。

根据环境选择交付方式：

- 桌面 / Codex 类环境：返回绝对路径，并在支持时内联展示图片。
- 托管环境：使用可用上传工具上传 PNG。
- CLI 环境：返回输出目录和文件名。

## 不可违反的规则

- 不伪造数据、百分比、价格、发布信息、新闻事实。
- 不把 AI 图伪装成真实截图或真实证据。
- 不拉伸图片。
- 不裁掉人脸、关键 UI、产品主体和重要文字。
- 不让文字贴边、溢出或压到页脚。
- 不让 3:4 卡片出现无理由的大面积空白。
- 不在图片里写可见的使用说明、快捷键说明或制作提示。
- 不做无意义装饰，例如随机圆点、色块、贴纸、形状堆砌。
- 不使用嵌套卡片作为默认布局。
- 不把 21:9 封面直接裁成 1:1，必须分别构图。
- 信息型大标题不要使用过粗字重；大字号越大，字重应越轻。
- 杂志型页面不能只有衬线标题和空背景，必须有真实的图像、纸感、注释、引文或版式层次支撑。
- 网络图片必须记录来源。
- 用户已提供足够信息时，直接执行；只有缺失关键信息才提问。

## 推荐输出说明格式

```text
已完成一版，共 N 张。

输出目录：
<absolute-path>/output

规格：
- page-01.png — 1080×1440
- page-02.png — 1080×1440

检查：
- 尺寸已确认
- 已做快速视觉检查
- 尚未运行完整自动核查

图片来源：
- 用户提供 / 网络来源 / AI 生成
- 来源记录见 assets/SOURCES.md

先你自己看，还是我先自动核查一遍？
```
