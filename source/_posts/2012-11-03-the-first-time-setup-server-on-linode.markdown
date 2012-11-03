---
layout: post
title: "The First Time Setup Server on Linode"
date: 2012-11-03 13:58
comments: true
categories: ["linux", "ubuntu", "rails", "postgresql"]
---

因為發現Heroku實在太貴了，光設定一個HTTPS就要收你20鎂(而且還是每月收)，
再開一個worker就貴到炸掉了，毅然決然的就跟業主建議改放Linode。
而這篇文章就是來記錄我這次在Linode架設Rails on Ubuntu的過程。

<!-- more -->

因為主要參考 [Railscasts.com的Deploying to a VPS][1] 這篇文章，
而這篇文章其實是要付費(pro)才看得到，所以在railscasts有提到的部份，
我這邊就大略帶過，有興趣的就直接去原站看囉～其實很划算的。

1. Linode 開 VM
---------------
當然要先註冊、付錢，然後開VM。

但是問題是現在上面的Ubuntu版本是11.04（Railscasts上面是10.04），
所以就只能選這個了。而且版本不同也是造成問題的主要原因之一。

再來就用ssh登入囉。


2. Server Install
-----------------
要安裝的東西及順序如下：

1. curl
2. git-core (for capistrano)
3. python-software-properties
4. nginx
5. postgresql (這邊會出現[問題](#install_psql_problem))
	* change password
	* create user 
	* create database
6. telnet (postfix)
7. postfix (我沒裝因為用gmail)
8. nodejs (for compile assets pipeline)
9. [rvm requirements](#rvm_requirements) (必看！)
10. rvm or rbenv (我這邊是用rvm)
11. ruby
12. bundler (如果沒有跑rvm requirements就會出現一些[問題](#install_bundler_problem))

2.1 RVM Requirements
--------------------
這邊是裝rvm時會需要的東西，雖然沒安裝這些，rvm應該都裝得起來。
但是之後如過要用到某些package時就會碰到更多麻煩，該裝還是裝一裝吧。

``` bash
sudo apt-get -y install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config
```

如果是在裝了ruby以後才發現有東西沒裝，那就請重新安裝ruby

```bash
	rvm reinstall 1.9.3 --force
```

3. Other TODO before deploy
---------------------------
1. adduser deployer
2. ssh git@github.com (exchange key with github)
	* 雖然最後是Permission denied但似乎還是可以用（？


4. Rails Project Setting
------------------------
在安裝完server以後，要來把Rails的Gemfile設定好：

1. capistrano ([very first version](#capistrano_setting))
2. thin or unicorn (與Railscast不同，這邊用thin)

<a id="capistrano_setting"></a>
4.1 Capistrano Setting (Very first version deploy)
--------------------------------------------------
透過Capistrano來做deploy的動作，設定步驟如下：

1. capify (請不要在這個時間點開啟`load 'deploy/assets'` [[WHY]](#dont_enable_assets_too_early))
2. write config/deploy.rb
3. cap deploy:setup
4. cap deploy
	* 如果有github連線的問題，請執行`ssh-add`，這樣ssh agent forward才會成功

``` ruby config/deploy.rb

require "bundler/capistrano"
require "rvm/capistrano" # 這行跟railscast不同，用rvm必加
						 # 不加會抱怨bundle找不到
						 # 當然local必須安裝rvm-capistrano
						 # gem install rvm-capistrano

server "[[server ip here]]", :web, :app, :db, primary: true

set :user, "deployer"
set :application, "[[app name here]]"
set :deploy_to, "/home/#{user}/apps/#{application}"
set :deploy_via, :remote_cache
set :use_sudo, false

set :scm, "git"
set :repository, "{{repository here}}"
set :branch, "master"

default_run_options[:pty] = true
ssh_options[:forward_agent] = true

after "deploy:restart", "deploy:cleanup"

namespace :deploy do
  %w[start stop restart].each do |command|
    desc "#{command} thin server"
    task command, roles: :app, :except => {no_release: true} do
    end 
  end 
end
```

結語
----
啥？這樣就結語？不是還有一大堆東西沒講嗎？

沒錯，不過到這一步就可以把code傳到server上了，差在還不能跑而已。
下一篇就會接著講如何設定Thin, nginx, database等

(待續)

----------

Problems
========

<a id="install_psql_problem"></a>
Problem of Install PostgreSQL
-----------------------------
照著步驟來會發現無法登入postgres service，原因是其實postgres並沒有正常被啟動，
而沒有被啟動的原因是環境變數中缺少了兩個值，把他們append到bashrc就可以了

from: <http://serverfault.com/questions/308295/postgresql-didnt-install-on-ubuntu-11-04>

{% codeblock ~/.bashrc lang:bash %}
	export LANGUAGE="en_US.UTF-8"
	export LC_ALL="en_US.UTF-8"
{% endcodeblock %}

記得要自己開啟psql server

{% codeblock lang:bash %}
	source ~/.bashrc # reload ENV
	pg_createcluster 8.4 main --start
{% endcodeblock %}

<a id="install_bundler_problem"></a>
Problem of Install Bundler
--------------------------
**UPDATE**: 如果有正常安裝rvm requirements應該是不會有這個問題才是

與Railscasts不同的是，我是用rvm安裝而非rbenv（因為覺得local是用rvm，所以認為server 
side用rvm會比較沒問題），在這邊又碰上了一些package的問題

在執行`gem install bundler`時噴的錯誤如下：

``` bash
	It seems your ruby installation is missing psych (for YAML output).
	To eliminate this warning, please install libyaml and reinstall your ruby.
	ERROR:  Loading command: install (LoadError)
		cannot load such file -- zlib
	ERROR:  While executing gem ... (NameError)
		uninitialized constant Gem::Commands::InstallCommand
```

應該就是少了zlib這個library，解決方法：
(不知道為什麼不能直接`apt-get install zlib`，可能名稱錯誤)

``` bash
	# 因為他沒辦法check checksum所以加後面的--verify-downloads 1
	rvm pkg install zlib --verify-downloads 1 
	rvm reinstall all --force
```

再重跑一次`gem install bundler`就可以了。

<a id="dont_enable_assets_too_early"></a>
Don't Enable Assets TOO EARLY
-----------------------------
先不要開啟Capfile裡面的`load 'deploy/assets'`

原因是，某些Gem會在assets precompile的程序連接資料庫，
但我們目前還沒設定資料庫連線，所以會炸掉。 

TODO
----
1. 自動化安裝
2. 貌似rvm-capistrano附帶了ruby, rvm安裝程序


[1]: http://railscasts.com/episodes/335-deploying-to-a-vps
