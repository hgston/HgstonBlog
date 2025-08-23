---
title: "从零开始：使用 Hugo + PaperMod 搭建博客并部署到 GitHub Pages 和 Cloudflare Pages"
date: 2025-08-23T17:30:38+08:00
draft: false
---

## 前言

在这个数字化时代，拥有一个属于自己的技术博客是展示个人技能和分享知识的重要方式。本文将带你从零开始，使用 Hugo 静态网站生成器配合 PaperMod 主题搭建一个美观、快速的博客，并同时部署到 GitHub Pages 和 Cloudflare Pages，实现双重备份和更快的访问速度。

### 整体流程概览

1.  **安装环境**：安装 Git 和 Hugo。
2.  **创建网站**：使用 Hugo 命令行创建一个新站点。
3.  **安装主题**：将 PaperMod 主题添加到你的站点。
4.  **配置网站**：修改配置文件，让它变成你想要的样子。
5.  **写一篇文章**：创建你的第一篇博文。
6.  **本地预览**：在浏览器里查看网站效果。
7.  **部署到 GitHub**：设置仓库和 GitHub Actions，实现自动部署。
8.  **部署到 Cloudflare**：设置 Cloudflare Pages 实现双部署。

---

## 第一步：安装必要的环境

工欲善其事，必先利其器。在开始创建博客之前，我们需要先安装必要的工具。

### 1. 安装 Git

Git 是一个分布式版本控制系统，我们将用它来管理博客的源代码。

*   前往 [Git 官网](https://git-scm.com/) 下载并安装适合你操作系统的版本。
*   安装后，打开终端（Windows 用户可以使用 Git Bash），设置你的用户名和邮箱：
```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub邮箱"
```

### 2. 安装 Hugo (Extended 扩展版)

Hugo 是一个用 Go 语言编写的静态网站生成器，以速度快而著称。

*   **强烈建议安装 Extended 版**，因为很多主题（包括 PaperMod）需要它来处理 SCSS 文件。
*   **Windows (使用 Scoop)**:
  1.  打开 PowerShell，安装 Scoop：`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser; irm get.scoop.sh | iex`
  2.  安装 Hugo Extended: `scoop install hugo-extended`
*   **macOS (使用 Homebrew)**:
  1.  安装 Homebrew: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
  2.  安装 Hugo Extended: `brew install hugo`
*   **Linux (Ubuntu/Debian)**:
   *   参考官方指南: [https://gohugo.io/installation/linux/](https://gohugo.io/installation/linux/)

安装完成后，在终端输入 `hugo version`，确认安装成功且显示 **`hugo extended`**。

---

## 第二步：创建你的 Hugo 网站

现在我们可以开始创建博客网站了。

1.  打开终端，切换到你希望创建项目的目录（例如 `Documents`）。
2.  运行以下命令来创建一个名为 `myblog` 的新网站：
```bash
hugo new site myblog
cd myblog
```
这会创建一个包含基础文件夹结构的 `myblog` 目录。

---

## 第三步：安装 PaperMod 主题

PaperMod 是一个快速、简洁、响应式的 Hugo 主题，非常适合技术博客。

我们将使用 Git Submodule 的方式来安装主题，这是官方推荐的方式，便于后续更新。

1.  在你的网站根目录 (`myblog`) 下，运行：
```bash
git init  # 初始化git仓库
git add .
git commit -m "initial commit"
```
2.  将 PaperMod 主题添加为子模块：
```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```
3.  **重要**：编辑网站根目录下的配置文件 `hugo.yaml` (如果不存在，可能是 `hugo.toml` 或 `hugo.json`，建议新建 `hugo.yaml`)。删除所有内容，并添加以下基本配置来启用主题：

```yaml
baseURL: "https://yourusername.github.io/" # 稍后替换为你的 GitHub 用户名
languageCode: "zh-cn"
title: "我的技术博客"
theme: "PaperMod"

outputFormats:
  HTML:
    mediaType: "text/html"
    isPlainText: false
    isHTML: true

outputs:
  home: ["HTML", "RSS", "JSON"]

params:
  defaultTheme: "auto" # 主题模式：auto, light, dark
  # 其他详细配置可后续在 params 下添加，参考主题Wiki
```

---

## 第四步：创建一篇文章

有了网站和主题，现在让我们来写第一篇文章。

1.  使用 Hugo 命令创建一篇名为 `我的第一篇文章` 的博文：
```bash
hugo new posts/我的第一篇文章.md
```
这会在 `content/posts/` 目录下生成一个 Markdown 文件。

2.  用任何文本编辑器（如 VS Code）打开这个文件 `content/posts/我的第一篇文章.md`。你会看到文件开头有 `---` 分隔的"前言部分"（Front Matter），下面是正文。

```markdown
---
title: "我的第一篇文章"
date: 2023-10-27T15:19:02+08:00
draft: true # 草稿状态，设为 false 或发布时 hugo 命令忽略草稿
---

## 欢迎来到我的博客！

这里是正文，使用 **Markdown** 语法书写。

-   列表项
-   另一个列表项

[这是一个链接](https://google.com)
```

3.  将 `draft: true` 改为 `draft: false`，否则文章不会被构建发布。

---

## 第五步：本地预览

在正式发布之前，我们可以在本地预览网站效果。

1.  在终端里，确保你在网站根目录 (`myblog`)，然后启动 Hugo 本地服务器：
```bash
hugo server -D
```
`-D` 参数代表同时构建草稿文章，方便预览。

2.  命令行会输出一个本地地址（通常是 `http://localhost:1313`）。在浏览器中打开它，你就能看到你的网站和文章了！

3.  **随时保存文件**，浏览器中的页面会自动刷新显示最新效果。

---

## 第六步：部署到 GitHub Pages

GitHub Pages 是 GitHub 提供的静态网站托管服务，非常适合托管个人博客。

### A. 创建 GitHub 仓库

1.  在 GitHub 上创建一个**新的公共仓库**，命名为 **`你的用户名.github.io`** (例如，用户名叫 `john`，仓库名就叫 `john.github.io`)。这是 GitHub Pages 的硬性规定。

### B. 配置 GitHub Actions

这是实现**自动化部署**的关键。Hugo 会帮你生成一个工作流配置文件。

1.  在网站根目录下创建文件夹和文件：`.github/workflows/hugo.yaml`。
2.  将以下代码复制到 `hugo.yaml` 文件中：

```yaml
name: Deploy Hugo to GitHub Pages

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Setup Pages
        uses: actions/configure-pages@v3

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v2
```

3.  修改网站的配置文件 `hugo.yaml`，确保 `baseURL` 是你的 GitHub Pages 地址：
```yaml
baseURL: "https://yourusername.github.io/" # 替换 yourusername
```

### C. 推送到 GitHub

1.  在终端中，运行以下命令将代码推送到 GitHub：
```bash
git add .
git commit -m "添加网站文件和GitHub Actions工作流"
# 将本地仓库链接到你的GitHub远程仓库
git branch -M main
git remote add origin https://github.com/yourusername/yourusername.github.io.git # 替换你的仓库URL
git push -u origin main
```

2.  完成后，打开你的 GitHub 仓库，点击 **`Actions`** 标签页。你会看到一个工作流正在运行。等待几分钟，它完成后会显示绿色的对勾。

3.  然后进入仓库的 **`Settings` -> `Pages`** 页面。在 `Build and deployment` 下，选择 **`GitHub Actions`** 作为源。

4.  现在，访问 `https://yourusername.github.io`，你的网站就应该上线了！

---

## 第七步：同时部署到 Cloudflare Pages

Cloudflare Pages 是 Cloudflare 提供的静态网站托管服务，部署更简单，并且通常速度更快。

### A. 连接仓库

1.  登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)，进入 **`Pages`**。
2.  点击 **`Create a project`** -> **`Connect to Git`**。
3.  授权 Cloudflare 访问你的 GitHub 账户，并选择你刚创建的那个仓库 (`yourusername.github.io`)。

### B. 配置构建设置

Cloudflare 会自动检测到是 Hugo 项目，但我们需要确认设置：
*   **Build command:** `hugo --minify`
*   **Build output directory:** `public`
*   **Environment variables:** 点击 **`Add variable`** 添加一个变量：
    *   Variable name: `HUGO_VERSION`
    *   Value: `latest` (或者一个具体的稳定版本号，如 `0.125.4`)

### C. 开始部署

点击 `Save and Deploy`。Cloudflare 会自动拉取你的代码，运行构建命令，并将网站部署到一个 `*.pages.dev` 的临时域名上。

### D. (可选) 绑定自定义域名

1.  在 Cloudflare Pages 项目控制台，点击 **`Custom domains`** -> **`Set up a custom domain`**。
2.  输入你已经托管在 Cloudflare 上的域名（或者按照指引添加新域名）。
3.  这样，你的网站就可以通过你自己的域名访问了。

---

## 总结

恭喜你！现在你拥有了一个：
1.  **源代码仓库**：位于 GitHub 的 `main` 分支。
2.  **自动化部署**：每次你 `git push` 后，GitHub Actions 会自动构建并将网站发布到 **GitHub Pages**。
3.  **双平台部署**：Cloudflare Pages 也通过监听你的 GitHub 仓库实现了自动部署。

你以后只需要在本地 `myblog` 文件夹里用 `hugo new` 写文章，然后 `git add .`, `git commit`, `git push`，两边的网站就都会自动更新了！享受你的新博客吧！