# 饺子の日志

基于 [Hexo](https://hexo.io/) + [Fluid](https://github.fluid-dev.com/) 主题的个人博客源码。

- 线上地址：<https://buildgw.github.io/>
- 源码分支：`dev`
- 发布分支：`master`（由 `hexo deploy` 自动生成，请勿手动修改）

## 环境要求

| 依赖 | 版本要求 | 说明 |
| --- | --- | --- |
| Node.js | ≥ 18（实测 22.14.0） | 运行 Hexo |
| npm | ≥ 8 | 安装依赖 |
| Git | 任意较新版本 | 部署需要；本机 SSH key 须已在 GitHub 登记 |

## 目录结构

```
.
├── _config.yml              # 站点主配置（标题、URL、主题、部署地址等）
├── _config.fluid.yml        # 主题配置（菜单、配色、首页、文章页、评论等）
├── package.json             # 依赖与脚本
├── scaffolds/               # 新建文章/页面的模板
├── source/
│   ├── _posts/              # 已发布文章（按分类分目录）
│   │   └── <分类>/<文章>/    # post_asset_folder 开启后的文章配图目录
│   ├── _drafts/             # 草稿，不会生成页面
│   └── (页面、图片等)
├── themes/fluid/            # 内嵌主题（不是 npm 依赖，升级需整目录替换）
├── .deploy_git/             # hexo deploy 的临时仓库（已忽略）
└── public/                  # 生成的静态站点（已忽略）
```

被 `.gitignore` 忽略、不要提交的内容：`public/`、`.deploy_git/`、`db.json`、`node_modules/`、`.idea/`、`*.log`。

## 本地运行

```bash
npm install
npm run server          # http://localhost:4000
```

也可以指定端口：

```bash
npx hexo server --port 4000
```

服务器支持热更新：改 `source/` 下的文章或配置后刷新页面即可（改 `_config.yml` 这类配置建议重启）。

## 写一篇新文章

```bash
npx hexo new post "文章标题"
```

生成的文件在 `source/_posts/` 下，建议按分类移动目录，例如 `source/_posts/计算机基础/线性代数/`。

### Front-matter 规范

```yaml
---
title: 线性代数基础01—向量
date: 2025-07-18 13:20:30
categories: [计算机基础, 线性代数]
tags: [线性代数, 图形学]
---
```

约定：

- 用 `tags`（复数）；不要用 `tag`、也不要写 `archive` 这类非 Hexo 字段。
- `categories` 统一用两级，见下面的分类体系。
- 文件名的 slug 决定 URL（`/年/月/日/文件名/`），改 `title` 不影响链接，改文件名会。
- 正文一级标题可选：主题会先渲染 `title`，正文再写 `#` 会重复显示，视文章风格自行取舍。

### 分类体系

| 一级分类 | 二级分类 | 示例文章 |
| --- | --- | --- |
| 计算机基础 | 线性代数 | 向量、坐标系、三角函数 |
| AI | SpringAI、openclaw、MCP | Spring AI 概览、OpenClaw 安装与使用、MCP 教程 |
| 数据库 | Redis | Redis 入门 |
| 软考 | 高项 | 项目管理过程组 |
| 实用工具 | Cloudflare | 自建订阅链接（草稿） |

### 草稿

写一半的文章放进 `source/_drafts/`，`_config.yml` 中 `render_drafts: false`，因此不会被生成、也不会出现在列表里。要发布时把文件移到 `_posts/`。

## 配图

站点已开启 `post_asset_folder: true`：每篇文章对应一个**与文件名同名的目录**，图片放在里面。

```
source/_posts/线性代数/1线性代数基础-向量.md
source/_posts/线性代数/1线性代数基础-向量/001-1.jpg
```

正文用标签引用，不要写 `./assets/001-1.jpg` 这种相对路径（部署后会 404）：

```
{% asset_img 001-1.jpg 向量的模 %}
```

也可以直接写站点绝对路径（图片放到 `source/img/` 下时）：`![](/img/xxx.png)`。

## 构建与发布

```bash
npm run clean           # 清理 public/ 与缓存
npm run build           # 生成静态文件到 public/
npm run deploy          # 生成并推送到远端 master 分支
```

发布原理：

1. `hexo generate` 生成 `public/`；
2. `hexo-deployer-git` 把 `public/` 同步到 `.deploy_git/` 并提交（提交信息形如 `Site updated: ...`）；
3. 强推 `.deploy_git` 到 `master`，GitHub Pages 从 `master` 发布。

因此**不要直接改 `master` 分支**，内容会在下次发布时被覆盖。日常改动都提交到 `dev`：

```bash
git add -A
git commit -m "说明"
git push master dev:dev
```

注意远端名是 `master`（不是常见的 `origin`），部署地址与本机远端都使用 SSH：

- 仓库远端：`git@github.com:BuildGW/BuildGW.github.io.git`（SSH）
- `_config.yml` 中 `deploy.repo` 同样是 SSH 地址，`hexo deploy` 不再需要输入账号密码

## 常见问题

**图片显示不出来 / 404**
检查图片是否在“文章同名目录”内，正文是否用了 `{% asset_img %}`，而不是 `./assets/...` 相对路径。

**公式显示为 `$$ ... $$` 原文**
当前 `_config.fluid.yml` 中 `post.math.enable: false`，且渲染器是 `hexo-renderer-marked`，所以 LaTeX 不会转换。要启用需要换渲染器（如 `hexo-renderer-markdown-it` 或 katex 方案）后再打开 `math`，属于待办事项。

**导航里“关于”是 404**
`source/about/index.md` 尚未创建；需要时执行 `npx hexo new page about` 并在 front-matter 指定 `layout: about`。

**发布后内容没变**
GitHub Pages 有构建延迟（通常 1–2 分钟）；另外确认是否真的推到了 `master`，以及浏览器是否命中旧缓存。

## 待办

- [ ] 启用数学公式渲染（需更换 Markdown 渲染器）
- [ ] 创建“关于”页面
- [ ] 清除 `source/_posts/高项/img.png` 这张未引用的图片
- [ ] 补完 `source/_drafts/自建订阅链接.md`

## 备注

仓库默认分支是 `master`（发布产物），所以 GitHub 仓库首页默认看不到这个 README；在 `dev` 分支页可以查看。若希望仓库首页直接展示说明，可把 GitHub 的默认分支改为 `dev`。
