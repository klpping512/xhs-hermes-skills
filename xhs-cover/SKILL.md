---
name: xhs-cover
description: 生成小红书风格封面图（1080×1440px，3:4 比例）。HTML+CSS → Playwright 截图。
metadata:
  hermes:
    tags: [xiaohongshu, cover, design, screenshot, html]
---

# 小红书封面图生成

输入标题文案，生成小红书风格封面图（1080×1440px，3:4 比例）。

## Trigger

用户要求生成小红书封面图/封面设计时加载此 skill。

## Prerequisites

- **Playwright**: `pip3 install playwright && playwright install chromium`
- **Python 3**: 用于截图

## 6 种预设风格

| 风格 | 适用场景 | 主色 | 背景 |
|------|----------|------|------|
| morandi | 知识分享、方法论 | #c45c4a（暖红） | 低饱和米色渐变 |
| academic | 论文解读、技术分析 | #2563eb（学术蓝） | 冷灰蓝渐变 |
| dark | AI/编程/开源 | #a78bfa（亮紫） | 深蓝渐变 |
| mint | 工具推荐、效率方法 | #059669（翠绿） | 薄荷绿渐变 |
| sunset | 个人感悟、里程碑 | #ea580c（橘红） | 暖橘渐变 |
| bw | 观点输出、深度思考 | #18181b（纯黑） | 灰白渐变 |

## 自动选风格规则

- 论文/科研 → academic
- AI/编程/开源 → dark 或 morandi
- 工具/效率 → mint
- 个人感悟/发布 → sunset
- 观点/思考 → bw

## 核心流程

### Step 1: 解析输入

从用户文案中提取：
- **主标题**（最大字，1 行，核心概念/产品名）
- **副标题**（次大字，1-2 行，核心洞察）
- **标签**（3 个，每个 2-4 字）
- **角标**（左上小字，如 OPEN SOURCE / 工具推荐）
- **底部**（小字，3 个关键词用 · 分隔）

### Step 2: 生成 HTML

写入 `/tmp/xhs-cover.html`，模板结构：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { width: 1080px; height: 1440px; overflow: hidden;
         font-family: -apple-system, "PingFang SC", "Helvetica Neue", sans-serif; }
  .canvas {
    width: 1080px; height: 1440px;
    background: {风格背景};
    display: flex; flex-direction: column; justify-content: center;
    padding: 60px 70px; gap: 36px; position: relative;
  }
  .badge { display: inline-flex; align-items: center; gap: 10px;
           background: {主色 alpha 0.12}; border: 1.5px solid {主色 alpha 0.2};
           border-radius: 10px; padding: 12px 24px; font-size: 28px;
           font-weight: 700; color: {主色}; letter-spacing: 3px; width: fit-content; }
  .title-row { display: flex; align-items: baseline; gap: 20px; white-space: nowrap; }
  .title-main { font-size: 120-130px; font-weight: 900; color: {主色}; letter-spacing: -2px; }
  .title-secondary { font-size: 110-120px; font-weight: 900; color: {副色}; }
  .divider { width: 80px; height: 5px;
             background: linear-gradient(90deg, {主色}, {副色});
             border-radius: 3px; margin: 10px 0; }
  .tagline { font-size: 76-82px; font-weight: 800; color: {文字色};
             letter-spacing: 8px; line-height: 1.4; }
  .tagline .accent { color: {主色 或副色 淡化}; }
  .chip { background: {标签背景}; border-radius: 14px; padding: 18px 32px;
          font-size: 34px; font-weight: 700; letter-spacing: 2px; }
  .footer { font-size: 30px; color: {文字色 淡化}; font-weight: 600; letter-spacing: 2px; }
</style>
</head>
<body>
  <div class="canvas">
    <div class="badge">{角标}</div>
    <div class="title-row">
      <span class="title-main">{主标题}</span>
      <span class="title-secondary">{副标题}</span>
    </div>
    <div class="divider"></div>
    <div class="tagline">{副标题/核心洞察}</div>
    <div style="display: flex; gap: 16px; flex-wrap: wrap;">
      <span class="chip">{标签1}</span>
      <span class="chip">{标签2}</span>
      <span class="chip">{标签3}</span>
    </div>
    <div class="footer">{关键词1} · {关键词2} · {关键词3}</div>
  </div>
</body>
</html>
```

根据选定风格替换颜色变量：
- `{主色}` → 风格主色
- `{副色}` → 主色的互补色或淡化版
- `{文字色}` → 深色背景用白色，浅色背景用深灰
- `{风格背景}` → 风格对应的渐变背景
- `{标签背景}` → 主色 alpha 0.08-0.15
- 字号根据标题长度微调，确保不溢出

### Step 3: 截图

```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page(viewport={"width": 1080, "height": 1440})
    page.goto("file:///tmp/xhs-cover.html")
    page.wait_for_timeout(500)
    page.screenshot(path="/Users/ylanlll/Desktop/cover.png", full_page=False)
    browser.close()
```

### Step 4: 展示结果

展示图片给用户，询问是否调整（颜色、字号、布局、文案）。

## 风格配色详情

### morandi
- 主色: #c45c4a
- 副色: #d4a574
- 背景: linear-gradient(135deg, #f5f0eb, #e8e0d8)
- 文字: #2d2d2d
- 标签背景: rgba(196, 92, 74, 0.08)

### academic
- 主色: #2563eb
- 副色: #60a5fa
- 背景: linear-gradient(135deg, #f0f4f8, #dbe2ea)
- 文字: #1e293b
- 标签背景: rgba(37, 99, 235, 0.08)

### dark
- 主色: #a78bfa
- 副色: #c4b5fd
- 背景: linear-gradient(135deg, #0f172a, #1e1b4b)
- 文字: #f1f5f9
- 标签背景: rgba(167, 139, 250, 0.15)

### mint
- 主色: #059669
- 副色: #34d399
- 背景: linear-gradient(135deg, #ecfdf5, #d1fae5)
- 文字: #064e3b
- 标签背景: rgba(5, 150, 105, 0.08)

### sunset
- 主色: #ea580c
- 副色: #fb923c
- 背景: linear-gradient(135deg, #fff7ed, #fed7aa)
- 文字: #431407
- 标签背景: rgba(234, 88, 12, 0.08)

### bw
- 主色: #18181b
- 副色: #52525b
- 背景: linear-gradient(135deg, #fafafa, #e4e4e7)
- 文字: #18181b
- 标签背景: rgba(24, 24, 27, 0.06)

## 错误处理

| 场景 | 处理方式 |
|------|----------|
| Playwright 未安装 | 提示 `pip3 install playwright && playwright install chromium` |
| 标题过长溢出 | 自动缩小字号或截断 |
| 截图失败 | 报错并提示检查 Playwright chromium 安装 |

## 注意事项

1. **不用 emoji 做图标**: 封面设计用纯文字排版，不依赖 emoji
2. **字号自适应**: 主标题超过 4 字时适当缩小，确保不溢出 1080px 宽度
3. **Retina 清晰度**: Playwright 默认 1x 截图即可，小红书会压缩图片
