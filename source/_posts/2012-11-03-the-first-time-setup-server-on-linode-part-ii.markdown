---
layout: post
title: "The First Time Setup Server on Linode Part II"
date: 2012-11-03 20:42
comments: true
categories: ["rails", "thin", "capistrano"]
---

[上一篇][1]講述最簡單Deploy到Linode（或其他VPS）的方法，
除了如何安裝該安裝的以外，看起來好像就只有clone code上去而已（？），
而Part II則負責database, thin以及nginx的設定。

<!-- more -->

1. Database Setting
-------------------
在上篇，因為Database還沒設定好，所以不能使用migration，也不能開啟server。
任何有關Database的動作一律都要繞過去，不過至少我們知道Capistrano是可以正常運作的。
而設定Database的方法滿簡單的(看起來有五步，但其實熟練的話兩個步驟就做完了)：

1. 設定shared config folder
2. 設定config/database.example.yml
3. copy config/database.example.yml(local) to #{shared\_path}/config/database.yml(server side)
4. 建立task for symbolic of database.yml
5. check

1.1 設定shared config folder
---------------------------

建立一個shared的config folder，這樣就可以在個releases中分享同樣的設定值  
自動生成shared config folder的recipe如下：
``` ruby config/deploy.rb
  # in namespace :deploy
  task :setup_config, roles: :app do
    run "mkdir -p #{shared_path}/config"
  end
  after "deploy:setup", "deploy:setup_config"
```

完成後請再次執行`cap deploy:setup`，這樣server就有shared config folder了。

1.2 設定config/database.example.yml
-----------------------------------

複製dataabse.yml成database.example.yml，要記得把config/database.yml從repository拿掉，
database.example.yml也請不要輸入真的密碼。再把database.example.yml加進repository

``` yaml database.example.yml

production:
  adapter: postgresql
  encoding: unicode
  database: [[database name]]
  pool: 5
  host: localhost
  username: [[db username]]
  password: [[db password]]

```

完成後`git commit -a && git push`，再`cap deploy`，這樣server就有database.example.yml
可以參考了。

1.3 copy config/database.example.yml to #{shared\_path}/config/database.yml
---------------------------------------------------------------------------
登入server，把database.example.yml複製一份到#{shared\_path}/config/database.yml，
並填入正確的database資料

1.4 建立task for symbolic of database.yml
-----------------------------------------

在這一步我們建立symbolic link，這樣就可以做到各release都share同一份config file了。

``` ruby config/deploy.rb
  # in namespace :deploy
  task :symlink_config, roles: :app do
    run "ln -nfs #{shared_path}/config/database.yml #{release_path}/config/database.yml"
  end
  after "deploy:finalize_update", "deploy:symlink_config"
```

1.5 check
---------
最後把`Capfile`中的`load 'deploy/assets'`打開，再做一次`cap deploy`，
如果都沒噴錯誤的話，就是成功了。


其實直接把database.yml加進repository commit上去也可以，不過這樣你的密碼就會被其他人看到。

2. Thin Setting
---------------
Database設定完成以後，接下來就是Thin server的設定。capistrano內部預定：

* deploy:start
* deploy:stop
* deploy:restart

為三個給app使用的task，實作上就看用什麼樣的app server有不同的設定。

``` ruby config/deploy.rb
  # 在 task command, roles: :app, :except => {no_release: true} do; end 中插入
  %w[start stop restart].each do |command|
    desc "#{command} thin server"
    task command, roles: :app, :except => {no_release: true} do
      run "cd #{current_path} && bundle exec thin #{command} -e production -d -s 1"
    end
  end
```

設定完成後，執行`cap deploy:cold`，capistrano會做以下三件事情

1. update
2. migration _# 之前都沒做的部份_
3. start

這樣應該就會正常在port 3000啟動，thin server設定也算是完成了一半。

3. Nginx Setting
----------------
雖然這樣其實就可以使用了，但是一般來說都會在app server前面再擋一個web server，
可以用來serve static file、load balance、caching等等，可以大大減輕app server的負擔。  
（雖然大概這麼理解，但我本人其實除了static file以外沒什麼經驗XD)

OK, 所以我們要一個Nginx設定擋。而其實安裝完成以後就會有一個Default的設定檔，
在`/etc/nginx/nginx.conf`，而這個設定擋會去include`/etc/nginx/sites-enabled/`
資料夾下的所有設定擋，也就是我們只需要製作一個子設定擋放在這個資料夾下面就行了。
([Nginx Wiki][2]上面的範例是直接從`/etc/nginx/nginx.conf`講解)

又因為我們不想把設定擋到處分散，所以放在rails project裡面，再用symbolic link
連會比較集中。

``` plain config/nginx.conf
upstream thin {
  server 127.0.0.1:3000;
}

server {
  listen 80; 

  # for assets serve
  root /home/deployer/apps/payu/current/public;
  location ^~ /assets/ {
    gzip_static on; 
    expires max;
    add_header Cache-Control public;
  }

  # for static page
  try_files $uri/index.html $uri @thin; # 一個一個try，直到有東西或最後丟給@thin
  
  # reverse proxy
  location @thin {			# @thin 只是取個名字，上面的try_files有用到
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://thin; # 配合上面的upstream
  }

  error_page 500 502 503 504 /500.html;
}
```

接著加入symbolic link到symlink_config recipe裡：

``` ruby config/deploy.rb
  # 修改 task :symlink_config
  task :symlink_config, roles: :app do
    run "ln -nfs #{shared_path}/config/database.yml #{release_path}/config/database.yml"
    sudo "ln -nfs #{current_path}/config/nginx.conf /etc/nginx/sites-enabled/#{application}"
	# 注意一定要用sudo，因為要存取/etc下面的資料
  end
```

最後三件事情：

1. 執行`cap deploy`，把新的設定放上去。
2. 刪除`/etc/nginx/sites-enabled/defaults`
3. `service nginx restart`

這樣應該就可以在一般的port 80連到了。

One more thing  
--------------
轉用socket file，詳細步驟我就不寫了。

1. 在nginx.config裡面，把`127.0.0.1:3000`改成`unix:/tmp/thin.sock`
2. 在`deploy:start/stop/restart`中，加`-S /tmp/thin.sock`

其實這步可以不用做，不過如果有人可以連production server的port 3000不是很怪嗎:p


補充1: current\_path 與 release\_path 的差異
--------------------------------------------
current\_path 是指連到current這個symbolic link的資料夾  
release\_path 是使用真實的資料夾

補充2: Nginx設定修改後必須重新啟動
----------------------------------
也就是如果nginx.conf有修改就要`sudo service nginx restart`


TODO: 加速Assets Precompile
---------------------------
其實我本來以為assets precompile不會花太多時間，
結果沒想到在linode最小的instance上要跑滿久的。
當初還因為這個問題跟別人筆戰起來了，結果自己還是碰到了orz


[1]: /blog/2012/11/03/the-first-time-setup-server-on-linode/
[2]: http://wiki.nginx.org/FullExample

