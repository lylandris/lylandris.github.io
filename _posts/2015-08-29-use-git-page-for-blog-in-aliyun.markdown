---
layout: post
title:  "在阿里云搭建 Jekyll 环境"
date:   2015-08-29 18:56:10
categories: 技术日志
---

# 在阿里云搭建 Jekyll 环境

考虑到 GitHub Page 需要 Jekyll 环境，而使用的笔记本没权限安装 Linux 或者虚拟机，因此就在阿里云的上海节点搞了一台 ECS，打算以后在这个机器上处理写好的日志。
不过在搭建过程中，也遇到了不少问题。

参考了先驱者 jibing57 的 blog <http://www.jibing57.com/blog/2015/03/19/github-pages.html>，第一句话“Github Pages搭建参见官方说明 <https://pages.github.com/>, 简洁明了，一目了然。”就折腾了半天。

系统选取了 `CentOS 7.0 x86_64` ，使用 `gem install bundler` 安装的时候，报错如下：

    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ gem install bundler
    ERROR:  Could not find a valid gem 'bundler' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/latest_specs.4.8.gz)

第一反应就是“神秘人（You-Known-Who）”。所以 Google 了一下，发现一篇文章 <http://www.cnblogs.com/ToDoToTry/p/4422454.html>，提到了这个问题。在文章的介绍中，找到了一个淘宝网搭建的镜像 <https://ruby.taobao.org/>。按照镜像的说明修改后，完成 `gem install bundler`。

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

搜索了好几个地方：

* http://www.bytecho.com/2014/04/16/install-jekyll-on-mavericks/ 尝试安装了 `clang` 或 `gcc`，不行
* 参考了其中引用的链接 http://stackoverflow.com/questions/22846685/clang-build-error-when-trying-to-install-jekyll ，还是不行
* 找到了 http://www.zhihu.com/question/26096289 发现提问者遇到了同样的问题
* 翻阅到了 http://segmentfault.com/q/1010000000392817 和 http://stackoverflow.com/questions/4393189/failing-installing-pg-gem-mkmf-rb-cant-find-header-files-for-ruby-mac-osx-1 看到了里面虽然没采纳，但可能的一个原因，没装 ruby-devel

于是立即 `sudo yum install ruby-devel` ，再执行 `bundle install -V` 过了刚才的砍，进入下一个坑。

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

看着看着总觉得缺了什么，尝试搜索了一下软件库：

    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ sudo yum search zlib-dev
    Loaded plugins: langpacks
    ================================================ N/S matched: zlib-dev =================================================
    ghc-zlib-devel.x86_64 : Haskell zlib library development files
    zlib-devel.i686 : Header files and libraries for Zlib development
    zlib-devel.x86_64 : Header files and libraries for Zlib development

      Name and summary matches only, use "search all" for everything.
    [jiangz@iZ11w7r4n04Z lylandris.github.io]$ sudo yum install zlib-devel

终于看到了下面的话：

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

写了个简单的 Hello World 后，按照说明运行 `bundle exec jekyll serve`，继续报错。

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

找到了 <https://ruby-china.org/topics/692> 暴力地直接装上了 nodejs （平时老听说，但始终不知道也没去怎么了解这是什么东东）。

开始挖 <https://jekyllrb.com/> ，在目录下执行 `jekyll new .`，创建基本文件后，运行 `jekyll serve`，浏览器访问看到了基本的架子。

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

