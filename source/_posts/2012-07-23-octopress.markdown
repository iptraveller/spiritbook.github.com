---
layout: post
title: "使用github创建blog"
date: 2012-07-23 05:42
comments: true
categories: 
---
code6同学很早就用github搞了一个博客，[猛击这里](http://code6.github.com/) <p>
今天根据教程试了一下，发现在ubuntu下还是略有不同。本菜使用ubuntu10.04，大致需要操作如下。<p>
##0.略过github账号申请过程。
##1. 安装[rvm](https://rvm.io/)
    $ curl https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer | bash -s stable
##2. 修改环境变量<p>
    [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"
    source ~/.bashrc
在~/.bashrc中添加如上字眼，如果是root用户，需要将$HOME/.rvm替换成/usr/local/rvm<p>
##3. 安装ruby,rubygems
    rvm install 1.9.2 && rvm use 1.9.2
    rvm rubygems latest
##4.下载octopress
    git clone git://github.com/imathis/octopress.git octopress
    cd octopress
##5.安装bundler
    gem install bundler
    bundle install
##6.生成模板文件
    rake install
##7.发布到github
先需要在github中创建<b>yourname</b>.github.com的repository<p>
设定github pages
	rake setup_github_pages
输入git@github.com:<b>yourname/yourname</b>.github.com.git<p>
建立并发布<p>
	rake generate
	rake deploy
##8.将source也加入git
	git add .
	git commit -m 'initial source commit'
	git push origin master
##9.创建新文章
	rake new_post["how-to-install-octopress"]
##A.支持中文显示
文章makrdown保存为无bom的utf-8即可。<p>
##B.rake generate错误1
	in 'require': no such file to load -- openssl (LoadError)
原因:本地是root用户，路径不对。解决方法:
	rvm reinstall 1.9.2 --with-openssl-dir=/usr/local/rvm
##C.本地预览
	rake preview
浏览器中输入http://localhost:4000
##附录(参考链接)
1. [Blog = GitHub + Octopress](http://mrzhang.me/blog/blog-equals-github-plus-octopress.html)<p>
2. [Octopress: a blogging framework for hackers](http://lyhdev.com/note:octopress)<p>
3. [Ruby开源项目介绍(1)：octopress——像黑客一样写博客](http://www.yangzhiping.com/tech/octopress.html)<p>