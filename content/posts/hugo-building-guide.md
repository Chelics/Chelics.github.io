---
title: "如何利用Hugo搭建个人博客"
subtitle: ""
date: 2025-12-21T19:36:34+08:00
lastmod: 2025-12-21T19:36:34+08:00
slug: "hugo-building-guide"
author: "Chelics"
description: "搭建过程记录与心得"
categories: ["工程实践"]
tags: ["Hugo", "Github Actions"]
comment: true
toc: true
draft: false
---
为了搭建个人主页，我评估了目前主流的几种方案。

考虑到第三方平台在内容管理上的局限性，以及部分静态框架（如 Hexo）在编译速度和依赖维护上的成本, 经过对比，本人最终选择利用 Hugo 来搭建博客系统。

## 主要流程

### 1. 环境准备

在 Windows 环境下，由于 FixIt 主题依赖 SCSS 编译，必须安装 Extended 版本的 Hugo。

```
# 使用 winget 安装扩展版 Hugo
winget install Hugo.Hugo.Extended

# 验证安装是否成功（必须包含 +extended 标识）
hugo version
```

### 2. 项目初始化与主题集成

其实在选择主题的时候, 我也有看到很多大佬个性化的主题, 在Hugo的基础上改得非常好看, 但是为了掌控站点架构, 我还是选择从零构建项目骨架。先做起来, 等到后面有需要的时候, 再对样式进行美化。

```
# 1. 创建站点
hugo new site my-blog --format toml
cd my-blog

# 2. 初始化 Git 并以子模块形式引入 FixIt 主题
git init
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
```

---

关于我为什么会用Git子模块:

A. 源码解耦与平滑升级

- 独立维护：可以随时通过 `git submodule update --remote` 获取主题作者的 Bug 修复或新功能，而不会与自己的站点逻辑产生冲突。

B. 遵循 Hugo 的“查寻顺序”进行定制

- Hugo 拥有一套优雅的 Lookup Order（查寻顺序）。如果想修改主题的某个样式或页面模板，不需要修改 themes/ 下的任何文件。只需要在根目录的 layouts/ 或 assets/ 下创建同名文件，Hugo 就会优先使用根目录的文件，而不是主题的文件。

C. 保持项目骨架的“轻量化”

- 符合我“先做起来，后续美化”的迭代思路。

---

### 3. 核心配置修改

按需配置 `hugo.html`文件

```
baseURL = 'https://your-username.github.io/'
languageCode = 'zh-cn'
title = 'title-example'
theme = 'FixIt'

# 开启中文增强支持（优化搜索和字数统计）
hasCJKLanguage = true
```

### 4. 内容管理：核心操作命令

在内容创作阶段，常用的命令如下：

#### 新建页面

```
# 新建第一篇博文
hugo new posts/first-blog.md

# 新建关于页面
hugo new about/index.md
```

#### 本地调试预览

```
# -D 参数用于预览包含 draft = true 的草稿
hugo server -D
```

### 5. 自动化流水线：GitHub Actions

为了实现“本地写稿，一推即发”，我配置了 CI/CD 流程。

在 `.github/workflows/` 下创建 `deploy.yaml` 文件：

```
name: Deploy Hugo site to Pages
on:
  push:
    branches: ["main"]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive # 关键：确保自动拉取主题子模块
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true # 必须使用 extended 编译 SCSS
      - name: Build
        run: hugo --minify
      # ... 部署至 GitHub Pages 的后续步骤
```

### 6. Git 协作

在以上的步骤中, 我们只是在本地搭建, 还没有与GitHub的个人主页产生连接。

首先, 要新建一个public仓库, 名称必须为 `your-name.github.io`.

然后在本地项目目录下:

```
# 关联远程 GitHub 仓库
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
git branch -M main

# 标准提交工作流
git add .
git commit -m "xxxxx"
git push -u origin main
```

### 7. 规范化提交

为了规范化仓库, 我设计了一套commit编写规则。

| 类型               | 含义      | 适用场景                                                          |
| :----------------- | :-------- | :---------------------------------------------------------------- |
| **feat**     | 新功能    | 网站逻辑/功能的增加（如：新增评论插件、搜索功能、侧边栏小组件）   |
| **content**  | 内容更新  | 发布新文章、修改文章错别字、更新文章分类或标签                    |
| **fix**      | 修复      | 修复网站模板 Bug、修复 CSS 布局错乱、修复 404 链接                |
| **docs**     | 项目文档  | 修改 `README.md`、项目说明文档、部署教程（不涉及博客页面内容）  |
| **style**    | 代码格式  | 仅指代码缩进、删除空格等不改变逻辑的格式化操作（不用于 UI 修改）  |
| **refactor** | 重构      | 修改代码结构，既不修复 Bug 也不新增功能（如：优化 Hugo 模板代码） |
| **chore**    | 任务/杂事 | 更新依赖（hugo 升级）、配置 GitHub Actions、修改 `.gitignore`   |

## 避坑小结

About 页空白问题：起初误用了 _index.md，导致 Hugo 将其识别为列表页（Branch Node）而不渲染正文。将其重命名为 index.md（Leaf Node）后成功修复。

## 结语

博客的搭建只是开始，内容的持续输出才是内核。

希望能帮到同样在路上的开发者。
