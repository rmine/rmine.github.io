---
layout: post
title: Codeblock Style
categories: [highlight, jekyll, pygments]
---

之前默认的代码块样式实在是不堪入目，颜色排版等问题突出，所以要好好捣鼓一下。

采用的是pygments，大名鼎鼎了，有名气用起来就有底气，重要的是支持的色彩搭配确实很赞。

<h3>1.安装</h3>
首先本地先安装好支持库，然后开发编辑的时候，就可以选各种主题了。
{% highlight bash linenos=table%}
sudo apt-get install python-pygments
{% endhighlight %}
如果你没有装redcarpet的gem包，请安装:
{% highlight bash linenos=table%}
sudo gem install redcarpet
{% endhighlight %}


<h3>2.配置</h3>
在`_config.yml`中配置如下：
{% highlight bash linenos=table%}
markdown: redcarpet
highlighter: pygments
redcarpet:
    extensions: ["fenced_code_blocks", "tables", "highlight", "strikethrough"]
{% endhighlight %}

<h3>3.生成样式文件并加入你的网站引用</h3>
{% highlight bash linenos=table%}
pygmentize -S default -f html > your/path/pygments.css
{% endhighlight %}
主题有很多，['monokai', 'manni', 'rrt', 'perldoc', 'borland', 'colorful', 'default', 'murphy', 'vs', 'trac', 'tango', 'fruity', 'autumn', 'bw', 'emacs', 'vim', 'pastie', 'friendly', 'native']
{% highlight bash linenos=table%}
# -S 参数后面就是主题名称
pygmentize -S monokai -f html > your/path/pygments.css
{% endhighlight %}
生成的pygments.css文件或者内容自己在网站头部引用：
{% highlight html linenos=table%}
<link rel="stylesheet" href="/your/path/pygments.css">
{% endhighlight %}

<h3>4.jekyll代码块高亮</h3>
代码块放在两个标签之间，` highlight c linenos=true` 和 `endhighlight `，其中的linenos是行号显示，加上table值可以copy代码的时候忽略行号。


<h3>5.打完收工</h3>
以下是展示，可能c的高亮支持不太好，以后可以继续优化。
{% highlight c linenos=table%}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "base64.h"
#include "config.h"
#include "cmdline.h"
#include "proxytunnel.h"
#include "basicauth.h"

/*
 * Create the HTTP basic authentication cookie for use by the proxy. Result
 * is stored in basicauth.
 */
char *basicauth(char *user, char *pass) {
	char *b64str = malloc(80);

	int len = strlen( user ) + strlen( pass ) + 2;
	char *p = (char *) malloc( len );

	/* Set up the cookie in clear text */
	sprintf( p, "%s:%s", user, pass );

	/*
	 * Base64 encode the clear text cookie to create the HTTP base64
	 * authentication cookie
	 */
	base64( (unsigned char *)b64str, (unsigned char *)p, strlen(p) );

//	if( args_info.verbose_flag ) {
//		message( "Proxy basic auth of %s is %s\n", p, basicauth );
//	}

	free( p );

	return b64str;
}
{% endhighlight %}


{% highlight javascript linenos=table%}
var work = function()
{
    console.log("working hard");
}

var doWork = function(f)
{
    console.log("starting");

//    try
//    {
//        f();
//    }
    catch(ex)
    {
        console.log(ex);
    }

    console.log("ending");
}

doWork(work);
{% endhighlight %}


{% highlight ruby linenos=table%}
require 'rubygems'
require 'gauntlet'

##
# GemGauntlet validates all current gems. Currently these packages are
# borked:
#
# Asami-0.04           : No such file or directory - bin/Asami.rb
# ObjectGraph-1.0.1    : No such file or directory - bin/objectgraph
# evil-ruby-0.1.0      : authors must be Array of Strings
# fresh_cookies-1.0.0  : authors must be Array of Strings
# plugems_deploy-0.2.0 : authors must be Array of Strings
# pmsrb-0.2.0          : authors must be Array of Strings
# pqa-1.6              : authors must be Array of Strings
# rant-0.5.7           : authors must be Array of Strings
# rvsh-0.4.5           : No such file or directory - bin/rvsh
# xen-0.1.2.1          : authors must be Array of Strings

class GemGauntlet < Gauntlet # :nodoc:
  def run(name)
    warn name

    spec = begin
             Gem::Specification.load 'gemspec'
           rescue SyntaxError
             Gem::Specification.from_yaml File.read('gemspec')
           end
    spec.validate

    self.data[name] = false
    self.dirty = true
  rescue SystemCallError, Gem::InvalidSpecificationException => e
    self.data[name] = e.message
    self.dirty = true
  end

  def should_skip?(name)
    self.data[name] == false
  end

  def report
    self.data.sort.reject { |k,v| !v }.each do |k,v|
      puts "%-21s: %s" % [k, v]
    end
  end
end

gauntlet = GemGauntlet.new
gauntlet.run_the_gauntlet ARGV.shift
gauntlet.report

{% endhighlight %}