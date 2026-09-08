---
title: 博客搭建完成：Hexo + Fluid + GitHub Pages
date: 2026-09-08
categories:
  - 工具
tags:
  - Hexo
  - GitHub Pages
---

欢迎来到我的博客！这篇文章既是第一篇博文，也是一份写作模板示例。

<!-- more -->

## 为什么写博客

- 费曼学习法：把知识讲清楚，才算真正掌握
- 沉淀踩坑记录，方便日后查阅
- 建立个人技术影响力

## 本站技术栈

| 组件 | 说明 |
|------|------|
| Hexo | 静态博客框架 |
| Fluid | 简洁优雅的主题，支持目录导航 |
| GitHub Pages | 免费静态托管 |
| GitHub Actions | 推送后自动构建部署 |

## 写作流程

日常写作只需三步：

```bash
# 1. 新建文章
npx hexo new "文章标题"

# 2. 本地预览
npx hexo server

# 3. 提交推送，GitHub Actions 自动部署
git add . && git commit -m "post: 文章标题" && git push
```

## 代码高亮示例

```javascript
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet('World'));
```

## Markdown 提示块

> 提示：在文章中使用 `<!-- more -->` 标记摘要截断点，首页只会显示标记之前的内容。

祝写作愉快！
