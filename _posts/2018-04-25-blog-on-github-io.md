---
layout: post
comment: true
titles:
  en: Setup your blog on github
  zh: 在github.io上建立博客
  zh-Hans: 在github.io上建立博客
  zh-Hant: 關於
key: blog-on-github-io
---
需要有一个地方存放自己的技术笔记、资料汇集、心得体会等等，研究了一下，也许使用[github Pages](https://pages.github.com/)是最为简便的方法。
# 建立github页面
第一步，首先需要有一个github账号。程序员可能很少没有github账号吧。
第二步，可以选择一个jekyll模板，从别人已有的框架开始。github推荐使用jekyll生成静态页面。
比如我从[Jekyll Themes](http://jekyllthemes.org/)上找到现在使用的模板[TeXt](https://github.com/kitian616/jekyll-TeXt-theme)。
从TeXt项目Fork，并将repository改名为\<username>.github.io
第三步，将项目同步到本地：
> git clone https://github.com/*username*/*username*.github.io

然后在本地编辑提交到github上，就可以看到你写的博客了。
# 环境配置和工具使用
## Jekyll
在上一步将项目clone到本地后，需要安装jekyll和bundle。我的工作环境是Mac OS，所以这里只讲一下Mac OS上的情况。
```
gem install jekyll bundler
brew install libxml2
brew link --force libxml2
bundle config build.nokogiri --use-system-libraries --with-xml2-include=/usr/include/libxml2/
bundle install
```
`bundle install`之前加入的那些是为了解决安装nokogiri失败的问题，可参考[这个回答](https://github.com/sparklemotion/nokogiri/issues/1483#issuecomment-252468222)。
## WebStorm
这个工具可以用来配置web项目，不是必需选择。但是既然安装了，也可以用一下。
## Markdown编辑器
我用的是[MarkEditor](http://zrey.com/app/markeditor)。用来编辑markdown文档还是不错的选择。在项目目录下创建`_posts`目录，然后就可以查看生成的页面了。
# 添加comments
在[DISQUS](https://disqus.com)上创建账号，并为你的_username_.github.io创建对应的评论站点，设置short name。然后在项目的`_config.yml`里设置disqus：
```
## Comment system (Disqus) ##
disqus:
    shortname: bitfan
```
把bitfan改成你设置的short name
（评论系统在本地看不到，需要提交之后才能看到。）

# 本地实时查看编辑的页面
到项目目录，启动本地服务器
```
bundle exec jekyll serve
```
浏览器打开<http://127.0.0.1:4000>就可以查看新写的博客了。

# 提交到github

Enjoy!
