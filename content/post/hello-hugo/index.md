---
title: "建站记：用 Hugo + Stack 把 GitHub Pages 改成博客"
date: 2026-10-26
slug: hello-hugo
description: "原本仓库里只有一个 README，GitHub Pages 拿默认主题直接渲染。这次把它换成正经博客：Hugo 生成、Stack 主题、GitHub Actions 自动部署，顺便记下以后怎么发文章。"
categories:
    - 折腾
tags:
    - Hugo
    - GitHub Pages
    - GitHub Actions
image: cover.png
---

这个仓库一开始只有一份 `README.md` 加两张截图，GitHub Pages 用默认的 Jekyll 主题把 README 直接渲染成首页，能看，但不像博客。这次动手改造，目标有三个：

1. 有正经的博客布局：文章列表、归档、标签、分类、搜索、暗色模式；
2. 写文章只用 Markdown，不用碰 HTML；
3. push 之后自动构建部署，不需要本地装一堆东西。

<!--more-->

## 技术选型

| 方案 | 结论 |
| --- | --- |
| 继续用 Jekyll | 改动最小，但主题和插件生态都比较旧 |
| Hexo | 中文生态好，但每次写文章都要本地跑 Node 构建再提交产物 |
| **Hugo + Stack** | 单文件二进制、构建秒级，主题现代，配合 Actions 完全自动 |

最后选了 **Hugo + Stack 主题**，用 GitHub Actions 跑 `hugo` 并把产物发布到 Pages。

## 目录结构

```text
moony-hub.github.io/
├── .github/workflows/pages.yaml   # 自动构建 + 部署
├── hugo.toml                      # 站点配置
├── archetypes/post.md             # 新建文章模板
├── assets/img/avatar.png          # 头像等资源
├── content/
│   ├── _index.md                  # 首页
│   ├── page/                      # 关于 / 归档 / 搜索
│   └── post/                      # 文章，一篇一个目录（page bundle）
│       ├── dengbao-2-0/index.md
│       └── qq-traffic-analysis/
│           ├── index.md
│           └── qq_quic.png
├── static/favicon.ico
└── themes/hugo-theme-stack/       # 主题直接内置，不依赖 submodule
```

文章用 **page bundle** 的形式存放（一个目录一个 `index.md`），图片和文章放在一起，引用时直接写文件名，Hugo 会自动处理尺寸和懒加载。

## 自动部署

`.github/workflows/pages.yaml` 在每次 push 到 `main` 时执行：checkout → 安装 Hugo（extended）→ `hugo --minify` → 上传 `public/` 并用官方 `deploy-pages` 发布。

仓库只需要在 **Settings → Pages → Source** 里选 **GitHub Actions**，之后就不用手动管了。

## 以后怎么写新文章

最省事的方式是在 GitHub 网页上直接建文件：

1. 打开 `content/post/`，**Add file → Create new file**；
2. 文件名写成 `my-new-post/index.md`（斜杠会自动建目录）；
3. 贴上下面这段 front matter，正文用 Markdown 写；
4. Commit 到 `main`，等 Actions 跑完就上线了。

```markdown
---
title: "文章标题"
date: 2026-10-26
slug: my-new-post
description: "一句话摘要，会显示在列表和搜索结果里。"
categories:
    - 流量分析
tags:
    - Wireshark
image: cover.png          # 可选，同目录下的封面图
---

正文从这里开始。

<!--more-->
```

如果想在本地预览（需要先装 Hugo extended）：

```bash
hugo server -D     # 打开 http://localhost:1313 ，改文件自动刷新
hugo new content post/新文章/index.md
```

## 后续想加的东西

- 评论区（giscus，基于 GitHub Discussions，不需要额外服务器）
- 把打靶和漏洞复现的笔记也整理上来
- 也许换成自己的域名

先这样，能写起来最重要。
