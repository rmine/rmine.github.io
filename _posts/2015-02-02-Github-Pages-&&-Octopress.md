---
layout: post
title: Github Pages && Octopress
categories: [blog, git, ruby, rvm, markdown]
---
##简介

Octopress是利用Jekyll博客引擎开发的一个博客系统，号称是hacker专属的一个博客系统，就像官网自己描述的一样 ---A blogging framework for hackers.

Octopress是个静态博客，就是一些静态的html css js等展示，所以不像一些有后台等支撑的博客，复杂而臃肿，而且静态网站的访问速度也是众所周知的.

Octopress可以支持3种部署方式:

- github
- heroku
- Rsync

github对于程序员来说,再熟悉不过了,基于git的全球最大的社交编程及代码托管网站,支持全方位的服务,此博客就是基于github page功能的展现,当然,之所以选它,除了方便强大的功能支持外,良好的网络访问是关键；heroku是一个非常强大的云平台,支持多种语言的部署,之所以国人不太熟悉,使用的人不多,就是因为网络访问过慢,国内没有相应的服务器,我不用它也是因为这个,但是功能还是很强大的,收费偏高,有兴趣的可以去玩玩看；Rsync模式就是自给自足模式了,一台服务器,把写好的文件上传到服务器,有个配置好的web server(nginx,apache等),自己配置转发,没有资源的同学就不要考虑这种了.

本文以自身为例，介绍如何一步步搭建一套基于Octopress的github的博客系统,读者需要熟悉一些shell命令,基本的git操作,ruby的一些相关知识,和markdown语法.__如果觉得这些知识让你听起来一团乱麻,那么兄弟,你还是不要跳进这个坑为妙__.

##部署

先插一句,"所有方案都比不上官方文档",如果有耐心并且英语还可以的同学,不妨好好参考[官网文档](http://octopress.org/docs).

#### 1 终端环境和git

mac和linux下都有终端,在终端下运行和安装软件都非常便捷,windows比较麻烦,不过有个git bash的工具可以模拟Unix*环境,也支持了git环境,mac已经自带了git,而linux(这里统一按ubuntu为例,mac下的brew也是很方便就不介绍了)安装也非常方便.
{% highlight bash %}
$ sudo apt-get install git
{% endhighlight %}
[git参考](http://git-scm.com/)

#### 2 安装ruby

(熟悉ruby和rails的读者,估计对环境的搭建轻车熟路了,可以直接忽略过去；非熟悉的人,这里只考虑了ruby的安装,gemset等不必深究)

* 2.1 rvm安装

Octopress需要Ruby环境，[RVM(Ruby Version Manager)](https://rvm.io/),负责安装和管理Ruby的环境。所以我们先在终端输入如下命令，来安装RVM：(请在用户权限下操作,慎用sudo或者root权限)
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

安装2.1.1版本,并且将此版本设置成系统的默认版本.
{% highlight bash %}
rvm install 2.1.1
rvm use 2.1.1 --default
{% endhighlight %}
安装成功后,请输入`ruby -v`查看版本信息.

#### 3 安装Octopress

* 3.1 下载源代码

{% highlight bash %}
git clone git://github.com/imathis/octopress.git octopress
cd octopress
{% endhighlight %}

* 3.2 安装依赖包

{% highlight bash %}
gem install bundler
bundle install
{% endhighlight %}

* 3.3 安装Octopress默认主题

{% highlight bash %}
rake install
{% endhighlight %}

#### 4 配置github page(User/Organization pages)

来到[github pages](https://pages.github.com/),根据提示创建,没有github帐号的自己建个.

* 4.1 创建一个代码仓库

[创建仓库](https://github.com/new)时,这个代码仓库名称中的username一定要保证你的github帐号的username一致.
然后一路默认下来,在这个仓库的`settings`中有`Automatic page generate`,让你选个主题,这里你可以一路点确定,因为后面会有Octopress的主题,所以会覆盖这里的主题.

* 4.2 Octopress关联到github page

这里比较关键,就是将2个系统结合起来,Octopress已经为我们做好了.回到octopress目录,输入以下命令:
{% highlight bash %}
rake setup_github_pages
{% endhighlight %}
输入后会有个输入,填上你刚创建的github上的代码仓库路径(e.g. git@github.com:username/username.github.io.git或者https://github.com/username/username.github.io.git).
这时候会把你的git路径从octopress转到github page,分支切换到了source,_deploy被放到了master分支,这里就是github部署的分支.

接着输入
{% highlight bash %}
rake generate
rake deploy
{% endhighlight %}
就会把最新的代码提交到github上,能直接展示了,[username].github.io,欣赏你自己的网站吧(当然还没完,最初的样式确实不满意).

_注意:如果这里`rake deploy`发生错误,应该是分支代码没有同步的关系,请进入`_deploy/`目录下,将它里面的东西提交到github上去,如下:_
{% highlight bash %}
git add .
git commit -m 'init deploy'
git push origin master
{% endhighlight %}

接着把你工程里的资源文件也备份到github上去吧
{% highlight bash %}
git add .
git commit -m 'your message'
git push origin source
{% endhighlight %}

* 4.3 自定义域名

在source目录下创建CNAME文件.然后把你的域名放进去:
{% highlight bash %}
echo 'your-domain.com' >> source/CNAME
# OR
echo 'www.your-domain.com' >> source/CNAME
{% endhighlight %}
然后在你自己的域名提供商后台设置到你的git域名吧.


#### 5 配置Octopress

Octopress的配置还是相对简化的,一般只要配置`_config.yml`就可以了,下面是配置文件列表:
{% highlight bash %}
_config.yml       # Main config (Jekyll's settings)
Rakefile          # Configs for deployment
config.rb         # Compass config
config.ru         # Rack config
{% endhighlight %}

#### 6 写博客

前面搞了那么多,无非就是能写博客,这里就介绍一下怎么写博客,首先你得会[markdown语法](http://wowubuntu.com/markdown/).

新建一个博客命令:
{% highlight bash %}
rake new_post["first article"]
{% endhighlight %}
运行后,会在`source/_posts`下生成一个文件,文件名的格式为`YYYY-MM-DD-first-article.markdown`,打开这个文件会看见如下:
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

如果你的编辑器没有支持markdown的预览功能,Octopress也支持直接的页面预览,以下命令:
{% highlight bash %}
rake generate   # Generates posts and pages into the public directory
rake watch      # Watches source/ and sass/ for changes and regenerates
rake preview    # Watches, and mounts a webserver at http://localhost:4000
{% endhighlight %}
`rake generate`可以直接生成静态文件(注意_config.yml文件备份),`rake watch`可以让你知道哪些变了,`rake preview`就是启动了一个本地的web服务,运行在后台,这时候你就可以打开浏览器,输入`http://localhost:4000/`直接预览你的博客了.

如果觉得满意就提交吧,还记得那个命令吧,`rake deploy`, 记得别忘了把最新的代码push上去哦.

#### 7 主题和样式

虽然致力于成为一个full stack developer,但是不得不承认,css是我的短板,但是有Octopress主题安装,我也就淡然了(Y(^_^)Y).

主题安装很简单,以我为例,我找了一个[Greyshade主题],然后安装
{% highlight bash %}
$ git clone git@github.com:shashankmehta/greyshade.git .themes/greyshade
$ echo "\$greyshade: color;" >> sass/custom/_colors.scss //Substitue 'color' with your highlight color
$ rake "install[greyshade]"
$ rake generate
{% endhighlight %}
然后...然后就OK啦,很无脑,很安逸.

至于那些css大牛,你们完全可以定制自己的样式,请参考官方的样式[定制文档](http://octopress.org/docs/theme/styles/).

##完结

差不多就全部介绍完了,可能有不少问题我没有面面具到,比如git可能出现各种各样的问题,比如这个主题其实有些bug的,比如...但是一步步解决这些问题不就是我们折腾它的理由吗?发现问题,思考问题,解决问题,这就是我们工作和生活中最直接的表达.

Enjoy Yourself~

-------------

#### 后记

这篇文章是很久前写的了，现在翻出来给人参考用的，至于现在嘛，嘿嘿，我已经用jekyll了，最大的理由就是主题多！下回可以介绍下jekyll，流程和octopress是非常相似的。现在的这个jekyll主题，对于代码块的样式实在惨不忍睹，这几天会处理一下。