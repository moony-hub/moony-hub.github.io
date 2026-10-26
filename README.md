# moony-hub.github.io

个人博客，线上地址：<https://moony-hub.github.io>

用 [Hugo](https://gohugo.io/) 生成，主题为 [Stack](https://github.com/CaiJimmy/hugo-theme-stack)，通过 GitHub Actions 自动构建并部署到 GitHub Pages。

## 内容

| 位置 | 说明 |
| --- | --- |
| `content/post/` | 文章，一篇一个目录（page bundle），图片与 `index.md` 放在一起 |
| `content/page/` | 独立页面：关于、归档、搜索 |
| `hugo.toml` | 站点配置（标题、菜单、侧边栏、小部件、Markdown 渲染） |
| `themes/hugo-theme-stack/` | 主题源码（直接内置，不用 submodule，方便网页端维护） |
| `.github/workflows/pages.yaml` | push 到 `main` 后自动构建并发布 |

原有的课程作业（等保 2.0、QQ 流量分析）已整理为正式文章，分别位于 `content/post/dengbao-2-0/` 与 `content/post/qq-traffic-analysis/`，截图也一并放进了文章目录。

## 发一篇新文章

### 方式一：GitHub 网页（推荐，无需本地环境）

1. 进入 `content/post/`，点 **Add file → Create new file**
2. 文件名填 `my-new-post/index.md`（带斜杠会自动创建目录）
3. 复制 `archetypes/post.md` 的模板，改掉标题、日期、`slug`、分类和标签
4. 提交到 `main` 分支，等待 Actions 跑完即可访问 `https://moony-hub.github.io/p/my-new-post/`

### 方式二：本地

需要安装 Hugo extended（>= 0.157）。

```bash
hugo new content post/my-new-post/index.md   # 按模板新建
hugo server -D                               # 本地预览 http://localhost:1313
hugo --gc --minify                           # 生成静态文件到 public/
```

## 部署

- 仓库 **Settings → Pages → Source** 设为 **GitHub Actions**
- 每次 push 到 `main` 触发 `.github/workflows/pages.yaml`：安装 Hugo → `hugo --minify` → 上传 `public/` → `deploy-pages`
- 也可以在 Actions 页面手动 **Run workflow**

## 备注

- 主题为 GPL-3.0，LICENSE 保留在 `themes/hugo-theme-stack/LICENSE`
- 站点文章内容采用 CC BY-NC-SA 4.0
