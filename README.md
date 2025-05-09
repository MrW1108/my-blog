# my-blog

基于 hexo 搭建的个人博客，Record All~

## hexo 使用

```bash
# 清楚构建生成的 public 目录
hexo clean
# 生成当前目录资源
hexo g
# 部署到远程
hexo d
# 本地预览
hexo s
```

### 文章编写

```bash
hexo new post <title>
```

#### 示例

```markdown
---
title: typora-vue-theme主题介绍
date: 2021-01-08 09:25:00
author: 赵奇
img: /source/images/xxx.jpg
top: true
cover: true
coverImg: /images/1.jpg
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: false
summary: 这是你自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要
categories: Markdown
tags:
  - Typora
  - Markdown
---
```

| 配置选项      | 默认值                         | 描述                                                                                                                                                                                       |
| ------------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| title         | `Markdown` 的文件标题          | 文章标题，强烈建议填写此选项                                                                                                                                                               |
| date          | 文件创建时的日期时间           | 发布时间，强烈建议填写此选项，且最好保证全局唯一                                                                                                                                           |
| author        | 根 `_config.yml` 中的 `author` | 文章作者                                                                                                                                                                                   |
| img           | `featureImages` 中的某个值     | 文章特征图，推荐使用 oss(腾讯云、阿里云等)来做图片的路径.如: `http://xxx.com/xxx.jpg`                                                                                                      |
| top           | `true`                         | 推荐文章（文章是否置顶），如果 `top` 值为 `true`，则会作为首页推荐文章                                                                                                                     |
| cover         | `false`                        | `v1.0.2`版本新增，表示该文章是否需要加入到首页轮播封面中                                                                                                                                   |
| coverImg      | 无                             | `v1.0.2`版本新增，表示该文章在首页轮播封面需要显示的图片路径，如果没有，则默认使用文章的特色图片                                                                                           |
| password      | 无                             | 文章阅读密码，如果要对文章设置阅读验证密码的话，就可以设置 `password` 的值，该值必须是用 `SHA256` 加密后的密码，防止被他人识破。前提是在主题的 `config.yml` 中激活了 `verifyPassword` 选项 |
| toc           | `true`                         | 是否开启 TOC，可以针对某篇文章单独关闭 TOC 的功能。前提是在主题的 `config.yml` 中激活了 `toc` 选项                                                                                         |
| mathjax       | `false`                        | 是否开启数学公式支持 ，本文章是否开启 `mathjax`，且需要在主题的 `_config.yml` 文件中也需要开启才行                                                                                         |
| summary       | 无                             | 文章摘要，自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要                                                                     |
| categories    | 无                             | 文章分类，本主题的分类表示宏观上大的分类，只建议一篇文章一个分类                                                                                                                           |
| tags          | 无                             | 文章标签，一篇文章可以多个标签                                                                                                                                                             |
| keywords      | 文章标题                       | 文章关键字，SEO 时需要                                                                                                                                                                     |
| reprintPolicy | cc_by                          | 文章转载规则， 可以是 cc_by, cc_by_nd, cc_by_sa, cc_by_nc, cc_by_nc_nd, cc_by_nc_sa, cc0, noreprint 或 pay 中的一个                                                                        |

## 部署

通过 github ci 自动部署到 [github pages](https://mrw1108.github.io)

## 感谢

感觉开源的 hexo 主题 [hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery)

感谢[hexo-matery-modified](https://github.com/godweiyang/hexo-matery-modified)
