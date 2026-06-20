# 像素艺术整合书 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将书 A（程序员游戏美术课）与书 B（像素艺术完全指南）整合为一本以像素为核心、书 A 声调、全书原创 SVG 插图的中文像素艺术教材。

**Architecture:** 以书 A 六部→坍缩五部为骨架（观察·练手·风格·制作·继续），五子能力坍缩为三能力（看/做/整）。书 B 全部内容融入对应章节并全面改写为书 A 程序员声调。全书插图统一原创 SVG（像素用 `<rect>` 网格还原，概念图沿用书 A 风格，界面画简化示意图）。10 阶段推进：搭骨架→SVG审计→按部写作（含SVG）→附录→全局重穿→通读校对。

**Tech Stack:** Markdown 书稿；原创 SVG 插图（`xmllint` 校验）；`rg` 做文风/路径/编号一致性检查；git 版本控制。

**配套文档:** `/Users/xiaokai/pixel-art/SPEC.md` —— 逐章合并方案（第五节）、文风规范（第六节）、图像策略（第七节）是本计划的依据。

**文风列图例（来自 SPEC）：**
- A原声 = 书 A 主体保留，仅像素重定向/编号/轻改
- A声+B改写 = 书 A 框架 + 书 B 融入改写为 A 声调
- A声改写 = 书 B 主体（A 无对应），全面改写为 A 声调

**通用验证命令（全书适用）：**
- 路径检查：`rg "extracted/|placeholder_" /Users/xiaokai/pixel-art/` → 应无结果（禁用提取位图与占位图引用）
- 文风检查：`rg "章节来源|B1_p|B2_p|B3_p" /Users/xiaokai/pixel-art/` → 应无结果（书 B 痕迹清除）
- SVG 校验：`xmllint --noout <file>.svg` → 无 error
- 章节装置检查：`rg "^## 这一章解决什么问题" <chapter>` → 应有结果（每章开头）

---

## Phase 1: 搭骨架

### Task 1.1: 创建目录结构与软链接

**Files:**
- Create: 目录结构（见下）
- Create: `/Users/xiaokai/pixel-art/源书/书A-gamedesign-ebook` (软链接)
- Create: `/Users/xiaokai/pixel-art/源书/书B-gameart-book` (软链接)

- [ ] **Step 1: 创建所有目录**

```bash
mkdir -p /Users/xiaokai/pixel-art/{第一部-观察,第二部-练手,第三部-风格,第四部-制作,第五部-继续,附录}
mkdir -p /Users/xiaokai/pixel-art/images/{序章,观察,练手,风格,制作,继续,附录}
mkdir -p /Users/xiaokai/pixel-art/源书
```

- [ ] **Step 2: 建软链接到两本源书**

```bash
ln -s /Users/xiaokai/gamedesign/ebook /Users/xiaokai/pixel-art/源书/书A-gamedesign-ebook
ln -s /Users/xiaokai/gameart/book /Users/xiaokai/pixel-art/源书/书B-gameart-book
```

- [ ] **Step 3: 验证软链接可用**

Run: `ls /Users/xiaokai/pixel-art/源书/书A-gamedesign-ebook/ch00-一个程序员为什么要学看.md /Users/xiaokai/pixel-art/源书/书B-gameart-book/01-pixel-art-and-aseprite.md`
Expected: 两个文件路径都列出

### Task 1.2: 创建全部章节空文件 + 合并清单头注

每个空文件开头写一段 HTML 注释头注，标明来源（SPEC 第五节）、文风、要生成的 SVG。这样后续每章执行时，作者一眼看到合并要求。

**Files:** 31 章 + 1 序章 + 7 附录 = 39 个空 .md 文件（路径见 SPEC 第十一节目录结构）

- [ ] **Step 1: 创建序章与全部章节空文件（带头注模板）**

每个文件写入头注，格式（以练手01为例）：
```markdown
<!--
章名: 练手01 线条：从颤抖到控制
来源: A ch06(线条) + B ch02(线条/形状/轮廓)
文风: A声+B改写
合并要点:
  - A: 线四功能(轮廓/引导/姿态/质感) + 线宽↔情绪 + 路径规划(1:1/2:1/1:0) + 曲线平滑序列
  - B: 阶梯原理修锯齿 + 直/对角/曲线规则 + 五种轮廓 + 边缘处理
  - 统一: 把 A 的"路径规划"与 B 的"staircase principle"写成同一套
SVG: 沿用 A ch06 的 4 张(像素斜率/锯齿对比/线条情绪/曲线平滑) + 按需新增 B 的轮廓对比图
练习: L1 线条情绪速写 / L2 无锯齿曲线 / L3 引导线分析
-->

# 练手01 · 线条：从颤抖到控制
```

为 39 个文件分别写头注（来源/文风/合并要点/SVG/练习，取自 SPEC 第五节与第八节）。

- [ ] **Step 2: 验证文件数与编号连续**

Run: `find /Users/xiaokai/pixel-art -name "*.md" -not -path "*/源书/*" | sort | wc -l`
Expected: 41（SPEC.md + PLAN.md + 39 章/附录）

Run: `ls /Users/xiaokai/pixel-art/第二部-练手/`
Expected: 练手01-线条.md ... 练手09-八概念合奏.md 共 9 个

- [ ] **Step 3: Commit**

```bash
cd /Users/xiaokai/pixel-art && git init && git add -A && git commit -m "scaffold: 目录结构、软链接、39章空文件头注"
```

---

## Phase 2: 书 A SVG 审计

### Task 2.1: 审计书 A 全部 100 张 SVG，产出问题清单

**Files:**
- Create: `/Users/xiaokai/pixel-art/AUDIT-SVG.md`

- [ ] **Step 1: 列出全部 SVG 文件**

Run: `find /Users/xiaokai/gamedesign/ebook/images -name "*.svg" | sort > /tmp/svg-list.txt && wc -l /tmp/svg-list.txt`
Expected: ~100 行

- [ ] **Step 2: 逐张 xmllint 校验语法**

Run: `while read f; do r=$(xmllint --noout "$f" 2>&1); if [ -n "$r" ]; then echo "SYNTAX: $f"; echo "$r"; fi; done < /tmp/svg-list.txt`
记录所有语法错误到 AUDIT-SVG.md。

- [ ] **Step 3: 检查必备属性（xmlns/viewBox/width/height）**

Run: `while read f; do for a in xmlns viewBox width height; do if ! rg -q "$a" "$f"; then echo "MISSING-$a: $f"; fi; done; done < /tmp/svg-list.txt`
记录缺失属性。

- [ ] **Step 4: 检查图编号引用一致性**

Run: `rg -o "图 \d+\.\d+" /Users/xiaokai/gamedesign/ebook/*.md | sort -u`
对照 SVG 文件名（`chNN-figMM-*.svg`），找出 markdown 引用的图号与文件名不匹配的条目。记录到 AUDIT-SVG.md。

- [ ] **Step 5: 渲染抽查（浏览器快照若干代表图）**

用 playwright 打开若干 SVG（每章抽 1 张），截图检查渲染是否错位/文字溢出/不显示。记录明显渲染问题到 AUDIT-SVG.md。

- [ ] **Step 6: 写 AUDIT-SVG.md 汇总**

按问题类型分组：语法错误 / 缺失属性 / 编号不匹配 / 渲染问题 / 内容疑似错误。每条标明文件路径与问题描述。无问题的章节标"通过"。

- [ ] **Step 7: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add AUDIT-SVG.md && git commit -m "audit: 书A 100张SVG审计问题清单"
```

---

## Phase 3: 观察部（序章 + 第一部 5 章，A原声为主）

> 本部以书 A 为主体保留原文，做像素重定向 + 编号调整 + SVG 修复（沿用 A 原 SVG，重编号到 `images/观察/`）。

### Task 3.1: 序章 一个程序员为什么要学"观察"

**Files:**
- Write: `/Users/xiaokai/pixel-art/00-序章-一个程序员为什么要学观察.md`（覆盖头注空文件）
- Copy+重编号: `源书/书A.../images/ch00/*.svg` → `images/序章/`

- [ ] **Step 1: 读源 + 写正文**

读 `源书/书A-gamedesign-ebook/ch00-一个程序员为什么要学看.md`。保留：Stardew/Eric Barone 案例、三优势（系统思维/迭代/工具亲和）、"会画画≠会做游戏美术"区分、三阅读路径（12周/24周/按需）、时间胶囊练习。改动：书名/语气从"全门类游戏美术"收窄为"像素艺术"；"看"统一为"观察"（部名）；更新六部结构图为五部。

- [ ] **Step 2: 复制并重编号 SVG**

```bash
cp /Users/xiaokai/gamedesign/ebook/images/ch00/ch00-fig01-book-structure.svg /Users/xiaokai/pixel-art/images/序章/序章-fig01-五部结构.svg
cp /Users/xiaokai/gamedesign/ebook/images/ch00/ch00-fig02-reading-paths.svg /Users/xiaokai/pixel-art/images/序章/序章-fig02-阅读路径.svg
```
按 AUDIT-SVG.md 修复 ch00 的问题（若有）。改图内标题文字为五部结构。`xmllint --noout` 校验。

- [ ] **Step 3: 验证**

Run: `rg "这一章解决什么问题" /Users/xiaokai/pixel-art/00-序章*.md` → 有
Run: `rg "六部|选兵器" /Users/xiaokai/pixel-art/00-序章*.md` → 应已改为五部（"选兵器"不再作为独立部）
Run: `wc -l /Users/xiaokai/pixel-art/00-序章*.md` → 80-150 行
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/序章/*.svg` → 无 error
Run: `rg "extracted/" /Users/xiaokai/pixel-art/00-序章*.md` → 无

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "序章: 一个程序员为什么要学观察"
```

### Task 3.2: 观察01 三能力模型与自评

**Files:**
- Write: `第一部-观察/观察01-三能力模型与自评.md`
- SVG: `images/观察/观察01-fig01-三能力模型.svg`、`观察01-fig02-能力自评雷达.svg`、`观察01-fig03-分数路由.svg`

- [ ] **Step 1: 读源 + 改写**

读 `源书/书A.../ch01-你的美术能力不是零.md`。核心改动：五子能力模型（感知/词汇/执行/工具/整合）改写为**三能力模型（看/做/整）**——看=感知+词汇，做=执行+工具（像素场景工具固定=Aseprite+Godot，故"选兵器"坍缩），整=整合（对应制作后半：上引擎/管线/一致性）。保留 10 题自评（题目调整为三能力维度）、雷达图、分数→起点章路由表（按新编号重写：0-15→观察02；…；41-50→风格03 之类）。

- [ ] **Step 2: 重画三张 SVG**

基于 A 原 ch01 三张 SVG 改：①五层协议栈→三能力栈图；②雷达图改三轴；③分数路由表按新章号。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "五子能力|五层协议" /Users/xiaokai/pixel-art/第一部-观察/观察01*.md` → 应无（已改三能力）
Run: `rg "三能力|看.*做.*整" /Users/xiaokai/pixel-art/第一部-观察/观察01*.md` → 有
Run: `rg "观察02|练手01|风格03" /Users/xiaokai/pixel-art/第一部-观察/观察01*.md` → 路由表用新编号
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/观察/观察01-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "观察01: 三能力模型与自评"
```

### Task 3.3: 观察02 八概念速览

**Files:**
- Write: `第一部-观察/观察02-八概念速览.md`
- SVG: 沿用 A ch02 的 4 张，例子换像素游戏

- [ ] **Step 1: 读源 + 改写**

读 `源书/书A.../ch02-视觉编程语言八概念速览.md`。保留：八概念框架（线/形状/空间/明度/色彩/质感/构图/风格）、"八种数据类型"类比、视觉权重/对比哲学、便利店测试练习。改动：每概念的游戏例子对换为像素游戏（Celeste/Dead Cells/Stardew/Hollow Knight/Downwell 等）。

- [ ] **Step 2: 复制重编号 SVG + 按审计修复**

复制 A ch02 四张 SVG 到 `images/观察/观察02-figNN-*.svg`，按 AUDIT-SVG 修复，`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "八概念|视觉权重" /Users/xiaokai/pixel-art/第一部-观察/观察02*.md` → 有
Run: `rg "Overwatch|League|riot" -i /Users/xiaokai/pixel-art/第一部-观察/观察02*.md` → 非像素例子应已替换或仅作通用引用
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/观察/观察02-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "观察02: 八概念速览"
```

### Task 3.4: 观察03 VTS侦探法

**Files:**
- Write: `第一部-观察/观察03-VTS侦探法.md`
- SVG: 沿用 A ch03 的 2 张

- [ ] **Step 1: 读源 + 改写**

读 `源书/书A.../ch03-VTS侦探法.md`。保留：VTS 三问、debug↔VTS 类比、每日 5min 模板。改动：worked example 从 Hollow Knight（非像素）换成像素游戏（如 Dead Cells 的牢房场景或 Celese 的第一章），三轮分析重写。

- [ ] **Step 2: 复制重编号 SVG + 修复**

- [ ] **Step 3: 验证**

Run: `rg "VTS|三个问题|侦探" /Users/xiaokai/pixel-art/第一部-观察/观察03*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/观察/观察03-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "观察03: VTS侦探法"
```

### Task 3.5: 观察04 喂养眼睛

**Files:**
- Write: `第一部-观察/观察04-喂养眼睛.md`
- SVG: 沿用 A ch04 的 2 张

- [ ] **Step 1: 读源 + 改写**

读 `源书/书A.../ch04-喂养眼睛.md`。保留：视觉节食 vs 暴食、四输入源、参考图归档系统（按功能不按来源）、Pinterest/Are.na/PureRef 用法、借鉴不抄袭。改动：四输入源的例子加像素向（Lospec、pixel art joint、@pixelpedro 等）；folder tree 示例换像素分类。

- [ ] **Step 2: 复制重编号 SVG + 修复**

- [ ] **Step 3: 验证**

Run: `rg "视觉节食|参考|PureRef" /Users/xiaokai/pixel-art/第一部-观察/观察04*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/观察/观察04-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "观察04: 喂养眼睛"
```

### Task 3.6: 观察05 AI美术私教

**Files:**
- Write: `第一部-观察/观察05-AI美术私教.md`
- SVG: 沿用 A ch05 的 2 张

- [ ] **Step 1: 读源 + 改写**

读 `源书/书A.../ch05-AI美术私教.md`。保留：AI 作反馈不作生成、能力边界、结构化 prompt 模板（8 维表 + TOP3）、四场景、四反馈渠道对比。改动：prompt 模板的填充例子换成像素作品分析。

- [ ] **Step 2: 复制重编号 SVG + 修复**

- [ ] **Step 3: 验证**

Run: `rg "AI|prompt|反馈" /Users/xiaokai/pixel-art/第一部-观察/观察05*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/观察/观察05-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "观察05: AI美术私教"
```

### Task 3.7: 观察部整体验证

- [ ] **Step 1: 部内交叉引用检查**

Run: `rg "观察0[1-5]" /Users/xiaokai/pixel-art/第一部-观察/*.md` → 引用都在 01-05 范围
Run: `rg "ch0[1-5]|ch01\b" /Users/xiaokai/pixel-art/第一部-观察/*.md` → 无旧编号残留

- [ ] **Step 2: 文风一致性快查**

Run: `rg "章节来源|B[123]_|读者朋友|本章小结" /Users/xiaokai/pixel-art/第一部-观察/*.md` → 无书 B 痕迹
Run: `rg "你打开|程序员|类比" /Users/xiaokai/pixel-art/第一部-观察/*.md` → 有书 A 声调

- [ ] **Step 3: SVG 全检**

Run: `for f in /Users/xiaokai/pixel-art/images/观察/*.svg /Users/xiaokai/pixel-art/images/序章/*.svg; do xmllint --noout "$f" || echo "FAIL: $f"; done` → 全部无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "观察部: 整体验证通过"
```

---

## Phase 4: 练手部（第二部 9 章，A+B 合并改写重灾区 + ~70 张 SVG）

> 本部是工作量最大的部分。每章 = 书 A 框架 + 书 B 融入改写。SVG 沿用 A 原（修复）+ 为 B 内容新生成。保留贯穿式角色练习（练手01→练手05→制作02 一个角色项目串起来）。

### Task 4.1: 练手01 线条

**Files:**
- Write: `第二部-练手/练手01-线条.md`
- SVG: 沿用 A ch06 4 张 + 新增 B ch02 轮廓对比图

- [ ] **Step 1: 读两个源 + 合并改写**

读 `源书/书A.../ch06-线条.md`（304行）与 `源书/书B.../02-lines-shapes-and-outlines.md`（157行）。
A 框架：线四功能（轮廓/引导/姿态/质感）、线宽↔情绪、像素线=路径规划（1:1/2:1/1:0 斜率）、jaggies、曲线平滑序列 `[5,4,3,2,2,1,1,1,2,3]`、Linter 类比、L1/L2/L3。
B 融入：阶梯原理（=A 的路径规划，统一为一套表述）、直/对角/曲线规则、**五种轮廓**（无轮廓/黑色内描/黑色外轮廓/彩色轮廓/着色轮廓）及对比表、边缘处理、形状组合。
改写为 A 声调：为 B 的五轮廓补编程类比（如"轮廓=类型系统的边界检查，可选 strict/none"）。

- [ ] **Step 2: SVG**

沿用 A ch06 四张（像素斜率/锯齿对比/线条情绪/曲线平滑），按 AUDIT 修复，重编号 `练手01-figNN`。新生成 1 张"五种轮廓对比"SVG（用 `<rect>` 像素网格画同一角色五种轮廓）。`xmllint` 校验。

- [ ] **Step 3: 贯穿练习衔接**

在练习部分明确：本章 L1 起开始一个角色项目（画轮廓/线稿），将在练手05 上色、制作02 做成完整角色。

- [ ] **Step 4: 验证**

Run: `rg "阶梯原理|五种轮廓|路径规划" /Users/xiaokai/pixel-art/第二部-练手/练手01*.md` → 都有
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第二部-练手/练手01*.md` → 无
Run: `rg "贯穿|角色项目|练手05|制作02" /Users/xiaokai/pixel-art/第二部-练手/练手01*.md` → 衔接说明在
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手01-*.svg` → 无 error

- [ ] **Step 5: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手01: 线条 (A ch06 + B ch02 合并)"
```

### Task 4.2: 练手02 形状与体积

**Files:**
- Write: `第二部-练手/练手02-形状与体积.md`
- SVG: 沿用 A ch07 4 张 + 新增 B ch4.5 几何体光照（5张）+ B ch06 可读性（3张）

- [ ] **Step 1: 读三个源 + 合并改写**

读 `源书/书A.../ch07-形状与体积.md`、`源书/书B.../04-5-form-and-volume.md`、`源书/书B.../06-readability-and-sprite-design.md`。
A 框架：剪影=单元测试、形状心理学（圆=安全/方=稳定/三角=危险）、剪影→三视图四步、三体积法、TypeScript interface 类比、Drawabox。
B ch4.5 融入：**五几何体**（球/方/柱/锥/环）光照模式、**材质区分**（金属/布/皮肤/水玻璃/木）、AO、反射光、物体拆解表。
B ch06 融入：可读性为 #1 优先、尺寸策略、象征画法（手/眼/脸四步）、比例（chibi vs 写实）、剪影设计、可读性检查（100% 缩放/模糊）。
改写为 A 声调：为"可读性"补类比（"可读性=API 的最小惊讶原则，小尺寸下信息密度要克制"）。

- [ ] **Step 2: SVG**

沿用 A ch07 四张（形状心理/剪影测试/剪影→三视图/三体积法），修复重编号。新生成：五几何体光照（5张 `<rect>` 网格）、材质对比（1张）、可读性检查前后对比（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "剪影|单元测试|五几何体|可读性|象征画法" /Users/xiaokai/pixel-art/第二部-练手/练手02*.md` → 都有
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第二部-练手/练手02*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手02-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手02: 形状与体积 (A ch07 + B ch4.5 + B ch06)"
```

### Task 4.3: 练手03 空间

**Files:**
- Write: `第二部-练手/练手03-空间.md`
- SVG: 沿用 A ch08 4 张

- [ ] **Step 1: 读源 + 改写（A原声，B 无对应）**

读 `源书/书A.../ch08-空间.md`。保留：CSS box model 类比、正负空间、五比例实验（50:50~90:10）、五层 2D 景深、呼吸点（≥10-15%）、A Short Hike 案例。改动：例子换像素游戏（如 Downwell 的简洁负空间、Celeste 的景深层）。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "CSS box model|负空间|呼吸点|五层" /Users/xiaokai/pixel-art/第二部-练手/练手03*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手03-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手03: 空间"
```

### Task 4.4: 练手04 明度

**Files:**
- Write: `第二部-练手/练手04-明度.md`
- SVG: 沿用 A ch09 5 张 + 新增 B ch3.5 九级明度阶/灰度→色彩

- [ ] **Step 1: 读两个源 + 合并改写**

读 `源书/书A.../ch09-明度.md` 与 `源书/书B.../03-5-value-structure.md`。
A 框架：明度=骨架（色是皮）、去色测试（Ctrl+Shift+U）、三区（高/中/低调）、四模板（高调/低调/高对比/低对比 + 面积比）、灰度缩略图（30s/2min/5min）、三点光照、游戏明度分层、程序员优势（明度=整数）。
B ch3.5 融入：**九级明度阶**、**灰度草图→上色工作流**、明度引导视线、明度与情绪（高长调/低长调等）、常见明度错误。
改写为 A 声调：为"灰度→上色工作流"补类比（"先写 HTML 结构再加 CSS 样式"）。

- [ ] **Step 2: SVG**

沿用 A ch09 五张，修复重编号。新生成：九级明度阶（1张 `<rect>` 灰阶）、灰度→色彩工作流（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "骨架|去色|九级|灰度.*色彩|三点光照" /Users/xiaokai/pixel-art/第二部-练手/练手04*.md` → 有
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第二部-练手/练手04*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手04-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手04: 明度 (A ch09 + B ch3.5)"
```

### Task 4.5: 练手05 色彩

**Files:**
- Write: `第二部-练手/练手05-色彩.md`
- SVG: 沿用 A ch10 7 张 + 新增 B ch03 色相偏移/6和谐方案/调色板管理

- [ ] **Step 1: 读两个源 + 合并改写**

读 `源书/书A.../ch10-色彩.md` 与 `源书/书B.../03-color-theory-and-palettes.md`。
A 框架：功能金字塔（交互>信息>情绪>装饰）、有限调色板、HSV、色域遮罩（Gurney）、Albers 相对性、90min 色彩脚本、偏色阴影、色盲三规则。
B ch03 融入：HSL/RGB/hex、**东西方着色两流派**（西方逐级 vs 东方 hue-shifting）、**色彩斜坡/ramp**、**6 和谐方案**（单色/类比/互补/分裂互补/三角/四角）+ 60-30-10 规则、调色板管理、经典调色板（PICO-8/DB16/Endesga32/Sweetie16/ARQ4）、调色板替换。
改写为 A 声调：为 hue-shifting 补类比（"色相偏移=给 ramp 加渐变插值函数，不是线性而是 HSL 空间曲线"）。

- [ ] **Step 2: SVG**

沿用 A ch10 七张，修复重编号。新生成：东西方着色对比（1张）、6 和谐方案速查（1张）、经典调色板陈列（1张）。`xmllint` 校验。

- [ ] **Step 3: 贯穿练习衔接**

明确：本章给练手01 的角色项目上色，将在制作02 完整化。

- [ ] **Step 4: 验证**

Run: `rg "功能金字塔|HSV|hue.?shift|色相偏移|60.?30.?10|PICO" /Users/xiaokai/pixel-art/第二部-练手/练手05*.md` → 有
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第二部-练手/练手05*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手05-*.svg` → 无 error

- [ ] **Step 5: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手05: 色彩 (A ch10 + B ch03)"
```

### Task 4.6: 练手06 质感

**Files:**
- Write: `第二部-练手/练手06-质感.md`
- SVG: 沿用 A ch11 3 张 + 新增 B ch07 抖动模式

- [ ] **Step 1: 读两个源 + 合并改写**

读 `源书/书A.../ch11-质感.md` 与 `源书/书B.../07-dithering.md`。
A 框架：触觉维度（镜像神经元）、Tile=2D 数组周期复制（无缝=周期边界条件）、抖动=离散采样模拟连续、四步无缝、四抖动模式、CSS background-repeat 类比。
B ch07 融入：抖动原理与历史、何时用/避免、**棋盘格及变体**（平行线/间断/凹痕/交织/随机）、**风格化抖动**、抖动与色彩混合、抖动笔刷、复古硬件意义（Genesis/SNES）。
改写为 A 声调：把 A 的"四模式"与 B 的"棋盘格变体"合并为一套完整抖动体系；为风格化抖动补类比（"风格化抖动=自定义采样核，不是固定 Bayer 矩阵"）。

- [ ] **Step 2: SVG**

沿用 A ch11 三张，修复重编号。新生成：抖动模式全集对比（1张 `<rect>` 网格画各种抖动纹理）、Yoshi's Island 抖动风格示意（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "抖动|棋盘格|无缝|Tile.*数组" /Users/xiaokai/pixel-art/第二部-练手/练手06*.md` → 有
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第二部-练手/练手06*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手06-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手06: 质感 (A ch11 + B ch07 抖动)"
```

### Task 4.7: 练手07 构图

**Files:**
- Write: `第二部-练手/练手07-构图.md`
- SVG: 沿用 A ch12 5 张 + 新增 B ch6.5 动作线

- [ ] **Step 1: 读两个源 + 合并改写**

读 `源书/书A.../ch12-构图.md` 与 `源书/书B.../06-5-composition-and-gesture.md`。
A 框架：布局引擎、视觉权重五规则（亮度/饱和度/尺寸/人脸/复杂度）、引导线=超链接、三分法=CSS grid、取景=overflow:hidden、三鲁棒性（移动焦点/UI遮挡/分辨率）、三游戏类型构图。
B ch6.5 融入：古典规则（三分/对角/取景/引导线/黄金比）、负空间、视觉权重与平衡、**三引导工具**（明度/饱和度/细节密度）、**动作线**（C/S/Z/竖/斜）、场景分析、构图清单。
改写为 A 声调：为动作线补类比（"动作线=角色姿态的主干向量，像函数的主返回值，细节是副作用"）。

- [ ] **Step 2: SVG**

沿用 A ch12 五张，修复重编号。新生成：动作线形状指南（1张，画 C/S/Z 线）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "布局引擎|视觉权重|动作线|三分法" /Users/xiaokai/pixel-art/第二部-练手/练手07*.md` → 有
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第二部-练手/练手07*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手07-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手07: 构图 (A ch12 + B ch6.5)"
```

### Task 4.8: 练手08 像素专属技法（新章，A 无对应）

**Files:**
- Write: `第二部-练手/练手08-像素专属技法.md`
- SVG: 全新生成（抗锯齿/Banding/透视）

- [ ] **Step 1: 读两个源 + 全面改写为 A 声**

读 `源书/书B.../05-anti-aliasing-and-banding.md` 与 `源书/书B.../08-game-perspectives.md`。A 无对应，全部改写。
内容：①抗锯齿（何时用/方法/线粗控制/过度危害）；②Banding（平行像素行的危害/识别/三种修复）；③游戏透视（正交类型/侧视/俯视/2:1 等距/塞尔达透视问题/45° dimetric/斜视/选择）。
补编程类比：AA="亚像素插值，像 CSS subpixel rendering"；Banding="规律性冗余行=代码里的重复模式，lint 该警告"；等距 2:1="等距网格是固定的坐标系变换矩阵"。

- [ ] **Step 2: SVG 全新生成**

生成：AA 原则（1张）、AA 过度对比（1张）、Banding 示例与修复（2张）、透视类型对比（1张）、2:1 等距网格（1张）、塞尔达透视问题（1张）。用 `<rect>` 像素网格。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "抗锯齿|Banding|等距|塞尔达|dimetric" /Users/xiaokai/pixel-art/第二部-练手/练手08*.md` → 有
Run: `rg "章节来源|extracted/" /Users/xiaokai/pixel-art/第二部-练手/练手08*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手08-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手08: 像素专属技法 (B ch05 + B ch08 全面改写)"
```

### Task 4.9: 练手09 八概念合奏

**Files:**
- Write: `第二部-练手/练手09-八概念合奏.md`
- SVG: 沿用 A ch14 2 张（管线预览图更新为五部）

- [ ] **Step 1: 读源 + 改写**

读 `源书/书A.../ch14-综合练习.md`。保留：全维度分析模板、优先级诊断流（明度→构图→色→质感，映射 V1→V2→V4→IT）、逆向工程工作流（情绪→视觉参数）。改动：例子换像素；管线预览图更新为五部结构；标记"第二部结束"。

- [ ] **Step 2: SVG 沿用修复 + 管线图重画**

- [ ] **Step 3: 验证**

Run: `rg "合奏|优先级|逆向工程|第二部结束" /Users/xiaokai/pixel-art/第二部-练手/练手09*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/练手/练手09-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手09: 八概念合奏 (第二部完结)"
```

### Task 4.10: 练手部整体验证

- [ ] **Step 1: 章数与编号**

Run: `ls /Users/xiaokai/pixel-art/第二部-练手/ | wc -l` → 9
Run: `ls /Users/xiaokai/pixel-art/第二部-练手/` → 练手01~09 连续

- [ ] **Step 2: 贯穿角色练习检查**

Run: `rg "角色项目|贯穿|练手01.*角色" /Users/xiaokai/pixel-art/第二部-练手/*.md` → 练手01/05 有衔接说明

- [ ] **Step 3: 文风与路径全检**

Run: `rg "extracted/|章节来源|B[123]_" /Users/xiaokai/pixel-art/第二部-练手/*.md` → 无
Run: `for f in /Users/xiaokai/pixel-art/images/练手/*.svg; do xmllint --noout "$f" || echo "FAIL: $f"; done` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "练手部: 整体验证通过 (9章+SVG)"
```

---

## Phase 5: 风格部（第三部 4 章）

### Task 5.1: 风格01 像素风格全景与四代

**Files:**
- Write: `第三部-风格/风格01-像素风格全景与四代.md`
- SVG: 沿用 A ch21 2 张 + A ch19 像素部分图

- [ ] **Step 1: 读源 + 合并**

读 `源书/书A.../ch21-像素艺术风格深度指南.md` 与 `ch19-2D游戏美术风格全景.md`（只取像素部分）。内容：何时**不**选像素、四代（NES 8bit/SNES 16bit/现代高 bit/调色板艺术）、像素风格定义/特征/代表游戏/适用场景/隐藏成本、成本矩阵、练习路线（Shifty 30 天/F.I.G.H.T.）。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "四代|NES|SNES|不选像素|FIGHT" /Users/xiaokai/pixel-art/第三部-风格/风格01*.md` → 有
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第三部-风格/风格01*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/风格/风格01-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "风格01: 像素风格全景与四代"
```

### Task 5.2: 风格02 分辨率与调色板决策

**Files:**
- Write: `第三部-风格/风格02-分辨率与调色板决策.md`
- SVG: 沿用 A ch21 分辨率图 + 新增调色板决策图

- [ ] **Step 1: 读源 + 合并**

读 `源书/书A.../ch21`（分辨率部分）与 `源书/书B.../03-color-theory-and-palettes.md`（调色板哲学部分）。内容：分辨率决策（16/32/64 + 能力边界 + 工时成本）、调色板选择策略（色数/情绪/平台限制/与练手05 衔接）。

- [ ] **Step 2: SVG**

沿用 A ch21 分辨率图修复重编号。新生成调色板决策流程图（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "16.*32.*64|分辨率|调色板.*决策|工时" /Users/xiaokai/pixel-art/第三部-风格/风格02*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/风格/风格02-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "风格02: 分辨率与调色板决策"
```

### Task 5.3: 风格03 风格选择与混血

**Files:**
- Write: `第三部-风格/风格03-风格选择与混血.md`
- SVG: 沿用 A ch23 2 张

- [ ] **Step 1: 读源 + 改写（A原声，像素向）**

读 `源书/书A.../ch23-风格选择与混血.md`。保留：约束清单（时间/技能/平台/受众/资产量）、逆向决策（排除绝对不要的）、风格 MVP 验证（3 概念图+1 剪影+1 调色板→陌生人反馈）、混血三原则（70-20-10/过渡区无硬切/桥接层+统一锚点）。改动：例子像素向；非像素混血精简。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "约束清单|MVP|混血|70.?20.?10" /Users/xiaokai/pixel-art/第三部-风格/风格03*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/风格/风格03-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "风格03: 风格选择与混血"
```

### Task 5.4: 风格04 视觉风格文档

**Files:**
- Write: `第三部-风格/风格04-视觉风格文档.md`
- SVG: 沿用 A ch24 2 张

- [ ] **Step 1: 读源 + 改写**

读 `源书/书A.../ch24-视觉风格文档.md`。保留：决策日志+约束集+对比标准、三模板（GameJam 1 页/小项目 1 页 Notion/完整 wiki）、Gold Standard 四铁律、.editorconfig 类比。改动：模板像素向；标记"第三部结束"。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "风格文档|Gold Standard|宪法|第三部结束" /Users/xiaokai/pixel-art/第三部-风格/风格04*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/风格/风格04-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "风格04: 视觉风格文档 (第三部完结)"
```

### Task 5.5: 风格部整体验证

- [ ] **Step 1: 检查**

Run: `ls /Users/xiaokai/pixel-art/第三部-风格/ | wc -l` → 4
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第三部-风格/*.md` → 无
Run: `for f in /Users/xiaokai/pixel-art/images/风格/*.svg; do xmllint --noout "$f" || echo "FAIL: $f"; done` → 无 error

- [ ] **Step 2: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "风格部: 整体验证通过"
```

---

## Phase 6: 制作部（第四部 9 章，A+B 合并 + B 全面改写 + ~40 张 SVG）

### Task 6.1: 制作01 你的工具：Aseprite 与工具逻辑

**Files:**
- Write: `第四部-制作/制作01-你的工具Aseprite.md`
- SVG: 新生成 Aseprite 界面示意图 + 沿用 A ch15 2 张精简

- [ ] **Step 1: 读源 + 合并改写（B 主体 + A 精简）**

读 `源书/书B.../01-pixel-art-and-aseprite.md`（Aseprite 安装/界面/工具/图层/选区/镜像/色彩）与 `源书/书A.../ch15-工具选择的底层逻辑.md`（精简为"为什么 Aseprite+Godot"论证：三原则但工具已定）。全面改写为 A 声：为每个 Aseprite 工具补类比（铅笔=单像素 setter、墨线=直线算法、填色=flood fill BFS）。

- [ ] **Step 2: SVG**

新生成 Aseprite 界面简化示意图（1-2张，SVG 画 UI 布局）。沿用 A ch15 三原则图精简。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "Aseprite|图层|选区|快捷键" /Users/xiaokai/pixel-art/第四部-制作/制作01*.md` → 有
Run: `rg "章节来源|extracted/" /Users/xiaokai/pixel-art/第四部-制作/制作01*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作01-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作01: 你的工具 Aseprite (B ch01 + A ch15)"
```

### Task 6.2: 制作02 像素角色工作流

**Files:**
- Write: `第四部-制作/制作02-像素角色工作流.md`
- SVG: 沿用 A ch16 3 张 + A ch25 3 张 + 新增 B ch09 转换法

- [ ] **Step 1: 读三个源 + 合并改写**

读 `源书/书A.../ch16-像素艺术工作流.md`、`ch25-角色设计.md`、`源书/书B.../09-from-concept-to-character.md`。
A ch16：10 步角色管线（新建→锁调色板→剪影→结构→细节→上色→阴影→变体→导出 PNG→引擎验证）、Tileset、帧时序。
A ch25：四层法（剪影→内部结构→细节→色彩）、最小表达、角色规格表 8 维、三层分化、表情。
B ch09 融入：草图→像素转换（最近邻缩放/图层描摹）、参考（含 Blender 白模）、水平翻转平衡、边缘细化、角色立绘、昼夜调色板替换。
改写为 A 声：合并为完整角色章，贯穿练习在此完成（练手01/05 的角色→完整管线）。

- [ ] **Step 2: SVG**

沿用 A ch16/ch25 六张，修复重编号。新生成：草图→像素两转换法（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "10步|四层|剪影|最近邻|昼夜|规格表" /Users/xiaokai/pixel-art/第四部-制作/制作02*.md` → 有
Run: `rg "贯穿|练手01|角色项目" /Users/xiaokai/pixel-art/第四部-制作/制作02*.md` → 贯穿练习收尾说明
Run: `rg "章节来源|extracted/" /Users/xiaokai/pixel-art/第四部-制作/制作02*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作02-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作02: 像素角色工作流 (A ch16+25 + B ch09, 贯穿练习收尾)"
```

### Task 6.3: 制作03 环境与 Tile

**Files:**
- Write: `第四部-制作/制作03-环境与Tile.md`
- SVG: 沿用 A ch26 3 张 + 新增 B ch10 9-Slice/3-Tile

- [ ] **Step 1: 读两个源 + 合并改写**

读 `源书/书A.../ch26-环境与场景.md` 与 `源书/书B.../10-tiled-backgrounds.md`。
A ch26：三层叙事（前景=谁的眼/中景=发生什么/背景=暗示什么）、Tileset 模块化三规则、三点光照、关卡视觉节奏。
B ch10 融入：Tile 概念与优势、尺寸（2 的幂）、无缝纹理 do/don't、**9-Slice 与 3-Tile 转换**、Tile 库、场景组装、调色板替换换季换时。
改写为 A 声：为 9-Slice 补类比（"9-Slice=CSS border-image，九宫格切片伸缩"）。

- [ ] **Step 2: SVG**

沿用 A ch26 三张，修复重编号。新生成：9-Slice vs 3-Tile 对比（1张）、无缝纹理问题（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "三层|Tileset|9.?Slice|3.?Tile|三点光照|无缝" /Users/xiaokai/pixel-art/第四部-制作/制作03*.md` → 有
Run: `rg "章节来源|extracted/" /Users/xiaokai/pixel-art/第四部-制作/制作03*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作03-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作03: 环境与 Tile (A ch26 + B ch10)"
```

### Task 6.4: 制作04 UI 设计

**Files:**
- Write: `第四部-制作/制作04-UI设计.md`
- SVG: 沿用 A ch27 3 张

- [ ] **Step 1: 读源 + 改写（A原声，B 无对应）**

读 `源书/书A.../ch27-UI设计.md`。保留：三功能（信息/操作/情绪）、四类型（非叙事/叙事/空间/元）、80-20 注意力、8 步工作流、四风格、色盲四原则、Dead Space 叙事 UI 案例。改动：例子像素 UI。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "80.?20|四类型|叙事 UI|色盲|HUD" /Users/xiaokai/pixel-art/第四部-制作/制作04*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作04-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作04: UI 设计"
```

### Task 6.5: 制作05 动画与特效

**Files:**
- Write: `第四部-制作/制作05-动画与特效.md`
- SVG: 沿用 A ch28 3 张 + 新增 B ch11/ch12

- [ ] **Step 1: 读三个源 + 合并改写**

读 `源书/书A.../ch28-动画入门.md`、`源书/书B.../11-animation-principles.md`、`12-special-effects.md`。
A ch28：五原则（挤压拉伸/ anticipation/慢入慢出/跟随/夸张）、帧矩阵（9 动作）、三降级、骨骼 vs 逐帧、四步工作流。
B ch11 融入：Aseprite 动画基础（帧/洋葱皮/FPS）、Disney 12 原则（取最相关 6）、核心概念、逐帧问题（像素闪烁/最小移动）、游戏动画状态、走路循环。
B ch12 融入：特效三原则（可读/一致/性能）、火/激光/发光/爆炸/破坏/物理、特效复用（换色/缩放/旋转）。
改写为 A 声：为帧时序补类比（"帧时长=定时器 interval，不等长 interval 制造节奏"）。

- [ ] **Step 2: SVG**

沿用 A ch28 三张，修复重编号。新生成：走路循环帧序列（1张）、特效配方（火/爆炸，1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "五原则|帧矩阵|洋葱皮|走路循环|爆炸|特效复用" /Users/xiaokai/pixel-art/第四部-制作/制作05*.md` → 有
Run: `rg "章节来源|extracted/" /Users/xiaokai/pixel-art/第四部-制作/制作05*.md` → 无
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作05-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作05: 动画与特效 (A ch28 + B ch11 + B ch12)"
```

### Task 6.6: 制作06 非手绘管线

**Files:**
- Write: `第四部-制作/制作06-非手绘管线.md`
- SVG: 全新生成（Blender 节点图等）

- [ ] **Step 1: 读源 + 全面改写（B 主体，A 无对应）**

读 `源书/书B.../14-non-handdrawn-methods.md`（553 行，B 最长章）。内容：为何读这章、方法概览、Blender 3D→像素管线（核心）、Blender 工具推荐、AI 辅助生成、程序化/批量生成、引擎实时渲染、Dead Cells 案例、方法对比与选择。
全面改写为 A 声：为每个管线补类比（3D→像素="编译器：高维源码编译到像素目标码"；AI="代码生成器，能起骨架不能负责正确性"；程序化="脚本批量生成，确定性可复现"）。把 B 的 ASCII 节点图改写为 SVG。

- [ ] **Step 2: SVG 全新生成**

生成：方法对比矩阵（1张）、Blender 3D→像素管线流程（1张）、Dead Cells 工作流（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "3D.*像素|Dead Cells|AI.*生成|程序化|实时渲染" /Users/xiaokai/pixel-art/第四部-制作/制作06*.md` → 有
Run: `rg "章节来源|extracted/|ASCII" /Users/xiaokai/pixel-art/第四部-制作/制作06*.md` → 无（ASCII 节点图已转 SVG）
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作06-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作06: 非手绘管线 (B ch14 全面改写)"
```

### Task 6.7: 制作07 上引擎

**Files:**
- Write: `第四部-制作/制作07-上引擎.md`
- SVG: 沿用 A ch29 2 张

- [ ] **Step 1: 读源 + 改写（A原声，Godot 为主）**

读 `源书/书A.../ch29-上引擎.md`。保留：Bilinear vs Nearest Neighbor 数学、PNG 导出清单、五首检（滤镜/整数缩放/锚点统一/alpha halo/帧序）、画布 vs 引擎叠加对比、"引擎是翻译不是显示"、"源画布修不在引擎修"。改动：三引擎对比精简为 Godot 为主（Unity/GameMaker 简提）。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "Nearest Neighbor|Bilinear|整数缩放|Godot|alpha" /Users/xiaokai/pixel-art/第四部-制作/制作07*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作07-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作07: 上引擎"
```

### Task 6.8: 制作08 生产管线

**Files:**
- Write: `第四部-制作/制作08-生产管线.md`
- SVG: 沿用 A ch18 2 张

- [ ] **Step 1: 读源 + 改写（A原声）**

读 `源书/书A.../ch18-建立你的生产管线.md`。保留：六阶段资产生命周期（参考→缩略→blockout→精修→导出→引擎验证）+时间分配、命名规范公式、迭代文件夹版本控制（v1/v2/v3/current）、瓶颈诊断四步、"final 版地狱"反模式。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "六阶段|命名规范|迭代文件夹|瓶颈|v1.*v2.*current" /Users/xiaokai/pixel-art/第四部-制作/制作08*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作08-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作08: 生产管线"
```

### Task 6.9: 制作09 一致性审计

**Files:**
- Write: `第四部-制作/制作09-一致性审计.md`
- SVG: 沿用 A ch30 2 张

- [ ] **Step 1: 读源 + 改写（A原声）**

读 `源书/书A.../ch30-一致性审计.md`。保留：10 项一致性清单（线宽/调色板/细节密度/抗锯齿/UI 融合/帧率/光源/透视/比例/字体）、5 常见视觉 bug（症状/诊断/修复）、Braid 案例、陌生人 3 秒测试、质量门、"一致性是工程，美是彩票"。标记"第四部结束"。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "10项|一致性|陌生人|3秒|第四部结束" /Users/xiaokai/pixel-art/第四部-制作/制作09*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/制作/制作09-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作09: 一致性审计 (第四部完结)"
```

### Task 6.10: 制作部整体验证

- [ ] **Step 1: 检查**

Run: `ls /Users/xiaokai/pixel-art/第四部-制作/ | wc -l` → 9
Run: `rg "extracted/|章节来源|B[123]_" /Users/xiaokai/pixel-art/第四部-制作/*.md` → 无
Run: `for f in /Users/xiaokai/pixel-art/images/制作/*.svg; do xmllint --noout "$f" || echo "FAIL: $f"; done` → 无 error

- [ ] **Step 2: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "制作部: 整体验证通过 (9章+SVG)"
```

---

## Phase 7: 继续部（第五部 4 章，含新写继续04）

### Task 7.1: 继续01 GameJam

**Files:**
- Write: `第五部-继续/继续01-GameJam.md`
- SVG: 沿用 A ch31 2 张

- [ ] **Step 1: 读源 + 改写（A原声，像素向）**

读 `源书/书A.../ch31-GameJam.md`。保留：48h 策略（"看起来完成"非"看起来美"）、5 低成本风格按 ROI 排、48h 时间线、三死法、四维复盘模板、奶奶测试。改动：5 风格例子像素向。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "48|GameJam|低成本|复盘|奶奶" /Users/xiaokai/pixel-art/第五部-继续/继续01*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/继续/继续01-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "继续01: GameJam"
```

### Task 7.2: 继续02 持续输出的习惯系统

**Files:**
- Write: `第五部-继续/继续02-持续输出的习惯系统.md`
- SVG: 沿用 A ch32 2 张

- [ ] **Step 1: 读源 + 改写（A原声）**

读 `源书/书A.../ch32-持续输出的习惯系统.md`。保留：热情-崩溃三阶段、F.I.G.H.T. 框架、最小承诺（5min/天+1件/周+1帖/月）、Gap 期（Ira Glass）、月度同题重画。

- [ ] **Step 2: SVG 沿用修复重编号**

- [ ] **Step 3: 验证**

Run: `rg "FIGHT|最小承诺|Gap|月度重画" /Users/xiaokai/pixel-art/第五部-继续/继续02*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/继续/继续02-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "继续02: 持续输出的习惯系统"
```

### Task 7.3: 继续03 一个人的艺术指导

**Files:**
- Write: `第五部-继续/继续03-一个人的艺术指导.md`
- SVG: 沿用 A ch33 3 张

- [ ] **Step 1: 读源 + 改写（A原声）**

读 `源书/书A.../ch33-一个人的艺术指导.md`。保留：三人格治理（艺术家/程序员/PM）、时间技能工具决策三角、外包/买资产/合作、约束藏瑕（JSLegendDev）、11 步独自工作流总览、结语"Go make something"。11 步总览按新结构重画。

- [ ] **Step 2: SVG 沿用修复重编号 + 11步总览重画**

- [ ] **Step 3: 验证**

Run: `rg "三人格|决策三角|约束藏瑕|11步|Go make" /Users/xiaokai/pixel-art/第五部-继续/继续03*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/继续/继续03-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "继续03: 一个人的艺术指导"
```

### Task 7.4: 继续04 像素之后（新写）

**Files:**
- Write: `第五部-继续/继续04-像素之后.md`
- SVG: 全新生成 1-2 张

- [ ] **Step 1: 读源精简 + 新写**

读 `源书/书A.../ch17-数字绘画与3D入门工作流.md`、`ch20-3D游戏美术风格全景.md`、`ch22-低多边形风格深度指南.md`，精简为简介。新写一章：像素之后的能力扩展方向——手绘数字绘画（Krita 简提）、3D 低模（Blender 概念）、矢量，每个给"何时转向"与"像素技能如何迁移"。指向扩展阅读（附录F）。标记"全书正文结束"。

- [ ] **Step 2: SVG 新生成**

生成：能力扩展路径图（1张）。`xmllint` 校验。

- [ ] **Step 3: 验证**

Run: `rg "手绘|3D低模|矢量|迁移|扩展" /Users/xiaokai/pixel-art/第五部-继续/继续04*.md` → 有
Run: `xmllint --noout /Users/xiaokai/pixel-art/images/继续/继续04-*.svg` → 无 error

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "继续04: 像素之后 (新写, 全书正文完结)"
```

### Task 7.5: 继续部整体验证

- [ ] **Step 1: 检查**

Run: `ls /Users/xiaokai/pixel-art/第五部-继续/ | wc -l` → 4
Run: `rg "extracted/|章节来源" /Users/xiaokai/pixel-art/第五部-继续/*.md` → 无
Run: `for f in /Users/xiaokai/pixel-art/images/继续/*.svg; do xmllint --noout "$f" || echo "FAIL: $f"; done` → 无 error

- [ ] **Step 2: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "继续部: 整体验证通过"
```

---

## Phase 8: 附录（7 个）

### Task 8.1: 附录A 资源罗盘（像素向）

**Files:**
- Write: `附录/附录A-资源罗盘.md`

- [ ] **Step 1: 读源 + 精简**

读 `源书/书A.../appendix-A-资源罗盘.md`。精简为像素向：保留调色板/Lospec/UI 参考/像素工具/字体/每日看图/原型素材；砍掉 3D 材质/建筑空间等非像素类。

- [ ] **Step 2: 验证 + Commit**

Run: `rg "Lospec|调色板|像素" /Users/xiaokai/pixel-art/附录/附录A*.md` → 有
```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "附录A: 资源罗盘(像素向)"
```

### Task 8.2: 附录B 术语中英对照表（合并 A+B）

**Files:**
- Write: `附录/附录B-术语中英对照表.md`

- [ ] **Step 1: 合并两书术语表**

读 `源书/书A.../appendix-B-术语中英对照表.md` 与 `源书/书B.../appendix.md`（附录D 术语表）。合并去重，按类别（八概念/色彩/构图/像素/3D/工具/动画/游戏美术）整理。每个术语一句定义。书 B 特有词（AA/Banding/Dithering/Hue Shifting/Jaggies/Mixels/Onion Skinning/Sub-Pixel 等）纳入。

- [ ] **Step 2: 验证 + Commit**

Run: `rg "AA|Banding|Jaggies|抖动|明度" /Users/xiaokai/pixel-art/附录/附录B*.md` → 都有
```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "附录B: 术语表(A+B合并)"
```

### Task 8.3: 附录C 练习速查卡（重编含 B）

**Files:**
- Write: `附录/附录C-练习速查卡.md`

- [ ] **Step 1: 重编**

读 `源书/书A.../appendix-C-练习速查卡.md`。把全书所有练习（A 的 + B 各章改写后并入的）重编为一张表：编号/名称/所在章（新编号）/难度 L1-L3/估时/核心步骤。验证每章练习都被收录。

- [ ] **Step 2: 验证 + Commit**

Run: `rg "L1|L2|L3" /Users/xiaokai/pixel-art/附录/附录C*.md | wc -l` → 练习数合理（30+）
Run: `rg "观察01|练手01|制作02" /Users/xiaokai/pixel-art/附录/附录C*.md` → 章号用新编号
```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "附录C: 练习速查卡(含B, 新编号)"
```

### Task 8.4: 附录D 自评量表（三能力版）

**Files:**
- Write: `附录/附录D-自评量表.md`

- [ ] **Step 1: 改写**

读 `源书/书A.../appendix-D-自评量表.md`。把五子能力月度表改为三能力（看/做/整）月度表，1-5 评分标准、雷达图绘制说明、三道月度反思题、季度复盘清单保留。

- [ ] **Step 2: 验证 + Commit**

Run: `rg "三能力|看.*做.*整|月度|雷达" /Users/xiaokai/pixel-art/附录/附录D*.md` → 有
Run: `rg "五子|五层" /Users/xiaokai/pixel-art/附录/附录D*.md` → 无
```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "附录D: 自评量表(三能力版)"
```

### Task 8.5: 附录E 像素风格文档模板

**Files:**
- Write: `附录/附录E-像素风格文档模板.md`

- [ ] **Step 1: 改写**

读 `源书/书A.../appendix-E-风格文档模板.md`。改为像素向：基本信息/3-5关键词/8-16色调色板+功能标签/角色规格/环境视觉层级/do&don't/技术规格(分辨率/色数/AA策略)/Gold Standard 图/修订记录。

- [ ] **Step 2: 验证 + Commit**

Run: `rg "调色板|分辨率|Gold Standard|do.*don" /Users/xiaokai/pixel-art/附录/附录E*.md` → 有
```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "附录E: 像素风格文档模板"
```

### Task 8.6: 附录F 推荐书单

**Files:**
- Write: `附录/附录F-推荐书单.md`

- [ ] **Step 1: 合并**

读 `源书/书A.../appendix-F-推荐书单.md`。保留按"卡点"组织的结构。加上书 B 的三本源书（Make Your Own Pixel Art / Pixel Logic / Pixel Art for Game Developers）标注为"像素技法深读"。学习路径图更新为五部。

- [ ] **Step 2: 验证 + Commit**

Run: `rg "Gurney|Molly Bang|Pixel Logic|Make Your Own" /Users/xiaokai/pixel-art/附录/附录F*.md` → 有
```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "附录F: 推荐书单(+B三源书)"
```

### Task 8.7: 附录G Aseprite 快捷键

**Files:**
- Write: `附录/附录G-Aseprite快捷键.md`

- [ ] **Step 1: 整理**

读 `源书/书B.../appendix.md`（附录A 快捷键）。整理为 Aseprite 快捷键速查表（按类别：工具/图层/帧/视图/文件）。

- [ ] **Step 2: 验证 + Commit**

Run: `rg "快捷键|Ctrl|B|N|V" /Users/xiaokai/pixel-art/附录/附录G*.md` → 有
```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "附录G: Aseprite快捷键"
```

---

## Phase 9: 全局重穿

### Task 9.1: 交叉引用重穿

**Files:** 全书 39 个 .md

- [ ] **Step 1: 找出所有旧编号引用**

Run: `rg "ch\d{2}|附录[A-F](?!-)|第[一二三四五六]部" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书" -g "!SPEC.md" -g "!PLAN.md"`

- [ ] **Step 2: 逐个替换为新编号**

按映射表（旧 ch00→序章；ch01→观察01；…；ch33→继续03；ch17/20/22→继续04 简介等）替换所有交叉引用。第一部~第五部用新部名。

- [ ] **Step 3: 验证无旧编号残留**

Run: `rg "ch\d{2}" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书" -g "!SPEC.md" -g "!PLAN.md" -g "!AUDIT-SVG.md"` → 应无（除有意保留的历史引用）
Run: `rg "选兵器|六部" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书" -g "!SPEC.md" -g "!PLAN.md"` → 应无

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "全局: 交叉引用重穿为新编号"
```

### Task 9.2: 分数→起点章路由表验证

**Files:** 观察01

- [ ] **Step 1: 检查路由表章号**

Run: `rg "观察|练手|风格|制作|继续" /Users/xiaokai/pixel-art/第一部-观察/观察01*.md` → 路由表所有目标章用新编号且真实存在

- [ ] **Step 2: 验证目标章存在**

对照 `ls` 各部目录，路由表指向的每个章号都有对应文件。

- [ ] **Step 3: Commit（若有改动）**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "全局: 路由表章号校验" --allow-empty
```

### Task 9.3: 目录（TOC）文件

**Files:**
- Create: `/Users/xiaokai/pixel-art/README.md` 或 `目录.md`

- [ ] **Step 1: 生成总目录**

写一个目录文件，列出序章 + 五部所有章 + 附录，含每章文件路径，作为全书入口。链接到各文件。

- [ ] **Step 2: 验证链接目标存在**

逐个 `ls` 确认目录中列出的文件路径都存在。

- [ ] **Step 3: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "全局: 总目录"
```

### Task 9.4: 图像路径统一检查

- [ ] **Step 1: 全书图像引用路径检查**

Run: `rg "!\[.*\]\(" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书" | rg -v "images/"` → 应无（所有图引用都指向 images/）

- [ ] **Step 2: 检查无提取位图/占位图**

Run: `rg "extracted/|placeholder_" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书" -g "!AUDIT-SVG.md"` → 应无

- [ ] **Step 3: 检查引用的 SVG 文件都存在**

提取所有 `images/...svg` 引用，逐个 `ls` 确认存在。

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "全局: 图像路径统一检查"
```

---

## Phase 10: 通读校对

### Task 10.1: 文风一致性全检

- [ ] **Step 1: 书 B 痕迹扫描**

Run: `rg "章节来源|B1_p|B2_p|B3_p|读者朋友|本章将介绍|本节将" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书" -g "!SPEC.md" -g "!PLAN.md" -g "!AUDIT-SVG.md"` → 应无

- [ ] **Step 2: 书 A 声调存在性扫描**

抽查每部首章：`rg "这一章解决什么问题|程序员类比|如果只记住|上手行动|小结" /Users/xiaokai/pixel-art/第一部-观察/观察01.md /Users/xiaokai/pixel-art/第二部-练手/练手01.md /Users/xiaokai/pixel-art/第四部-制作/制作02.md` → 结构装置在

- [ ] **Step 3: 修复发现的文风断层**

逐个修复扫描出的问题。

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "校对: 文风一致性"
```

### Task 10.2: 去重检查

- [ ] **Step 1: 找重复段落**

Run: `rg -l "剪影.*单元测试|CSS box model|视觉权重.*五规则" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书"` → 同一关键概念应只在其主章出现，他处只引用不重复展开。列出重复展开处。

- [ ] **Step 2: 合并/删减重复**

把重复展开改为简短引用（"详见练手02"）。

- [ ] **Step 3: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "校对: 去重"
```

### Task 10.3: SVG 渲染全检

- [ ] **Step 1: 全部 xmllint**

Run: `find /Users/xiaokai/pixel-art/images -name "*.svg" | while read f; do xmllint --noout "$f" 2>&1 | grep -v "^$" && echo "FAIL: $f"; done` → 无 FAIL

- [ ] **Step 2: 浏览器渲染抽查**

用 playwright 打开若干代表性 SVG（每部抽 2-3 张含像素网格、节点图、表格类），截图确认渲染正常。

- [ ] **Step 3: 修复渲染问题**

- [ ] **Step 4: Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "校对: SVG 渲染全检"
```

### Task 10.4: 最终验收对照成功标准

对照 SPEC 第十五节 8 条成功标准逐条验证：

- [ ] **Step 1: 文件齐全与编号**

Run: `find /Users/xiaokai/pixel-art -name "*.md" -not -path "*/源书/*" -not -name "SPEC.md" -not -name "PLAN.md" -not -name "AUDIT-SVG.md" -not -name "README.md" | wc -l` → 39

- [ ] **Step 2: 像素为核心**

确认非像素门类仅继续04 简介：`rg "手绘工作流|3D低模工作流|矢量工作流" /Users/xiaokai/pixel-art/ -g "*.md" -g "!源书" -g "!继续04*"` → 应无（除继续04）

- [ ] **Step 3: 三能力映射**

Run: `rg "三能力|看.*做.*整" /Users/xiaokai/pixel-art/00-序章*.md /Users/xiaokai/pixel-art/第一部-观察/观察01*.md` → 有，且有映射图

- [ ] **Step 4: 文风统一**（Task 10.1 已做）

- [ ] **Step 5: SVG 统一**（Task 10.3 已做 + 9.4 已做）

- [ ] **Step 6: 练习分级与速查卡**（附录C 已做）

Run: `rg "L[123]" /Users/xiaokai/pixel-art/附录/附录C*.md | wc -l` → 合理

- [ ] **Step 7: 贯穿角色练习**

Run: `rg "贯穿|角色项目" /Users/xiaokai/pixel-art/第二部-练手/练手01*.md /Users/xiaokai/pixel-art/第二部-练手/练手05*.md /Users/xiaokai/pixel-art/第四部-制作/制作02*.md` → 三处都有衔接

- [ ] **Step 8: 交叉引用网络**（Phase 9 已做）

- [ ] **Step 9: 最终 Commit**

```bash
cd /Users/xiaokai/pixel-art && git add -A && git commit -m "完成: 像素艺术整合书 (39章+7附录, 三能力五部, 全SVG)"
```

---

## Self-Review 结果

**1. Spec 覆盖：** SPEC 各节均有任务对应——目标(Task 1-10)、核心决策(贯穿)、结构(Task 1.2)、大纲(Task 3-8)、逐章合并(Task 3-8 每章)、文风规范(每章改写步骤+Task 10.1)、图像策略(Phase 2 审计+各章 SVG 步骤+Task 9.4/10.3)、练习整合(各章练习+Task 8.3)、附录(Phase 8)、编号交叉引用(Phase 9)、10阶段(本计划即 10 阶段)、成功标准(Task 10.4)。✓

**2. 占位符扫描：** 无 TBD/TODO；每章步骤给出具体来源文件与合并要点（取自 SPEC 第五节，已完整）。✓

**3. 类型/命名一致性：** 部名(观察/练手/风格/制作/继续)、章前缀、附录A-G、SVG 命名(`部名NN-figMM`)、验证命令在各任务一致。✓
