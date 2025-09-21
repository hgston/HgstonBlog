---
title: "从零开始：使用 Hugo + PaperMod 搭建博客并部署到 Cloudflare Pages"
date: 2025-08-23T17:30:38+08:00
draft: false
categories: ["环境部署"]
---

## 前言

在数字化时代，拥有个人技术博客是展示技能和分享知识的重要方式。本文将带你使用 Hugo 静态网站生成器配合 PaperMod 主题搭建博客，并部署到 Cloudflare Pages。

### 整体流程概览

1.  安装环境：安装 Git 和 Hugo。
2.  创建网站：使用 Hugo 命令行创建新站点。
3.  安装主题：将 PaperMod 主题添加到站点。
4.  配置网站：修改配置文件。
5.  写一篇文章：创建第一篇博文。
6.  本地预览：在浏览器中查看网站效果。
7.  创建 GitHub 仓库并推送代码：将网站源码上传到 GitHub。
8.  部署到 Cloudflare：设置 Cloudflare Pages 实现部署。

---

## 第一步：安装必要的环境

在创建博客之前，需要先安装必要的工具。

### 安装 Git

Git 是分布式版本控制系统，用于管理博客源代码。

*   前往 [Git 官网](https://git-scm.com/) 下载并安装适合操作系统的版本。
*   安装后，打开终端（Windows 用户可以使用 Git Bash），设置用户名和邮箱：
```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub邮箱"
```

### 安装 Hugo (Extended 扩展版)

Hugo 是用 Go 语言编写的静态网站生成器，以速度快著称。

*   强烈建议安装 Extended 版，因为很多主题（包括 PaperMod）需要它来处理 SCSS 文件。
*   Windows (使用 Scoop):
  1.  打开 PowerShell，安装 Scoop：`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser; irm get.scoop.sh | iex`
  2.  安装 Hugo Extended: `scoop install hugo-extended`
*   macOS (使用 Homebrew):
  1.  安装 Homebrew: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
  2.  安装 Hugo Extended: `brew install hugo`
*   Linux (Ubuntu/Debian):
   *   参考官方指南: [https://gohugo.io/installation/linux/](https://gohugo.io/installation/linux/)

安装完成后，在终端输入 `hugo version`，确认安装成功且显示 `hugo extended`。

---

## 第二步：创建你的 Hugo 网站

现在可以开始创建博客网站了。

1.  打开终端，切换到希望创建项目的目录（例如 `Documents`）。
2.  运行以下命令来创建一个名为 `myblog` 的新网站：
```bash
hugo new site myblog
cd myblog
```
这会创建一个包含基础文件夹结构的 `myblog` 目录。

---

## 第三步：安装 PaperMod 主题

PaperMod 是快速、简洁、响应式的 Hugo 主题，适合技术博客。

使用 Git Submodule 的方式来安装主题，这是官方推荐的方式，便于后续更新。

1.  在网站根目录 (`myblog`) 下，运行：
```bash
git init  # 初始化git仓库
git add .
git commit -m "initial commit"
```
2.  将 PaperMod 主题添加为子模块：
```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```
3.  编辑网站根目录下的配置文件 `hugo.yaml` (如果不存在，可能是 `hugo.toml` 或 `hugo.json`，建议新建 `hugo.yaml`)。删除所有内容，并添加以下基本配置来启用主题：

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

有了网站和主题，现在来写第一篇文章。

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

在正式发布之前，可以在本地预览网站效果。

1.  在终端里，确保在网站根目录 (`myblog`)，然后启动 Hugo 本地服务器：
```bash
hugo server -D
```
`-D` 参数代表同时构建草稿文章，方便预览。

2.  命令行会输出一个本地地址（通常是 `http://localhost:1313`）。在浏览器中打开它，就能看到网站和文章了！

3.  随时保存文件，浏览器中的页面会自动刷新显示最新效果。

---

## 第六步：创建 GitHub 仓库并推送代码

在部署到 Cloudflare Pages 之前，需要先将网站源码上传到 GitHub 仓库。

### 创建 GitHub 仓库

1.  登录 GitHub，点击右上角的 `+`，选择 `New repository`。
2.  仓库名称建议使用 `yourusername.github.io`（将 `yourusername` 替换为你的 GitHub 用户名），这样便于后续可能的 GitHub Pages 部署。
3.  选择 `Public`（私有仓库 Cloudflare Pages 也可以连接，但公开仓库更便于分享）。
4.  不要勾选 `Initialize this repository with a README`，因为我们将在本地初始化仓库。
5.  点击 `Create repository`。

### 推送代码到 GitHub

1.  在 Hugo 项目根目录 (`myblog`) 下，添加远程仓库地址并推送代码：
```bash
git remote add origin https://github.com/yourusername/yourrepo.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

现在，网站的源代码已经保存在 GitHub 仓库中了。

---

## 第七步：部署到 Cloudflare Pages

将网站部署到 Cloudflare Pages。

### 连接仓库

1.  登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)，进入 `Pages`。
2.  点击 `Create a project` -> `Connect to Git`。
3.  授权 Cloudflare 访问 GitHub 账户，并选择刚创建的那个仓库 (`yourrepo`)。

### 配置构建设置

Cloudflare 并不总是能自动检测到 Hugo 项目，有时需要手动选择 Hugo 项目类型然后直接下一步。

### 开始部署

点击 `Save and Deploy`。Cloudflare 会自动拉取代码，运行构建命令，并将网站部署到一个 `*.pages.dev` 的临时域名上。

### (可选) 绑定自定义域名

1.  在 Cloudflare Pages 项目控制台，点击 `Custom domains` -> `Set up a custom domain`。
2.  输入已经托管在 Cloudflare 上的域名（或者按照指引添加新域名）。
3.  这样，网站就可以通过自己的域名访问了。

---

## 总结

恭喜！现在拥有了：
1.  源代码仓库：位于 GitHub 的 `master` 分支。
2.  自动化部署：Cloudflare Pages 通过监听 GitHub 仓库实现了自动部署。

以后只需要在本地 `myblog` 文件夹里用 `hugo new` 写文章，然后 `git add .`, `git commit`, `git push`，网站就会自动更新了！享受新博客吧！