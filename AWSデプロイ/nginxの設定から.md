# nginxの設定、pumaの設定、railsの設定

## nginxの設定

```shell
[USERNAME@ip-10-0-11-43 ~]$ sudo amazon-linux-extras install nginx1.12
[USERNAME@ip-10-0-11-43 ~]$ sudo service nginx start
```

パブリックIPアドレスにアクセスしてnginxの画面が出ていたらOK

### nginxをrailsとつなげる

```
[USERNAME@ip-10-0-11-43 ~]$ cd /etc/nginx/conf.d
[USERNAME@ip-10-0-11-43 conf.d]$ vim hashlog.conf  # ここは自由
```

`hashlog`はディレクトリ名

```conf
# log directory
error_log  /var/www/hashlog/log/nginx.error.log;
access_log /var/www/hashlog/log/nginx.access.log;
# max body size
client_max_body_size 2G;
upstream hashlog {
  # for UNIX domain socket setups
  server unix:///var/www/hashlog/tmp/sockets/puma.sock fail_timeout=0;
}
server {
  listen 80;
  server_name 54.238.193.62; # 作成したEC2のIPアドレス
#  server_name 3.115.59.106; # IPアドレス
#  return 301 https://hashlog.work; # リダイレクト先
  # nginx so increasing this is generally safe...
  keepalive_timeout 5;
  # path for static files
  root /var/www/hashlog/public;
  # page cache loading
  try_files $uri/index.html $uri.html $uri @app;
  location @app {
    # HTTP headers
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://hashlog;
  }
  # Rails error pages
  error_page 500 502 503 504 /500.html;
  location = /500.html {
      root /var/www/hashlog/public;
  }
}
```

```shell
[USERNAME@ip-10-0-11-43 ~]$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

[USERNAME@ip-10-0-11-43 ~]$ sudo service nginx start
```

## pumaの設定

ローカルで`config/puma/production.rb`を作成

```rb
# config/puma/production.rb
environment "production"

# UNIX Socket
tmp_path = "#{File.expand_path("../../..", __FILE__)}/tmp"
bind "unix://#{tmp_path}/sockets/puma.sock"

threads 3, 3
workers 2
preload_app!

# daemonize
pidfile "#{tmp_path}/pids/puma.pid"
# stdout_redirect "#{tmp_path}/logs/puma.stdout.log", "#{tmp_path}/logs/puma.stderr.log", true

# Allow puma to be restarted by `rails restart` command.
plugin :tmp_restart
```

これをgit push & pull

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec puma -C config/puma/production.rb -e production -d
[32456] Puma starting in cluster mode...
[32456] * Version 3.12.6 (ruby 2.6.6-p146), codename: Llamas in Pajamas
[32456] * Min threads: 3, max threads: 3
[32456] * Environment: production
[32456] * Process workers: 2
[32456] * Preloading application
[32456] * Listening on unix:///var/www/hashlog/tmp/sockets/puma.sock
[32456] ! WARNING: Detected 2 Thread(s) started in app boot:
[32456] ! #<Thread:0x0000000001fd4278@/home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/concurrent-ruby-1.1.6/lib/concurrent-ruby/concurrent/atomic/ruby_thread_local_var.rb:38 sleep_forever> - /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/concurrent-ruby-1.1.6/lib/concurrent-ruby/concurrent/atomic/ruby_thread_local_var.rb:40:in `pop'
[32456] ! #<Thread:0x0000000003aa6718@/home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/activerecord-5.2.4.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:299 sleep> - /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/activerecord-5.2.4.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:301:in `sleep'
[32456] * Daemonizing...
# デーモンニングとなったらOK
# 一度エラーが出たけど、指定のファイルを作ったらうまくいった
```

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ sudo nginx -s stop
nginx: [warn] server name "http://18.182.8.118/" has suspicious symbols in /etc/nginx/conf.d/hashlog.conf:12
[USERNAME@ip-10-0-11-43 puma]$ sudo nginx -s stop
nginx: [error] open() "/run/nginx.pid" failed (2: No such file or directory)

[USERNAME@ip-10-0-11-43 puma]$ sudo touch /run/nginx.pid

[USERNAME@ip-10-0-11-43 puma]$ sudo nginx -s stop
nginx: [error] invalid PID number "" in "/run/nginx.pid"

[USERNAME@ip-10-0-11-43 hashlog]$ sudo service nginx start
Redirecting to /bin/systemctl start nginx.service
```

これで、IPアドレスにアクセスするとRailsのエラー画面が表示されるようになった。

```
[USERNAME@ip-10-0-11-43 hashlog]$ vim log/nginx.error.log
...
2020/06/09 21:37:33 [error] 3681#0: *7 connect() to unix:///var/www/hashlog/tmp/sockets/puma.sock failed (111: Connection refused) while connecting to upstream, client: 202.208.137.136, server: 18.182.8.118, request: "GET / HTTP/1.1", upstream: "http://unix:///var/www/hashlog/tmp/sockets/puma.sock:/", host: "18.182.8.118"
```


## Railsの設定

### 本番環境のタイムゾーン

```shell
[USERNAME@ip-10-0-11-43 ~]$ date
2020年  6月  9日 火曜日 12:18:55 UTC

[USERNAME@ip-10-0-11-43 ~]$ sudo ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

[USERNAME@ip-10-0-11-43 ~]$ date
2020年  6月  9日 火曜日 21:19:09 JST

[USERNAME@ip-10-0-11-43 ~]$ sudo vi /etc/sysconfig/clock

# 以下の内容を変更
ZONE="Asia/Tokyo"
UTC= true

[USERNAME@ip-10-0-11-43 ~]$ sudo reboot
```

https://qiita.com/Rubyist_SOTA/items/b0b5b7462b9af6b7a7bf

### redisのインストール

```shell
[USERNAME@ip-10-0-11-43 ~]$ sudo rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

[USERNAME@ip-10-0-11-43 ~]$ sudo yum install --enablerepo=epel,remi redis

[USERNAME@ip-10-0-11-43 ~]$ systemctl list-unit-files --type=service | grep redis
redis-sentinel.service                        disabled
redis.service                                 disabled

[USERNAME@ip-10-0-11-43 ~]$ systemctl enable redis
Failed to execute operation: The name org.freedesktop.PolicyKit1 was not provided by any .service files

[USERNAME@ip-10-0-11-43 ~]$ sudo systemctl enable redis
Created symlink from /etc/systemd/system/multi-user.target.wants/redis.service to /usr/lib/systemd/system/redis.service.

[USERNAME@ip-10-0-11-43 ~]$ systemctl list-unit-files --type=service | grep redis
redis-sentinel.service                        disabled
redis.service                                 enabled

[USERNAME@ip-10-0-11-43 ~]$ sudo systemctl start redis
```

なぜ追加したのかわからなかったので削除した。

```shell
[aiandrox@ip-10-0-11-43 ~]$ sudo yum remove redis
```

### nodeのインストール

```shell
[USERNAME@ip-10-0-11-43 log]$ cd ~
[USERNAME@ip-10-0-11-43 ~]$ git clone https://github.com/creationix/nvm.git ~/.nvm
[USERNAME@ip-10-0-11-43 ~]$ source ~/.nvm/nvm.sh
[USERNAME@ip-10-0-11-43 ~]$ vim .bash_profile
# 以下を追加
if [ -f ~/.nvm/nvm.sh ]; then
        . ~/.nvm/nvm.sh
fi

[USERNAME@ip-10-0-11-43 ~]$ nvm install 14.2.0  # ローカルと合わせた
Downloading and installing node v14.2.0...
Downloading https://nodejs.org/dist/v14.2.0/node-v14.2.0-linux-x64.tar.xz...
########################################################################################################################################################################### 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v14.2.0 (npm v6.14.4)

[USERNAME@ip-10-0-11-43 ~]$ npm --version
6.14.4
```

### yarnのインストール

```shell
[USERNAME@ip-10-0-11-43 ~]$ npm install yarn -g
/home/USERNAME/.nvm/versions/node/v14.2.0/bin/yarn -> /home/USERNAME/.nvm/versions/node/v14.2.0/lib/node_modules/yarn/bin/yarn.js
/home/USERNAME/.nvm/versions/node/v14.2.0/bin/yarnpkg -> /home/USERNAME/.nvm/versions/node/v14.2.0/lib/node_modules/yarn/bin/yarn.js
+ yarn@1.22.4
added 1 package in 0.387s

[USERNAME@ip-10-0-11-43 ~]$ yarn --version  # 結果的にローカルと同じになった
1.22.4
```

### プリコンパイル

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec rails assets:precompile RAILS_ENV=production
yarn install v1.22.4
[1/4] Resolving packages...
[2/4] Fetching packages...
...
[4/4] Building fresh packages...
Done in 158.24s.
rails aborted!
LoadError: cannot load such file -- uglifier
/var/www/hashlog/bin/rails:9:in `<top (required)>'
/var/www/hashlog/bin/spring:15:in `require'
/var/www/hashlog/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'

Caused by:
Bootsnap::LoadPathCache::FallbackScan:

Tasks: TOP => assets:precompile
(See full trace by running task with --trace)
```

`config/environments/production.rb`の`config.assets.js_compressor = :uglifier`をコメントアウト（webpacker以降のときにgemファイルを消していたがこれは消していなかった）  
ついでに`config.consider_all_requests_local = true`にしておく

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec puma -C config/puma/production.rb -e production -d
[USERNAME@ip-10-0-11-43 hashlog]$ sudo nginx -s stop
[USERNAME@ip-10-0-11-43 hashlog]$ sudo service nginx start
```

でアクセス

[![Image from Gyazo](https://i.gyazo.com/bd189a14dc54ca8d1ad2cd900e2e2524.png)](https://gyazo.com/bd189a14dc54ca8d1ad2cd900e2e2524)
やっぱりプリコンパイルができていない。

### プリコンパイル

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec rails assets:precompile RAILS_ENV=production
yarn install v1.22.4
[1/4] Resolving packages...
success Already up-to-date.
Done in 1.02s.
Compiling...
Compilation failed:
EntryModuleNotFoundError: Entry module not found: Error: Can't resolve 'eslint-loader' in '/var/www/hashlog'
```

あれ待って`bin/webpack`？？→同じ事っぽい  
https://numb86-tech.hatenablog.com/entry/2019/01/09/215714

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ bin/webpack -w

webpack is watching the files…

Insufficient number of arguments or no entry found.
Alternatively, run 'webpack(-cli) --help' for usage info.

Hash: a20a912099ac1916eb01
Version: webpack 4.42.1
Time: 58ms
Built at: 2020/06/10 21:23:33
        Asset      Size  Chunks             Chunk Names
manifest.json  23 bytes          [emitted]

ERROR in Entry module not found: Error: Can't resolve 'eslint-loader' in '/var/www/hashlog'

ERROR in Entry module not found: Error: Can't resolve 'eslint-loader' in '/var/www/hashlog'
```

```json
# hashlog/public/packs/manifest.json

{
  "entrypoints": {}
}
```

### es-lintのインストール

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ yarn run eslint --ext .vue --ext .js app/javascript
yarn run v1.22.4
$ /var/www/hashlog/node_modules/.bin/eslint --ext .vue --ext .js app/javascript

Oops! Something went wrong! :(

ESLint: 6.8.0.

ESLint couldn't find the plugin "eslint-plugin-vue".

(The package "eslint-plugin-vue" was not found when loaded as a Node module from the directory "/var/www/hashlog".)

It's likely that the plugin isn't installed correctly. Try reinstalling by running the following:

    npm install eslint-plugin-vue@latest --save-dev

The plugin "eslint-plugin-vue" was referenced from the config file in ".eslintrc.json".

If you still can't figure out the problem, please stop by https://gitter.im/eslint/eslint to chat with the team.

error Command failed with exit code 2.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
[USERNAME@ip-10-0-11-43 hashlog]$ yarn add eslint-plugin-vue@latest --save-dev
yarn add v1.22.4
[1/4] Resolving packages...
[2/4] Fetching packages...
...
success Saved 1 new dependency.
info Direct dependencies
└─ eslint-plugin-vue@6.2.2
info All dependencies
└─ eslint-plugin-vue@6.2.2
Done in 6.50s.
```

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ bin/webpack -w
# 黄色いwarnだけで通ったあああああ
```

## その他独自の設定

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ rake db:seed_fu RAILS_ENV=production
```
