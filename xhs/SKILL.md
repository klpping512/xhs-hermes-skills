---
name: xhs
description: 提取小红书帖子内容（文字+图片），保存为 Obsidian 笔记。Cookie 认证 + __INITIAL_STATE__ 解析。
metadata:
  hermes:
    tags: [xiaohongshu, scraping, extraction, obsidian, content-curation]
---

# 小红书帖子提取

从小红书帖子 URL 提取完整内容（文字、图片），输出 Peter Thiel 风格的决策型笔记。

## Trigger

用户给出小红书链接并要求提取/保存/归档内容时加载此 skill。

## Prerequisites

- **Cookies 文件**: `~/.hermes/xhs-cookies.json` — 从 Chrome 导出的小红书 cookies
- **Python 3**: 用 `urllib.request` + `ssl` 做 HTTP 请求
- **存储目录**: `~/Desktop/klpping/知识库/xhs/img/`

## 核心流程

### Step 0: Cookie 认证

检查 `~/.hermes/xhs-cookies.json` 是否存在。如不存在，引导用户导出：

```javascript
// 在 Chrome 打开 xiaohongshu.com 并登录后，在 DevTools Console 运行：
copy(JSON.stringify(document.cookie.split('; ').map(c => {
  const [name, ...rest] = c.split('=');
  return { name, value: rest.join('='), domain: '.xiaohongshu.com', path: '/',
    expires: Date.now()/1000 + 86400*30, size: name.length + rest.join('=').length,
    httpOnly: false, secure: false, session: false, priority: 'Medium',
    sameParty: false, sourceScheme: 'Secure', sourcePort: 443 };
})))
```

保存剪贴板内容到 `~/.hermes/xhs-cookies.json`。

### Step 1: 解析链接

从 URL 提取帖子 ID（24 位 hex）和 xsec_token 参数。

### Step 2: 获取帖子数据

```python
import json, urllib.request, ssl, re

with open('/Users/ylanlll/.hermes/xhs-cookies.json') as f:
    cookies = json.load(f)
cookie_str = '; '.join(f"{c['name']}={c['value']}" for c in cookies)

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

req = urllib.request.Request('<帖子URL>')
req.add_header('Cookie', cookie_str)
req.add_header('User-Agent', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36')

resp = urllib.request.urlopen(req, timeout=15, context=ctx)
html = resp.read().decode('utf-8', errors='ignore')

m = re.search(r'window\.__INITIAL_STATE__\s*=\s*(\{.+?\})\s*</script>', html, re.DOTALL)
raw = m.group(1).replace('undefined', 'null')
data = json.loads(raw)

# 帖子数据在: data['note']['noteDetailMap'][<key>]['note']
# 包含: title, desc, type, time, user, imageList, video, interactInfo, ipLocation
```

**Pitfall: Cookies 过期** — 如果请求被重定向到 404/错误页，说明 cookies 过期，需重新导出。

**Pitfall: `undefined` → `null`** — XHS 的 JS 输出含 undefined，JSON.parse 前必须替换。

### Step 3: 下载图片

```bash
curl -L -o ~/Desktop/klpping/知识库/xhs/img/{post_id}_{n}.jpg \
  -H "Referer: https://www.xiaohongshu.com/" <图片URL>
```

图片使用相对路径 `img/{post_id}_{n}.jpg` 嵌入 Markdown。

### Step 4: 输出 Markdown

**存储路径**: `~/Desktop/klpping/知识库/xhs/{YYYY-MM-DD} {短标题}.md`

短标题不超过 15 字，是核心洞察的极简概括。

**写作风格**: Peter Thiel 式——直接、反直觉、一句话给判断。笔记是决策工具，不是知识库。

**文件结构（无 YAML frontmatter）**:

```markdown
# 一句话核心洞察（反直觉的判断，不是描述性标题）

核心论点，2-3句话。直接给出"大多数人觉得X，但其实Y"的判断。
不废话，不铺垫。

**与我的关联：** 读取 Hermes memory 了解用户背景，说清楚这个内容跟用户有什么关系。

**值得深挖吗：** 是/否。一句话理由。

> [!tip]- 详情
> 帖子核心内容的结构化整理（折叠状态，点开才看到）：
> - 从 desc 中提炼，清理 #xxx[话题]# 标记
> - 按逻辑结构分节，保留关键数据和结论
> - 图片用 `![图N](img/xxx.jpg)` 嵌入

> [!info]- 笔记属性
> - **来源**: 小红书 · 作者名
> - **帖子ID**: xxx
> - **链接**: 原始链接
> - **日期**: YYYY-MM-DD
> - **类型**: image
> - **互动**: N赞 / N收藏 / N评论
> - **标签**: 标签1, 标签2, ...
```

**关键约束**:
- 折叠区域外的可见内容**不超过 6 行**
- 标题必须是洞察/判断，不是"XX帖子的总结"
- 图片使用相对路径 `img/xxx.jpg`

## 错误处理

| 场景 | 处理方式 |
|------|----------|
| Cookies 文件不存在 | 引导用户导出，提供一键复制脚本 |
| Cookies 过期（404/重定向） | 提示重新导出 |
| 帖子不存在/已删除 | 报错并跳过 |
| 图片下载失败 | 记录失败 URL，继续处理其他图片 |

## 注意事项

1. **敏感词处理**: 小红书文案中 Claude → 克莱德，ChatGPT → GPT
2. **反爬间隔**: 批量提取时每个帖子间隔 3 秒
3. **路径确认**: 执行前先 `ls` 确认目录存在
4. **Hermes memory 读取**: 用 `memory` 工具获取用户背景，用于生成"与我的关联"部分

## 与 xhs-evl-cl 的关系

- 本 skill (`xhs`): **读**小红书 — 提取帖子内容归档到 Obsidian
- `xhs-evl-cl`: **写**小红书 — 文案优化、封面设计
- 两者互补，不重叠
