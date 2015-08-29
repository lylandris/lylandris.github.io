---
layout: post
title:  "在阿里云搭建 Jekyll 环境"
date:   2015-08-29 18:56:10
categories: 技术日志
---

考虑到 GitHub Page 需要 Jekyll 环境，而使用的笔记本没权限安装 Linux 或者虚拟机，因此就在阿里云的上海节点搞了一台 ECS，打算以后在这个机器上处理写好的日志。
不过在搭建过程中，也遇到了不少问题。

参考了先驱者 [jibing57 的 blog](http://www.jibing57.com/blog/2015/03/19/github-pages.html)，其第一句话“Github Pages搭建参见官方说明 <https://pages.github.com/>, 简洁明了，一目了然。”就让我折腾了半天。

具体是这样的，在 ECS 上，我选择的系统是 `CentOS 7.0 x86_64` ，在使用 `gem install bundler` 安装的时候，出现了下面的报错信息。

    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ gem install bundler
    ERROR:  Could not find a valid gem 'bundler' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/latest_specs.4.8.gz)

基于经验，任何和网络相关的不正常行为都要怀疑“神秘人（You-Known-Who）”。因此立即 Google 了一下，发现 [ToDoToTry 的一篇文章](http://www.cnblogs.com/ToDoToTry/p/4422454.html)提到了这个问题。在文章的介绍中，找到了一个淘宝网搭建的 [Ruby 镜像站](https://ruby.taobao.org)。按照镜像的说明修改后，完成 `gem install bundler`。

接下来在 `bundle install` 的时候又停在了下面的错误上。

    Query List: []
    Resolving dependencies....
    Installing RedCloth 4.2.9 with native extensions
    Building native extensions.  This could take a while...

    Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.

        /usr/bin/ruby extconf.rb
    mkmf.rb can't find header files for ruby at /usr/share/include/ruby.h


    Gem files will remain installed in /home/jiangz/.gem/ruby/gems/RedCloth-4.2.9 for inspection.
    Results logged to /home/jiangz/.gem/ruby/gems/RedCloth-4.2.9/ext/redcloth_scan/gem_make.out

搜索了好几个地方，有几个方向的可能。其一可能是没有 `gcc` 编译器（[页一](http://www.bytecho.com/2014/04/16/install-jekyll-on-mavericks)），基于这个猜测，我安装了 `clang` 或 `gcc`，问题没有得到解决，同时在这篇文章中引用了 [Stack Overflow](http://stackoverflow.com/questions/22846685/clang-build-error-when-trying-to-install-jekyll) 的一篇问答，期望将报错信息变为警告信息使其强行通过，试下来并没效果，同时这个方法也不够优雅，对于这个解答，[知乎](http://www.zhihu.com/question/26096289)上也有同样的疑惑。再次更换关键字搜索，在 [Segment Fault](http://segmentfault.com/q/1010000000392817) 站点的文章和另一篇 [Stack Overflow](http://stackoverflow.com/questions/4393189/failing-installing-pg-gem-mkmf-rb-cant-find-header-files-for-ruby-mac-osx-1) 的问答，推测出一个原因，那就是没有安装 `ruby-devel` 软件包。立即执行 `sudo yum install ruby-devel` 安装该软件包后，尝试执行 `bundle install -V` ，顺利通过过了刚才的坎，进入下一个报错信息。

    Building native extensions.  This could take a while...

    Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.

        /usr/bin/ruby extconf.rb
    checking if the C compiler accepts ... yes
    Building nokogiri using packaged libraries.
    checking for gzdopen() in -lz... no
    zlib is missing; necessary for building libxml2
    *** extconf.rb failed ***
    Could not create Makefile due to some reason, probably lack of necessary
    libraries and/or headers.  Check the mkmf.log file for more details.  You may
    need configuration options.

有了之前的经验，还是猜测是否是什么软件包漏装了，尝试搜索了一下软件库，发现 `zlib-devel` 开发库软件包后，立即装上。

    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ sudo yum search zlib-dev
    Loaded plugins: langpacks
    ================================================ N/S matched: zlib-dev =================================================
    ghc-zlib-devel.x86_64 : Haskell zlib library development files
    zlib-devel.i686 : Header files and libraries for Zlib development
    zlib-devel.x86_64 : Header files and libraries for Zlib development

      Name and summary matches only, use "search all" for everything.
    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ sudo yum install zlib-devel

安装 `zlib-devel` 软件包后，再次执行 `bundle install -V` ，终于看到了下面的信息。

    Using bundler 1.10.6
    Bundle complete! 1 Gemfile dependency, 58 gems now installed.
    Use `bundle show [gemname]` to see where a bundled gem is installed.
    Post-install message from html-pipeline:
    -------------------------------------------------
    Thank you for installing html-pipeline!
    You must bundle Filter gem dependencies.
    See html-pipeline README.md for more details.
    https://github.com/jch/html-pipeline#dependencies
    -------------------------------------------------

至此，jibing57 的第一句话算是告于段落。

写了个简单的 Hello World 日志，按照说明运行 `bundle exec jekyll serve`，再一次出现了错误信息。

    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ bundle exec jekyll serve
    /home/jiangz/.gem/ruby/gems/execjs-2.6.0/lib/execjs/runtimes.rb:48:in `autodetect': Could not find a JavaScript runtime. See https://github.com/rails/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)
            from /home/jiangz/.gem/ruby/gems/execjs-2.6.0/lib/execjs.rb:5:in `<module:ExecJS>'
            from /home/jiangz/.gem/ruby/gems/execjs-2.6.0/lib/execjs.rb:4:in `<top (required)>'
            from /home/jiangz/.gem/ruby/gems/coffee-script-2.4.1/lib/coffee_script.rb:1:in `require'
            from /home/jiangz/.gem/ruby/gems/coffee-script-2.4.1/lib/coffee_script.rb:1:in `<top (required)>'
            from /home/jiangz/.gem/ruby/gems/coffee-script-2.4.1/lib/coffee-script.rb:1:in `require'
            from /home/jiangz/.gem/ruby/gems/coffee-script-2.4.1/lib/coffee-script.rb:1:in `<top (required)>'
            from /home/jiangz/.gem/ruby/gems/jekyll-coffeescript-1.0.1/lib/jekyll-coffeescript.rb:2:in `require'
            from /home/jiangz/.gem/ruby/gems/jekyll-coffeescript-1.0.1/lib/jekyll-coffeescript.rb:2:in `<top (required)>'
            from /home/jiangz/.gem/ruby/gems/jekyll-2.4.0/lib/jekyll/deprecator.rb:46:in `require'
            from /home/jiangz/.gem/ruby/gems/jekyll-2.4.0/lib/jekyll/deprecator.rb:46:in `block in gracefully_require'
            from /home/jiangz/.gem/ruby/gems/jekyll-2.4.0/lib/jekyll/deprecator.rb:44:in `each'
            from /home/jiangz/.gem/ruby/gems/jekyll-2.4.0/lib/jekyll/deprecator.rb:44:in `gracefully_require'
            from /home/jiangz/.gem/ruby/gems/jekyll-2.4.0/lib/jekyll.rb:141:in `<top (required)>'
            from /home/jiangz/.gem/ruby/gems/jekyll-2.4.0/bin/jekyll:6:in `require'
            from /home/jiangz/.gem/ruby/gems/jekyll-2.4.0/bin/jekyll:6:in `<top (required)>'
            from /home/jiangz/bin/jekyll:23:in `load'
            from /home/jiangz/bin/jekyll:23:in `<main>'

从字面意思看到似乎是没有 Java Script 引擎，完全不碰前端的我不知道什么引擎可用，继续 Google 找到了 [一篇问答](https://ruby-china.org/topics/692) ，按照这篇文章的方案，我直接暴力地直接装上了 **Node.js** 。对于这个名字，平时经常听说，但始终不知道也没去怎么了解这是什么东东，这次算是顺便搜索知道它是 *基于 Chrome JavaScript 运行环境，用于便捷地搭建快速、可扩展的网络应用*。

本来想跟着 jibing57 的手册尝试其主题，但是想想没明白 jekyll 的运行机制，就开始阅读 [jekyll 的手册](https://jekyllrb.com)了。按照手册的[快速入门](https://jekyllrb.com/docs/quickstart/)，首先在一个空的目录下执行 `jekyll new .` ，然后将创建的文件移动到版本库的根目录。这么做的原因是：这个创建步骤不允许在存在文件的路径下执行。随后，运行 `jekyll serve`，看到了正常的 Log 信息。

    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ jekyll serve
    Configuration file: /home/jiangz/lylandris.github.io/_config.yml
                Source: /home/jiangz/lylandris.github.io
           Destination: /home/jiangz/lylandris.github.io/_site
          Generating...
                        done.
     Auto-regeneration: enabled for '/home/jiangz/lylandris.github.io'
    Configuration file: /home/jiangz/lylandris.github.io/_config.yml
        Server address: http://0.0.0.0:4000/
      Server running... press ctrl-c to stop.

到这里基本的架子就可以通过浏览器看到了。

![基本架子的截图](/images/the-framework-of-my-blog.png)

编辑 `.gitignore` 将 jekyll 产生的临时路径 `_site` 排除在提交列表外，因为 GitHub 自己就有 jekyll 运行。提交改动后，就可以在主页看到更新的日志啦。
