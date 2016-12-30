---
title: 优雅地使用 Hexo 
date: 2016-12-30 19:53:28
categories: 技术
tags:
---

介绍一下我是怎么使用 Hexo 的。

我希望它具有这样的特性：

+ 快速部署

  根据 Git 的特性，尽量让纳入版本控制的文件小而多，只包含源码。

+ 自动部署

  不需要手动使用 `hexo deploy` 进行部署。只需要将源码传到 GitHub 上，它就能自动部署。

+ 免安装

  在没有 node.js 环境与 hexo 命令行工具，甚至没有 Git 时，也能编写/发布博客。


<!--more-->

# 搭建博客

接下来介绍如何从零开始，无须配置环境，纯靠浏览器搭建你自己的博客。

1. 注册一个 [GitHub](https://github.com) 账号。

2. 注册一个 [Travis CI](https://travis-ci.org/) 账号，与刚刚注册的 GitHub 账号关联。

3. 在 GitHub 上新建一个 Repository，名为 `<username>.github.io` ，如 `zccz14.github.io` 。

4. 再新建一个 Repository，名字随意，如 `blog`。

   注意要勾选 Initialize this repository with a README 来进行初始化。

5. 在 Travis CI 中添加对 `blog` 这个 repository 的关联。

6. 在 Travis CI 中配置这个 `blog` (找到 Settings)。

   1. 确保 `Build only if .travis.yml is present`, `Build pushes`, `Build pull requests` 处于打开状态。

   2. 添加环境变量

      + GITHUB_USERNAME: GitHub 的用户名

      + GITHUB_PASSWORD: GitHub 的密码

        确保此项的 `Display value in build log` 为关闭状态，否则你的密码可能会泄露。

      + GITHUB_EMAIL: GitHub 的邮箱

        如果能看懂接下来的配置脚本的话，可以更改此处邮箱的含义；否则，按照我推荐的来填。

      + THEME_URI: 主题的URI

        Hexo Theme 的 URI，如果你不知道应该填什么，就填一个`https://github.com/iissnan/hexo-theme-next.git`

      + THEME_NAME: 主题名

        Hexo Theme 的主题名，如果你不知道应该填什么，就填一个`next` 

7. 回到 GitHub 的 blog，创建文件 `package.json`：

   ```json
   {
     "name": "hexo-site",
     "version": "0.0.0",
     "private": true,
     "hexo": {
       "version": "3.2.2"
     },
     "dependencies": {
       "hexo": "^3.2.0",
       "hexo-generator-archive": "^0.1.4",
       "hexo-generator-category": "^0.1.3",
       "hexo-generator-index": "^0.2.0",
       "hexo-generator-tag": "^0.2.0",
       "hexo-renderer-ejs": "^0.2.0",
       "hexo-renderer-stylus": "^0.3.1",
       "hexo-renderer-marked": "^0.2.10",
       "hexo-server": "^0.2.0"
     }
   }
   ```

   如果日后要添加新的依赖的话直接修改这个文件即可。

8. 继续创建一个`_config.yml` 的 Hexo 配置文件：

   ```yaml
   # Hexo Configuration
   ## Docs: https://hexo.io/docs/configuration.html
   ## Source: https://github.com/hexojs/hexo/

   # Site
   title: Hexo
   subtitle:
   description:
   author: John Doe
   language:
   timezone:

   # URL
   ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
   url: http://yoursite.com
   root: /
   permalink: :year/:month/:day/:title/
   permalink_defaults:

   # Directory
   source_dir: source
   public_dir: public
   tag_dir: tags
   archive_dir: archives
   category_dir: categories
   code_dir: downloads/code
   i18n_dir: :lang
   skip_render:

   # Writing
   new_post_name: :title.md # File name of new posts
   default_layout: post
   titlecase: false # Transform title into titlecase
   external_link: true # Open external links in new tab
   filename_case: 0
   render_drafts: false
   post_asset_folder: false
   relative_link: false
   future: true
   highlight:
     enable: true
     line_number: true
     auto_detect: false
     tab_replace:

   # Category & Tag
   default_category: uncategorized
   category_map:
   tag_map:

   # Date / Time format
   ## Hexo uses Moment.js to parse and display date
   ## You can customize the date format as defined in
   ## http://momentjs.com/docs/#/displaying/format/
   date_format: YYYY-MM-DD
   time_format: HH:mm:ss

   # Pagination
   ## Set per_page to 0 to disable pagination
   per_page: 10
   pagination_dir: page

   # Extensions
   ## Plugins: https://hexo.io/plugins/
   ## Themes: https://hexo.io/themes/
   theme: landscape

   # Deployment
   ## Docs: https://hexo.io/docs/deployment.html
   deploy:
     type:

   ```

   这是利用 `hexo init` 生成的配置，先贴上，之后按照自己的情况改。

   其实有些项是没有用的，比如 deploy 项。

9. 创建一个文件：`source/_posts/hello-world.md` ：

   ```markdown
   ---
   title: Hello World
   ---
   Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

   ```

   这个也是官方生成的文件的节选。

   当然你也可以自己设定文件名与内容，`source/_posts` 目录下的Markdown文件都会被渲染成文章。

   如果你想发博客的话，就在`source/_posts` 下创建新的Markdown文件。

   > 实际上 `source` 目录与`public`(生成的目录)直接对应。如果你在 `source` 目录下创建一个图片 `1.jpg`，则在`public`中也有一个 `1.jpg`。

10. 最后，新建一个 `.travis.yml` ，并写入：

   ```yaml
   language: node_js
   node_js: 
       - "6"
   before_install:
       - git config --global user.email $GITHUB_EMAIL
       - git config --global user.name $GITHUB_USERNAME
       - npm i hexo-cli -g
       - git clone $THEME_URI themes/$THEME_NAME
   script:
       - hexo config theme $THEME_NAME
       - hexo generate
   after_success:
       - cd public
       - git init
       - git add .
       - git commit -m "Travis Deploy"
       - git push -f -q https://$GITHUB_USERNAME:$GITHUB_PASSWORD@github.com/$GITHUB_USERNAME/$GITHUB_USERNAME.github.io master
   ```

   然后你可以上 Travis CI 看它构建的过程，它会在最后将构建出来的东西推送到 `<username>.github.io` 那个Repository上，GitHub Pages服务会提供给你一个子域名：`<username>.github.io` 。

   然后你就可以访问你的博客了：`https://<username>.github.io`。

11. Have fun with GitHub & Travis CI & Hexo!


## 自定义的域名

创建 `source/CNAME` ，在其中写入一行，如我的域名是 `zccz14.com`

那么 CNAME 文件：

```
zccz14.com
```

就这样一行。

然后 GitHub 就会将之前的 `<username>.github.io` 重定向到这个域名上。

当然，你也需要在你的域名提供商那里创建规则，将请求导向 GitHub Pages 服务。

## 分离主题

在这个框架下，我们希望博客的主题与博客本身分离，保持版本控制的相对独立。

+ 如果你还没有主题，请去选择一个主题，找到它们的 Repo URI。

  比如我用的主题就是基于 [next](https://github.com/iissnan/hexo-theme-next) 主题，然后自己再魔改一番形成的。

  首先你需要 fork 它，在你的用户下保存它的一个副本，方便日后修改主题。

  为什么要 fork 而不是硬拷贝一份呢？原因有两个：

  1. 尊重原主题作者的知识产权。
  2. 当原主题更新时，你可以简单地通过合并分支来更新你的主题。

+ 如果你已经有了主题，并且没有分离到另外一个 Repository，请将它分离到另外一个 Repo 里。

  我猜你并不是自己创建了一个新主题。如果你的主题不是通过 fork 得来的话：

  我建议重新 fork 你的源主题，然后将你的改动重新做到对应的 Repo 里。

有了主题的 URI 以后，直接到 Travis CI 里修改对应 Repository 的 Settings 中的环境变量 `THEME_URI` 即可。至于这个 `THEME_NAME` 要不要改其实基本没有什么区别。

