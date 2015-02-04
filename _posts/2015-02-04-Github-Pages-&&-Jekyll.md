---
layout: post
title: Github Pages && Jekyll
categories: [blog, git, ruby, rvm, markdown, jekyll]
---

上一次我介绍了octopress的搭建，这次再介绍以下jekyll的搭建，也是目前这个博客系统正在用的。

##简介

jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如discuz。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。

Github和一些其他的应用上一次已经介绍过了，这里就不再赘述。

##部署

还是那句话，官方文档最全。[官网文档](http://jekyllrb.com/).

以下环境均以ubuntu为例。

#### 1 终端环境和git

git是必须的，直接用源安装就可以。
{% highlight bash %}
$ sudo apt-get install git
{% endhighlight %}
[git参考](http://git-scm.com/)

#### 2 安装ruby

(熟悉ruby和rails的读者,估计对环境的搭建轻车熟路了,可以直接忽略过去；非熟悉的人,这里只考虑了ruby的安装,gemset等不必深究)

* 2.1 rvm安装

jekyll的本地环境需要Ruby环境，[RVM(Ruby Version Manager)](https://rvm.io/),负责安装和管理Ruby的环境。所以我们先在终端输入如下命令，来安装RVM：(请在用户权限下操作,慎用sudo或者root权限)
{% highlight bash %}
$ curl -L get.rvm.io | bash -s stable
$ source ~/.bashrc
$ source ~/.bash_profile
{% endhighlight %}
修改RVM的Ruby安装源到国内的[淘宝镜像服务器](http://ruby.taobao.org/)，这样能提高安装速度.
{% highlight bash %}
$ sed -i -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' ~/.rvm/config/db
{% endhighlight %}

* 2.2 ruby的安装

用rvm安装ruby,并且将此版本设置成系统的默认版本.
{% highlight bash %}
rvm install 2.1.5
rvm use 2.1.5 --default
{% endhighlight %}
安装成功后,请输入`ruby -v`查看版本信息.

#### 3 创建github

* 3.1 github账号

去github注册账号，如果有的无视这一步。

* 3.2 创建代码仓库

[创建仓库](https://github.com/new)时,这个代码仓库名称中的username一定要保证你的github帐号的username一致.
然后一路默认下来,在这个仓库的`settings`中有`Automatic page generate`,让你选个主题,这里你可以一路点确定,用github给你的主题也可以，或者不选，用jekyll的定制版.

* 3.3 创建静态文件
当你的代码仓库已经建立后，可以下载到本地，用`git clone`命令。然后默认建立`index.html`文件，上传之后就会生效。
{% highlight bash %}
git clone https://github.com/username/username.github.io.git
cd username.github.io
touch index.html
echo 'Hello World!' >  index.html

#将新文件加入仓库
git add .
git commit -m 'first commit'
git push origin master
{% endhighlight %}

第一次部署可能需要半小时左右的时间，之后你可以访问你的域名了！ `http://username.github.io`，看看吧，`Hello World`应该生效了。

#### 4 安装jekyll

* 4.1 Bundler

Bundler是一个ruby插件包管理软件，非常容易管理各个依赖问题。

{% highlight bash %}
gem install bundler
{% endhighlight %}

* 4.2 安装依赖包

可以直接运行gem安装命令来安装依赖包。
{% highlight bash %}
gem install github-pages
{% endhighlight %}

或者用bundler，（这一步要等你的git仓库建立以后）在你的工程目录下面创建一个叫文件`Gemfile`，然后在此文件里面列出各项依赖
{% highlight bash %}
gem github-pages
{% endhighlight %}
最终执行`bundle install`

#### 5 配置jekyll主题

jekyll主题一般是丰富多样的，在很多网站都有，我一般去[Jekyll Themes](http://jekyllthemes.org/)。

一般拿到主题后，放入自己的工程里面（username.github.io）,然后就可以配置信息了。
jekyll的配置还是相对简化的,一般只要配置`_config.yml`就可以了,下面是配置文件列表:
{% highlight bash %}
_config.yml       # Main config (Jekyll's settings)
Rakefile          # Configs for deployment
config.rb         # Compass config
config.ru         # Rack config
{% endhighlight %}
例如我的_config.yml中就是这样的：
{% highlight bash %}
# SITE CONFIGURATION
baseurl: "" # the subpath of your site, e.g. /blog/
url: "http://rmine.github.io" # the base hostname & protocol for your site

# BUILD SETTINGS
excerpt_separator: ""
markdown: redcarpet
highlighter: pygments
redcarpet:
    extensions: ["fenced_code_blocks", "tables", "highlight", "strikethrough"]

# THEME-SPECIFIC CONFIGURATION
theme:
  # Meta
  title: RMine
  avatar: panda.jpg
  description: "RMine's blog posts, and pages." # used by search engines
  .
  .
  .
{% endhighlight %}

#### 6 配置自定义域名

在source目录下创建CNAME文件.然后把你的域名放进去:
{% highlight bash %}
echo 'your-domain.com' >> source/CNAME
# OR
echo 'www.your-domain.com' >> source/CNAME
{% endhighlight %}
然后在你自己的域名提供商后台设置到你的git域名吧.

#### 7 写博客

* 7.1 创建文件并书写

前面搞了那么多,无非就是能写博客,这里就介绍一下怎么写博客,首先你得会[markdown语法](http://wowubuntu.com/markdown/).

新建一个博客命令:
{% highlight bash %}
rake new title=["first article"]
{% endhighlight %}
运行后,会在`source/_posts`下生成一个文件（也可以手动创建）,文件名的格式为`YYYY-MM-DD-first-article.markdown`,打开这个文件会看见如下:
{% highlight bash %}
---
layout: post
title: "first article"
date: 2014-07-04 09:07:28 +0800
comments: true
categories: [blog,markdown]
---

这里就开始写你的文章啦
{% endhighlight %}

如果你的编辑器没有支持markdown的预览功能,也支持直接的页面预览,以下命令:
{% highlight bash %}
rake generate   # Generates posts and pages into the public directory
rake watch      # Watches source/ and sass/ for changes and regenerates
rake preview    # Watches, and mounts a webserver at http://localhost:4000
{% endhighlight %}
`rake generate`可以直接生成静态文件(注意_config.yml文件备份),`rake watch`可以让你知道哪些变了,`rake preview`就是启动了一个本地的web服务,运行在后台,这时候你就可以打开浏览器,输入`http://localhost:4000/`直接预览你的博客了.

如果觉得满意就提交吧。

* 7.2 提交

提交的分支这里稍微说明一下吧。
GitHub始终只会解析master分支下的内容，所以你的最终静态文件要放到master分支下，偷懒的做法就是你的开发源码分支和最终生成的静态文件全部放在master分支下（就如我）。
另外的一种做法是源码分支和静态文件分离，将开发源码放入source分支，每次jekyll build出来的文件（_sites）放入master分支，然后分别提交，此处可以用脚本完成。

另外jekyll还支持另一个保留分支`gh-pages`，这个是用来指定一个项目名称的时候生效，比如有一个项目`myjekyll`，那你的域名就是`http://username.github.io/myjekyll`，Github解析的时候就是认准你的`gh-pages`分支。

以我的系统为例，我将写完的代码提交并交给github发布：
{% highlight bash %}
git add .
git commit -m 'first commit'
git push origin master
{% endhighlight %}

整个流程就完成了，还是比较自由和简单的。

##完结

Enjoy Yourself~
