# 书A SVG 审计报告

审计范围：`/Users/xiaokai/gamedesign/ebook/images/` 下全部 SVG 文件。
审计方式：`xmllint --noout` 语法校验、根 `<svg>` 元素属性检测（多行感知正则）、Markdown 图号 / `<img src>` 引用交叉比对、文件大小筛查、代表性 SVG 内容抽查。**仅审计，未修改任何 SVG。**

## 概览
- 总文件数: 100
- 语法错误: 11
- 缺失属性: 32（均为根 `<svg>` 缺 `width` + `height`，`xmlns`/`viewBox` 齐全）
- 编号不匹配: 0（硬性不匹配：所有 "图 N.M" 引用都能找到对应 `chNN-figMM` 文件；所有 SVG 都被 `<img src>` 引用，无孤立文件）
  - 但有 28 个 SVG 缺 "图 N.M" 图注（仅以 `<img>` 嵌入，未编号），属图注规范不一致，详见第 3 节
- 其他问题: 3
  1. ch17 / ch18 的 `<img>` 标签未写 `width`，配合 SVG 缺 `width/height`，存在按 300×150 默认尺寸渲染的风险
  2. ch25-fig01 第 202 行 `<rect>` 残留 `<circle>` 的 `cx/cy/r` 属性（渲染忽略，但属标记瑕疵）
  3. ch23 出现跨章图号引用（图 19.1 / 19.2 / 20.1，目标文件均存在，非错误，仅提示）

## 1. 语法错误

共 11 个文件无法通过 `xmllint --noout`（XML 不良构，浏览器以 `<img>` 加载时按 XML 解析，会导致整图无法渲染）。错误分三类：

### 1.1 HTML 实体未定义（XML 只认 `&lt; &gt; &amp; &quot; &apos;`）— 9 个文件
| 文件 | 未定义实体 |
|---|---|
| `ch06/ch06-fig01-pixel-line-ratios.svg` | `&ensp;`、`&deg;` |
| `ch06/ch06-fig02-jaggies-compare.svg` | `&ensp;`、`&cross;`、`&check;` |
| `ch06/ch06-fig03-line-emotions.svg` | `&ensp;` |
| `ch06/ch06-fig04-curve-smooth-sequence.svg` | `&ensp;` |
| `ch08/ch08-fig02-five-ratios.svg` | `&ldquo;`、`&rdquo;`（多处） |
| `ch08/ch08-fig04-breathing-point.svg` | `&ldquo;`、`&rdquo;` |
| `ch11/ch11-fig02-dithering-modes.svg` | `&ensp;` |
| `ch14/ch14-fig01-full-analysis-template.svg` | `&ensp;` |
| `ch14/ch14-fig02-priority-debug.svg` | `&ensp;` |

修复方向：将 `&ensp;`→`&#8192;`（或空格）、`&deg;`→`°`（或 `&#176;`）、`&ldquo;`→`“`（或 `&#8220;`）、`&rdquo;`→`”`（或 `&#8221;`）、`&cross;`→`✗`、`&check;`→`✓`。

### 1.2 未转义的 `&`（`xmlParseEntityRef: no name`）— 1 个文件
- `ch32/ch32-fig01-fight-cycle.svg` 第 10 行：`Aidan Helfant — Focused Intentional Growth &` 末尾的 `&` 后面跟空白，应改为 `&amp;`。

### 1.3 未转义的 `<`（`StartTag: invalid element name`）— 1 个文件
- `ch26/ch26-fig03-level-visual-rhythm.svg` 第 158 行：文本 `1. 紧张区 < 关…` 中的 `<` 被当作标签起始，应改为 `&lt;`。

> 注：ch06 全部 4 张图、ch08 的 fig02/fig04 均在此列，意味着这两章相当一部分插图实际无法渲染，优先级高。

## 2. 缺失属性

根 `<svg>` 元素属性要求：`xmlns`、`viewBox`、`width`、`height` 四项齐全。
- 全部 100 个文件都有 `xmlns` 和 `viewBox`。
- 32 个文件缺 `width` 和 `height`（仅有 `xmlns` + `viewBox`）。按章分布：

| 章 | 缺失文件数 | 文件 |
|---|---|---|
| ch07 | 4 | fig01-shape-psychology / fig02-silhouette-test / fig03-silhouette-to-views / fig04-form-three-methods |
| ch08 | 4 | fig01-positive-negative / fig02-five-ratios / fig03-depth-five-layers / fig04-breathing-point |
| ch09 | 5 | fig01-value-three-zones / fig02-four-templates / fig03-before-after-desat / fig04-three-point-lighting / fig05-game-value-use |
| ch10 | 7 | fig01-color-function-pyramid / fig02-hsv-model / fig03-limited-palette-spectrum / fig04-gamut-mask / fig05-albers-experiment / fig06-color-script-template / fig07-color-vocabulary |
| ch17 | 3 | fig01-blender-five-steps / fig02-2d3d-hybrid / fig03-pbr-cheatsheet |
| ch18 | 2 | fig01-asset-lifecycle / fig02-naming-convention |
| ch31 | 2 | fig01-48hour-sprint / fig02-postmortem-template |
| ch32 | 2 | fig01-fight-cycle / fig02-minimum-commitment |
| ch33 | 3 | fig01-triple-personas / fig02-decision-triangle / fig03-outsource-decision |

> 渲染影响：`<img>` 标签若带了 `width="800"`（ch07/ch09/ch10/ch31/ch32/ch33 多数如此）或 `style="width:100%;max-width:800px"`，SVG 会按 `viewBox` 比例缩放，影响不大；但 **ch17、ch18 的 `<img>` 未写 `width`**，SVG 又无固有宽高，浏览器会回落到替换元素默认尺寸 300×150，把 800×500 的图压扁，属实际可见缺陷。

## 3. 编号不匹配

校验逻辑：Markdown 中的 "图 N.M" → 期望文件 `images/chNN/chNN-figMM-*.svg`（NN=N 两位、MM=M 两位）；并反向检查每个 SVG 是否被任一 Markdown 的 `<img src="images/…">` 引用。

- 引用了但无对应文件：**0**
- 有文件但无任何 Markdown 引用（孤立）：**0**（100 个 SVG 全部被 `<img src>` 引用）
- 图号与文件名编号不一致：**0**（凡出现 "图 N.M" 的，对应 `chNN-figMM` 文件均存在且编号吻合）

### 3.1 图注规范不一致（28 个 SVG 缺 "图 N.M" 编号）
以下章节的 SVG 仅以 `<img src="…">` 嵌入，Markdown 中没有 "图 N.M" 形式的图注文案，与其余章节的编号惯例不一致（非断链，仅规范问题）：

| 章 | 缺图注文件数 | 文件编号 |
|---|---|---|
| ch07 | 4 | fig01–fig04 |
| ch09 | 5 | fig01–fig05 |
| ch10 | 7 | fig01–fig07 |
| ch17 | 3 | fig01–fig03 |
| ch18 | 2 | fig01–fig02 |
| ch31 | 2 | fig01–fig02 |
| ch32 | 2 | fig01–fig02 |
| ch33 | 3 | fig01–fig03 |

附录的 `app-fig01-learning-path.svg`、`app-fig02-radar-tracker.svg` 经 `appendix-F-推荐书单.md` 的 `<img>` 引用，附录本不使用 "图 N.M" 编号，不计入不一致。

### 3.2 跨章图号引用（提示，非错误）
`ch23-风格选择与混血.md` 除引用自身的 图 23.1 / 23.2 外，还引用了 图 19.1、19.2、20.1（对应 ch19/ch20 的文件均存在），属正常的跨章指代。

## 4. 空 / 占位文件

无。全部 100 个文件均 ≥ 2606 字节（远高于 500 字节阈值），最小者为 `ch32/ch32-fig02-minimum-commitment.svg`（2606 字节，40 行，含三层同心圆 + 标注 + 底部原则条的完整示意图，非占位）。文件大小分布：最小 2606、最大 26786（`ch13-fig02-pixel-sub-styles.svg`），无异常小文件。

## 5. 内容疑似错误（抽查）

逐字通读 5 个代表文件：ch06-fig01、ch09-fig02、ch12-fig03、ch25-fig01，外加最小文件 ch32-fig02。结论：**内容与所在章节主题一致，未发现图号错标、数值错误或明显的图文错位**；发现 1 处标记瑕疵、若干文字偏长但均在画布内。

| 抽查文件 | 与章节匹配 | 发现 |
|---|---|---|
| `ch06/ch06-fig01-pixel-line-ratios.svg`（线条章） | 是 | 三种斜率 1:1/2:1/1:0 标注正确，角度 45°、≈26.6°（arctan 0.5）、0°/90° 准确；底部"程序员类比 CSS Grid"贴合程序员读者。**但**含 `&ensp;`/`&deg;` 实体错误（见第 1 节），实际无法渲染。长标签均在 800 宽画布内。 |
| `ch09/ch09-fig02-four-templates.svg`（明度章） | 是 | 高调/低调/高对比/低对比四模板及配对游戏案例（旅途·星露谷 / Inside·Limbo / 茶杯头·哈迪斯 / Gris）均合理；无语法错误；缺 width/height（img 带 width=800，渲染正常）。 |
| `ch12/ch12-fig03-rule-of-thirds.svg`（构图章） | 是 | 三分线位置 306/493（横）与 180/300（纵）相对 120..680 / 60..420 框架正好三等分，四个交点 A/B/C/D 标注正确；Z 型视线流与 CSS Grid 类比一致。四属性齐全、无语法错误。 |
| `ch25/ch25-fig01-four-floor-character.svg`（角色设计章） | 是 | 四层楼（剪影→内部结构→细节→色彩）层层递进，"不过关不上楼"箭头与各层检验标准齐全；8 色色板实有 8 色。**标记瑕疵**：第 202 行 `<rect cx="0" cy="0" r="3" x="78" y="84" width="5" height="6" fill="#F1C40F"/>` 残留了 `<circle>` 的 `cx/cy/r` 属性（应为腰带扣的圆，现渲染成 5×6 小矩形；多余属性被忽略，不影响良构）。 |
| `ch32/ch32-fig02-minimum-commitment.svg`（持续输出章，最小文件） | 是 | 三层同心圆（每天 5 分钟 / 每周 1 张稿 / 每月 1 次社区发布）+ "积累/分享"箭头 + 底部原则条；`>` 已正确转义为 `&gt;`；内容完整，**非占位**。缺 width/height。 |

## 6. 按章节通过情况

判定规则：凡存在语法错误或缺失必需属性即记"有问题"；编号 / 图注问题在第 3 节单独说明，不单独导致"有问题"（ch25 的 rect 标记瑕疵记为脚注，不影响通过判定）。

| 章 | SVG 数 | 状态 | 主要问题 |
|---|---|---|---|
| ch00 | 2 | 通过 | — |
| ch01 | 3 | 通过 | — |
| ch02 | 4 | 通过 | — |
| ch03 | 2 | 通过 | — |
| ch04 | 2 | 通过 | — |
| ch05 | 2 | 通过 | — |
| ch06 | 4 | 有问题 | 4 张全部语法错误（`&ensp;`/`&deg;`/`&cross;`/`&check;`） |
| ch07 | 4 | 有问题 | 4 张缺 width/height；无"图 N.M"图注 |
| ch08 | 4 | 有问题 | 4 张缺 width/height；fig02/fig04 语法错误（`&ldquo;`/`&rdquo;`） |
| ch09 | 5 | 有问题 | 5 张缺 width/height；无"图 N.M"图注 |
| ch10 | 7 | 有问题 | 7 张缺 width/height；无"图 N.M"图注 |
| ch11 | 3 | 有问题 | fig02 语法错误（`&ensp;`） |
| ch12 | 5 | 通过 | — |
| ch13 | 3 | 通过 | — |
| ch14 | 2 | 有问题 | 2 张全部语法错误（`&ensp;`） |
| ch15 | 2 | 通过 | — |
| ch16 | 3 | 通过 | — |
| ch17 | 3 | 有问题 | 3 张缺 width/height；无"图 N.M"图注；`<img>` 未写 width（渲染风险） |
| ch18 | 2 | 有问题 | 2 张缺 width/height；无"图 N.M"图注；`<img>` 未写 width（渲染风险） |
| ch19 | 2 | 通过 | — |
| ch20 | 2 | 通过 | — |
| ch21 | 2 | 通过 | — |
| ch22 | 3 | 通过 | — |
| ch23 | 2 | 通过 | 含跨章图号引用（19.1/19.2/20.1，目标存在） |
| ch24 | 2 | 通过 | — |
| ch25 | 3 | 通过 | fig01 第 202 行 rect 残留 circle 属性（标记瑕疵，不影响渲染） |
| ch26 | 3 | 有问题 | fig03 语法错误（未转义 `<`） |
| ch27 | 3 | 通过 | — |
| ch28 | 3 | 通过 | — |
| ch29 | 2 | 通过 | — |
| ch30 | 2 | 通过 | — |
| ch31 | 2 | 有问题 | 2 张缺 width/height；无"图 N.M"图注 |
| ch32 | 2 | 有问题 | 2 张缺 width/height；fig01 语法错误（未转义 `&`）；无"图 N.M"图注 |
| ch33 | 3 | 有问题 | 3 张缺 width/height；无"图 N.M"图注 |
| appendix | 2 | 通过 | app-fig01/02，经 appendix-F 的 `<img>` 引用 |

**汇总：35 个目录（ch00–ch33 + appendix）中，13 章有问题、22 章通过（含 appendix）。**
