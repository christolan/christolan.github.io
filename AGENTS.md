# 博客维护

维护帽子（Javen）的 Hexo 博客"煎饼果子不放葱"。

## 仓库概况

| 项目 | 内容 |
|------|------|
| 路径 | `~/christolan.github.io` |
| 框架 | Hexo 8.1.1 |
| 作者 | 帽子 |
| 主题 | Butterfly |
| 主题源码位置 | `node_modules/hexo-theme-butterfly/` |
| 文章目录 | `source/_posts/*.md` |
| 部署方式 | 推送到 `main` → GitHub Actions 自动构建 → GitHub Pages |

## CI/CD 流程

```
git push origin main
  → GitHub Actions: npm install
  → npm run build  (= hexo generate)
  → 上传 public/ 到 GitHub Pages
```

## 写一篇新文章

### 1. 使用 Hexo 创建文章

```bash
cd ~/christolan.github.io
npx hexo new post "<english-kebab-case-slug>"
```

命令里的标题用英文 kebab-case slug（如 `how-transformer-inference-works`），用于生成文件名和 permalink；中文展示标题写在 front matter 的 `title` 中。

### 2. 填写 Hexo front matter

```yaml
---
title: <文章标题>
date: YYYY-MM-DD HH:MM:SS
published: true
categories:
  - AI          # 或 游戏 / 兴趣 / 技术
tags:
  - AI
  - <话题>
---
```

permalink 使用 Hexo 默认格式 `:year/:month/:day/:title/`，`:title` 取文件名（去掉 `.md` 后缀）。**文件名必须为英文 kebab-case**。`date` 作为正式发布时间，会影响首页、归档和 RSS 的排序。

### 3. 撰写正文

- **开头**：用一句加粗的话抓住读者
- **结构**：分章节用自然段落展开，不要写成要点罗列
- **核心术语**：一律英文（beam search、nucleus sampling、attention 等）
- **篇幅**：完整技术讲解 800-2000 字
- **不要外链充数**：把答案写完整，别甩链接

### 4. 本地预览（可选）

```bash
cd ~/christolan.github.io
npm run build   # 或：hexo generate
hexo server     # http://localhost:4000
```

### 5. 发布上线

```bash
cd ~/christolan.github.io
git add source/_posts/<slug>.md
git commit -m "docs: add article <title>"
git push origin main
```

## 标签参考

已有 AI 标签：`AI`、`RAG`、`Transformer`、`解码策略`、`推理模型`、`流式`、`Agent`、`多模态`、`性能优化`

按需新增标签。分类：`AI` / `游戏` / `技术` / `兴趣`。

## 主题配置

博客当前使用 Butterfly 主题。

### ⚠️ Pitfall：主题源码在 node_modules 里，不是 themes/ 目录

Butterfly 通过 npm 安装，源码位于 `node_modules/hexo-theme-butterfly/`。`themes/` 目录下只有 `.gitkeep`，是空的。需要查看模板或样式时，去 `node_modules/` 下找：

- 模板：`node_modules/hexo-theme-butterfly/layout/`（.pug 文件）
- 样式：`node_modules/hexo-theme-butterfly/source/css/`（.styl 文件）
- 主题默认配置：`node_modules/hexo-theme-butterfly/_config.yml`

用户可覆盖的配置写在项目根目录的 `_config.butterfly.yml`，不要直接改 `node_modules/` 下的文件。

## 分页配置

Hexo 的分页由 `_config.yml` 全局 `per_page` 控制（默认 10），同时影响首页、归档、分类、标签页。

### 单独调整归档页分页

`hexo-generator-archive` 支持独立的 `archive_generator.per_page`，优先级高于全局 `per_page`。在 `_config.yml` 中添加：

```yaml
archive_generator:
  per_page: 0     # 0 = 禁用归档分页（所有文章一页展示），或设 30 等更大值
  yearly: true
  monthly: true
```

同理，`index_generator` 和 `category_generator`、`tag_generator` 也各有独立 `per_page`，可单独调大而不影响其他页面。

### 分页优先级链

```
各 generator 的 per_page（如 archive_generator.per_page）
  → 若未设置，继承全局 _config.yml 的 per_page
    → 若全局也未设置，默认 10
```

源码逻辑见 `node_modules/hexo-generator-archive/index.js`。

## 功能/插件配置管理

Butterfly 主题的功能开关（评论、聊天、侧边栏等）集中在 `_config.butterfly.yml`。

### 评论系统

评论配置在 `_config.butterfly.yml` 约第 396-510 行，包含：
- **主开关**：`comments.use` — 留空则禁用所有评论
- **计数显示**：`comments.count`（文章顶部）、`comments.card_post_count`（首页卡片）
- **各评论系统**：Waline / Gitalk / Disqus / Valine / Utterances / Giscus / Twikoo / Artalk 等，各自有独立配置块

**当前状态**：评论已禁用（`use:` 为空），Waline serverURL 已清空，Gitalk 凭证已清空。

### ⚠️ Pitfall：凭证泄露在 git 历史中

清空 `_config.butterfly.yml` 中的凭证只是从当前文件移除，**旧值仍留在 git 历史中**。涉及 OAuth client_secret、API key 等敏感信息时：
1. 立即到对应平台撤销/重新生成凭证
2. 如需彻底清除历史，用 `git filter-branch` 或 BFG Repo-Cleaner（慎重操作）

### 关闭功能的通用模式

1. 主开关置空/设 false
2. 关闭相关 UI 元素（计数、按钮、侧边栏 widget）
3. 清空凭证字段（安全考虑，即使功能已关）
4. 本地 `hexo clean && hexo generate` 验证构建
5. commit + push

## 依赖升级

### 检查过期依赖

```bash
cd ~/christolan.github.io
npm outdated --json
```

`npm outdated` 返回包含 `current` / `wanted` (semver 范围) / `latest` 三个版本号的 JSON。exit code 1 仅表示有过期包，不是错误。

### 升级流程

1. 修改 `package.json` 中的版本号到 `latest`
2. 同步更新 `"hexo": {"version": "..."}` 字段（`package.json` 中的 hexo 元数据版本）
3. `npm install`
4. `hexo clean && hexo generate` — 构建验证必须 0 错误
5. `hexo server -p <port>` + `curl` 验证首页、文章页、分类/标签页均返回 200
6. 确认无误后 commit + push

### ⚠️ Major 升级风险

多个包同时升 Major 版本风险较高，但以下路径已验证可行（2026-05-06）：
- hexo 7.2.0 → 8.1.1 ✓
- hexo-theme-butterfly 4.13.0 → 5.5.4 ✓
- hexo-generator-index 3.0.0 → 4.0.0 ✓
- hexo-renderer-marked 6.3.0 → 7.0.1 ✓

上述四个 Major 一次性升级，构建 84 文件 0 错误，页面全部 200。若日后出现新 Major 版本，仍建议逐包升级并构建验证。

### 验证页面清单

升级后至少抽查：
- 首页 `/`
- 一篇文章页（新 + 旧各一篇）
- 分类页 `/categories/<name>/`
- 标签页 `/tags/<name>/`
- 归档页 `/archives/`

## 常用命令

```bash
cd ~/christolan.github.io
hexo generate   # 构建 → public/
hexo server     # 本地预览
hexo deploy     # 推送到 GitHub Pages（已弃用——请用 git push）
hexo clean      # 清除缓存和构建产物
npm run build   # 同 hexo generate
```
