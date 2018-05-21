+++
title = "Migrate from Hexo to Hugo"
date = 2018-05-21T13:06:47+08:00
draft = false
categories = ["术业"]
tags = ["hugo", "hexo", "blog"]
+++
# 迁移
花了点时间，把 blog 从使用 Hexo 迁移到 [Hugo](https://gohugo.io/) 上了, 使用 node.js 的 Hexo 编译生成静态页面的确没有使用 Go 的 Hugo 快。而且 Hugo 现在直接支持了 Hexo 的格式，之前的所有文章都不需要修改，Hugo 直接就可以读取生成静态页面，还是方便了很多。基本上用 Hugo 新建一个 site，把 md 文件都拷贝到 content 目录下，然后修改 config.toml就可以了。

# GitHub Page HTTPS
另一个好消息是 GitHub Page 终于支持自定义域名的 HTTPS 了，之前得需要用 Cloudflare 提供的 DNS+HTTPS 服务，现在也不需要，简单不少，直接在 Settings 里设置一下就行。

# Hugo 主题
Hugo 貌似没有什么特别出彩的主题，之前用 Hexo 时的 Next 主题还是挺好的，目前暂时用了一个叫 tranquilpeak 的主题，稍微做了点修改。

# Deploy to GitHub
Hugo 没有像 Hexo 那样直接提供 deploy 的集成命令，官方文档里也是用了一个脚本来实现，也不算麻烦就是了。

# Table-of-Contents Bug
Hugo 默认的 table of contents 实现有一个小小的 bug，要求文章的标题从 H1 开始才可以正常显示目录，对于直接从 H2 或者以下开始的标题生成的样式就会很奇怪，这里是相应的 [issue](https://github.com/gohugoio/hugo/issues/1778)，可以根据 comment 里面提供的修改方法来修改一下支持从 H2-H6 开始的标题目录显示。