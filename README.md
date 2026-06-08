# xhs-hermes-skills

小红书 Hermes Agent Skills — 帖子提取 + 封面图生成。

## Skills

| Skill | 功能 | 触发方式 |
|-------|------|----------|
| `xhs` | 提取小红书帖子（文字+图片）→ Obsidian 笔记 | 给出小红书链接 |
| `xhs-cover` | 生成小红书风格封面图（1080×1440px） | 给出标题文案 |

## 安装

将两个目录复制到 `~/.hermes/skills/`：

```bash
cp -r xhs/ ~/.hermes/skills/
cp -r xhs-cover/ ~/.hermes/skills/
```

## 前置依赖

```bash
# 封面截图需要
pip3 install playwright
playwright install chromium

# 帖子提取需要 cookies 文件
# Chrome 登录小红书后，在 DevTools Console 运行导出脚本
# 保存到 ~/.hermes/xhs-cookies.json
```

## 存储路径

- 笔记：`~/Desktop/klpping/知识库/xhs/{YYYY-MM-DD} {短标题}.md`
- 图片：`~/Desktop/klpping/知识库/xhs/img/`

## 风格预设（xhs-cover）

| 风格 | 场景 | 主色 |
|------|------|------|
| morandi | 知识分享 | #c45c4a |
| academic | 论文解读 | #2563eb |
| dark | AI/编程 | #a78bfa |
| mint | 工具推荐 | #059669 |
| sunset | 个人感悟 | #ea580c |
| bw | 深度思考 | #18181b |
