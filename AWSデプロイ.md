# デプロイのコマンド記録

## 環境

- Ruby 2.6.6
- Rails 5.2.4.3
- node 14.4.0
- MySQL 8.0.19
- Bundler 2.1.4

`USERNAME`はEC2インスタンス内のユーザー名


`$ ssh -v -i "aws-key.pem" ec2-user@IPアドレス`

```shell
Last login: Mon Jun  8 12:39:32 2020 from o137136.dynamic.ppp.asahi-net.or.jp

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
4 package(s) needed for security, out of 8 available
Run "sudo yum update" to apply all updates.

[ec2-user@ip-10-0-11-43 ~]$ sudo yum -y update
読み込んだプラグイン:extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                               | 3.7 kB     00:00
依存性の解決をしています
...
完了しました!
```

## ユーザーの追加

```shell
[ec2-user@ip-10-0-11-43 ~]$ sudo useradd USERNAME

[ec2-user@ip-10-0-11-43 ~]$ sudo passwd USERNAME
ユーザー USERNAME のパスワードを変更。
新しいパスワード: # 自分で決める
新しいパスワードを再入力してください:
passwd: すべての認証トークンが正しく更新できました。

[ec2-user@ip-10-0-11-43 ~]$ sudo visudo

vimで
...
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
USERNAME  ALL=(ALL)       NOPASSWD: ALL # 追加
...

[ec2-user@ip-10-0-11-43 ~]$ cat ~/.ssh/authorized_keys
# コピー

[ec2-user@ip-10-0-11-43 ~]$ su - USERNAME
パスワード:

[USERNAME@ip-10-0-11-43 ~]$ mkdir .ssh
[USERNAME@ip-10-0-11-43 ~]$ chmod 700 .ssh
[USERNAME@ip-10-0-11-43 ~]$ touch ~/.ssh/authorized_keys
[USERNAME@ip-10-0-11-43 ~]$ chmod 600 ~/.ssh/authorized_keys
[USERNAME@ip-10-0-11-43 ~]$ vim ~/.ssh/authorized_keys
# ペースト
```

`$ ssh -v -i "aws-key.pem" USERNAME@IPアドレス`でEC2に入れるようになったよヤッター

## 初期設定インストール

```shell
[USERNAME@ip-10-0-11-43 ~]$ sudo yum install \
git make gcc-c++ patch \
openssl-devel \
libyaml-devel libffi-devel libicu-devel \
libxml2 libxslt libxml2-devel libxslt-devel \
zlib-devel readline-devel \
mysql mysql-server mysql-devel \
ImageMagick ImageMagick-devel \
epel-release

[USERNAME@ip-10-0-11-43 ~]$ sudo amazon-linux-extras install epel

[USERNAME@ip-10-0-11-43 ~]$ curl -L git.io/nodebrew | perl - setup
...
========================================
Export a path to nodebrew:

export PATH=$HOME/.nodebrew/current/bin:$PATH
========================================

[USERNAME@ip-10-0-11-43 ~]$ echo 'export PATH=$HOME/.nodebrew/current/bin:$PATH' >> ~/.bash_profile

[USERNAME@ip-10-0-11-43 ~]$ source ~/.bash_profile

[USERNAME@ip-10-0-11-43 ~]$ nodebrew install-binary latest
Fetching: https://nodejs.org/dist/v14.4.0/node-v14.4.0-linux-x64.tar.gz
######################################################################### 100.0%
Installed successfully

[USERNAME@ip-10-0-11-43 ~]$ nodebrew list
v14.4.0
current: none
# ローカルは14.2.0だからまあいいかな

[USERNAME@ip-10-0-11-43 ~]$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
[USERNAME@ip-10-0-11-43 ~]$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
[USERNAME@ip-10-0-11-43 ~]$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
[USERNAME@ip-10-0-11-43 ~]$ source ~/.bash_profile
[USERNAME@ip-10-0-11-43 ~]$ rbenv install -v 2.6.6  # ローカルに合わせる
# めっちゃ長い
[USERNAME@ip-10-0-11-43 ~]$ rbenv global 2.6.6
[USERNAME@ip-10-0-11-43 ~]$ rbenv rehash
[USERNAME@ip-10-0-11-43 ~]$ ruby -v
ruby 2.6.6p146 (2020-03-31 revision 67876) [x86_64-linux]
```

## GitHubとの連携、クローン

```shell
[USERNAME@ip-10-0-11-43 ~]$ cd .ssh  # さっき.sshディレクトリを作っているはず

[USERNAME@ip-10-0-11-43 .ssh]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/USERNAME/.ssh/id_rsa):  # エンター
Enter passphrase (empty for no passphrase):  # エンター
Enter same passphrase again:  # エンター
Your identification has been saved in /home/USERNAME/.ssh/id_rsa.
Your public key has been saved in /home/USERNAME/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ZCbswiAYHY9AohfuoT+TgHdG4IcCwpMkN0+mjQ4P4UU USERNAME@ip-10-0-11-43.ap-northeast-1.compute.internal
The key's randomart image is:
+---[RSA 2048]----+
|%+OEo            |
|O@o# .           |
|B.% * o +        |
|.@ * . =         |
|+ = = . S        |
| + + .           |
|  =              |
|   o             |
|                 |
+----[SHA256]-----+

[USERNAME@ip-10-0-11-43 .ssh]$ cat id_rsa.pub
# コピーしてGitHubに登録

[USERNAME@ip-10-0-11-43 .ssh]$ cd /
[USERNAME@ip-10-0-11-43 /]$ sudo chown USERNAME var
[USERNAME@ip-10-0-11-43 var]$ sudo mkdir www
[USERNAME@ip-10-0-11-43 var]$ sudo chown USERNAME www
[USERNAME@ip-10-0-11-43 var]$ cd www
[USERNAME@ip-10-0-11-43 www]$ git clone git@github.com:USERNAME/hashlog.git
```

## Railsアプリの初期設定

```shell
[USERNAME@ip-10-0-11-43 www]$ cd hashlog/config/

[USERNAME@ip-10-0-11-43 config]$ vim master.key
# ローカルのmaster.keyをコピペ

[USERNAME@ip-10-0-11-43 config]$ vim database.yml

...
default: &default
  adapter: mysql2
  encoding: utf8mb4
  charset: utf8mb4
  collation: utf8mb4_unicode_ci
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: admin # RDSのユーザーネーム
  password: # RDSのパスワード
  host: # RDSのエンドポイント ....rds.amazonaws.com
...

[USERNAME@ip-10-0-11-43 config]$ cd /var/www/hashlog/
[USERNAME@ip-10-0-11-43 hashlog]$ gem install bundler:2.1.4  # bundle installが通らなかったときに指示された。ローカルも2.1.4だった。
[USERNAME@ip-10-0-11-43 hashlog]$ bundle install
```

## MySQLの設定

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec rails db:create RAILS_ENV=production
# ちなみに指定をしないとdevelopment, testができる

Authentication plugin 'sha256_password' cannot be loaded: /usr/lib64/mysql/plugin/sha256_password.so: cannot open shared object file: No such file or directory
Couldn't create 'hashlog_production' database. Please check your configuration.
rails aborted!
Mysql2::Error: Authentication plugin 'sha256_password' cannot be loaded: /usr/lib64/mysql/plugin/sha256_password.so: cannot open shared object file: No such file or directory
/var/www/hashlog/bin/rails:9:in `<top (required)>'
/var/www/hashlog/bin/spring:15:in `require'
/var/www/hashlog/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
Tasks: TOP => db:create
(See full trace by running task with --trace)

[USERNAME@ip-10-0-11-43 hashlog]$ mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)

[USERNAME@ip-10-0-11-43 hashlog]$ cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d

[USERNAME@ip-10-0-11-43 hashlog]$ cd /var/lib

[USERNAME@ip-10-0-11-43 lib]$ ls -a
.             chrony    gssproxy       misc       postfix    stateless
..            cloud     hibinit-agent  mlocate    rpcbind    systemd
alternatives  dbus      initramfs      nfs        rpm        update-motd
amazon        dhclient  logrotate      os-prober  rpm-state  xfsdump
authconfig    games     machines       plymouth   rsyslog    yum

[USERNAME@ip-10-0-11-43 lib]$ mkdir mysql
mkdir: ディレクトリ `mysql' を作成できません: Permission denied

[USERNAME@ip-10-0-11-43 lib]$ sudo mkdir mysql
[USERNAME@ip-10-0-11-43 lib]$ cd mysql/
[USERNAME@ip-10-0-11-43 mysql]$ sudo touch mysql.sock

[USERNAME@ip-10-0-11-43 mysql]$ mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (13)

[USERNAME@ip-10-0-11-43 mysql]$ sudo chmod 777 /var/lib/mysql/mysql.sock
[USERNAME@ip-10-0-11-43 mysql]$ mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (111)
```

https://saton2.hatenablog.com/entry/2018/10/16/200616


### mysql-serverのインストール

結論から言うと、ローカルにMySQLをインストールする必要はなかった。  
gemから攻めるべし。

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ mysql.server start
-bash: mysql.server: コマンドが見つかりません
[USERNAME@ip-10-0-11-43 ~]$ systemctl list-unit-files --type=service | grep mysql
# なにもない
```

https://dev.mysql.com/downloads/repo/yum/  
https://qiita.com/ymasaoka/items/7dc131dc98ba10a39854  
https://blog.apar.jp/linux/9868/

Amazon Linux2はLinux7なので、そこに行って、`No thanks, just start my download.`のURLをコピー

```shell
[USERNAME@ip-10-0-11-43 ~]$ sudo yum localinstall -y https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm  # コピーしたURL

[USERNAME@ip-10-0-11-43 ~]$ sudo yum install -y mysql-community-server

[USERNAME@ip-10-0-11-43 ~]$ mysqld --version
/usr/sbin/mysqld  Ver 8.0.20 for Linux on x86_64 (MySQL Community Server - GPL)

[USERNAME@ip-10-0-11-43 ~]$ systemctl start mysqld.service
Failed to start mysqld.service: The name org.freedesktop.PolicyKit1 was not provided by any .service files
See system logs and 'systemctl status mysqld.service' for details.

[USERNAME@ip-10-0-11-43 ~]$ sudo systemctl start mysqld.service
Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.

[USERNAME@ip-10-0-11-43 ~]$ systemctl list-unit-files --type=service | grep mysql
mysqld.service                                enabled
mysqld@.service                               disabled

[USERNAME@ip-10-0-11-43 ~]$ systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html

[USERNAME@ip-10-0-11-43 ~]$ journalctl -xe
 6月 09 03:42:06 ip-10-0-11-43.ap-northeast-1.compute.internal sshd[27274]: Received disconnect from 202.208.137.136 port 63749:11: disconnected by user
 6月 09 03:42:06 ip-10-0-11-43.ap-northeast-1.compute.internal sshd[27274]: Disconnected from 202.208.137.136 port 63749

[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec rails db:create RAILS_ENV=production  # なんでかわからんけどエラー文が変わっている
rails aborted!
LoadError: libmysqlclient.so.18: cannot open shared object file: No such file or directory - /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/mysql2-0.5.3/lib/mysql2/mysql2.so
/var/www/hashlog/config/application.rb:18:in `<main>'
/var/www/hashlog/Rakefile:4:in `<main>'
/var/www/hashlog/bin/rails:9:in `<top (required)>'
/var/www/hashlog/bin/spring:15:in `require'
/var/www/hashlog/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
(See full trace by running task with --trace)
```

`LoadError: libmysqlclient.so.18: cannot open shared object file: No such file or directory - /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/mysql2-0.5.3/lib/mysql2/mysql2.so`を手がかりに調べてみる。

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ cd /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/mysql2-0.5.3/lib/mysql2/

[USERNAME@ip-10-0-11-43 mysql2]$ ls -a
.   client.rb   em.rb     field.rb   result.rb     version.rb
..  console.rb  error.rb  mysql2.so  statement.rb

[USERNAME@ip-10-0-11-43 mysql2]$ mysql2.so
-bash: mysql2.so: コマンドが見つかりません

[USERNAME@ip-10-0-11-43 hashlog]$ gem uninstall mysql2
Successfully uninstalled mysql2-0.5.3

[USERNAME@ip-10-0-11-43 hashlog]$ gem install mysql2
Fetching mysql2-0.5.3.gem
Building native extensions. This could take a while...
ERROR:  Error installing mysql2:
	ERROR: Failed to build gem native extension.

    current directory: /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/mysql2-0.5.3/ext/mysql2
/home/USERNAME/.rbenv/versions/2.6.6/bin/ruby -I /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/2.6.0 -r ./siteconf20200609-29709-qomkf7.rb extconf.rb
checking for rb_absint_size()... yes
checking for rb_absint_singlebit_p()... yes
checking for rb_wait_for_single_fd()... yes
checking for -lmysqlclient... no
-----
mysql client is missing. You may need to 'sudo apt-get install libmariadb-dev', 'sudo apt-get install libmysqlclient-dev' or 'sudo yum install mysql-devel', and try again.
-----
*** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.

Provided configuration options:
	--with-opt-dir
	--without-opt-dir
	--with-opt-include
	--without-opt-include=${opt-dir}/include
	--with-opt-lib
	--without-opt-lib=${opt-dir}/lib
	--with-make-prog
	--without-make-prog
	--srcdir=.
	--curdir
	--ruby=/home/USERNAME/.rbenv/versions/2.6.6/bin/$(RUBY_BASE_NAME)
	--with-mysql-dir
	--without-mysql-dir
	--with-mysql-include
	--without-mysql-include=${mysql-dir}/include
	--with-mysql-lib
	--without-mysql-lib=${mysql-dir}/lib
	--with-mysql-config
	--without-mysql-config
	--with-mysql-dir
	--without-mysql-dir
	--with-mysql-include
	--without-mysql-include=${mysql-dir}/include
	--with-mysql-lib
	--without-mysql-lib=${mysql-dir}/lib
	--with-mysqlclientlib
	--without-mysqlclientlib

To see why this extension failed to compile, please check the mkmf.log which can be found here:

  /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/extensions/x86_64-linux/2.6.0/mysql2-0.5.3/mkmf.log

extconf failed, exit code 1

Gem files will remain installed in /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/mysql2-0.5.3 for inspection.
Results logged to /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/extensions/x86_64-linux/2.6.0/mysql2-0.5.3/gem_make.out
```

```shell
[USERNAME@ip-10-0-11-43 mysql2-0.5.3]$ cat /home/USERNAME/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/extensions/x86_64-linux/2.6.0/mysql2-0.5.3/mkmf.log

...
"gcc -o conftest -I/home/USERNAME/.rbenv/versions/2.6.6/include/ruby-2.6.0/x86_64-linux -I/home/USERNAME/.rbenv/versions/2.6.6/include/ruby-2.6.0/ruby/backward -I/home/USERNAME/.rbenv/versions/2.6.6/include/ruby-2.6.0 -I. -I/usr/local/include -I/home/USERNAME/.rbenv/versions/2.6.6/include    -O3 -ggdb3 -Wall -Wextra -Wdeclaration-after-statement -Wdeprecated-declarations -Wduplicated-cond -Wimplicit-function-declaration -Wimplicit-int -Wmisleading-indentation -Wpointer-arith -Wrestrict -Wwrite-strings -Wimplicit-fallthrough=0 -Wmissing-noreturn -Wno-cast-function-type -Wno-constant-logical-operand -Wno-long-long -Wno-missing-field-initializers -Wno-overlength-strings -Wno-packed-bitfield-compat -Wno-parentheses-equality -Wno-self-assign -Wno-tautological-compare -Wno-unused-parameter -Wno-unused-value -Wsuggest-attribute=format -Wsuggest-attribute=noreturn -Wunused-variable  -fPIC conftest.c  -L. -L/home/USERNAME/.rbenv/versions/2.6.6/lib -Wl,-rpath,/home/USERNAME/.rbenv/versions/2.6.6/lib -L/usr/local/lib -Wl,-rpath,/usr/local/lib -L/usr/local/lib/mysql -Wl,-rpath,/usr/local/lib/mysql -L. -L/home/USERNAME/.rbenv/versions/2.6.6/lib  -fstack-protector-strong -rdynamic -Wl,-export-dynamic     -Wl,-rpath,/home/USERNAME/.rbenv/versions/2.6.6/lib -L/home/USERNAME/.rbenv/versions/2.6.6/lib -lruby -lmysqlclient  -lm   -lc"
/usr/bin/ld: -lmysqlclient が見つかりません
collect2: エラー: ld はステータス 1 で終了しました
checked program was:
/* begin */
 1: #include "ruby.h"
 2:
 3: /*top*/
 4: extern int t(void);
 5: int main(int argc, char **argv)
 6: {
 7:   if (argc > 1000000) {
 8:     int (* volatile tp)(void)=(int (*)(void))&t;
 9:     printf("%d", (*tp)());
10:   }
11:
12:   return 0;
13: }
14:
15: int t(void) { ; return 0; }
/* end */

--------------------
```

> mysql client is missing. You may need to 'sudo apt-get install libmariadb-dev', 'sudo apt-get install libmysqlclient-dev' or 'sudo yum install mysql-devel', and try again.

とのことなので、mysql-develをインストールしてみる。


```shell
[USERNAME@ip-10-0-11-43 ~]$ sudo yum install mysql-devel

[USERNAME@ip-10-0-11-43 ~]$ gem install mysql2
Building native extensions. This could take a while...
Successfully installed mysql2-0.5.3
Parsing documentation for mysql2-0.5.3
Installing ri documentation for mysql2-0.5.3
Done installing documentation for mysql2 after 0 seconds
1 gem installed

[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec rails db:create RAILS_ENV=production
Access denied for user 'hashlog'@'10.0.11.43' (using password: NO)
Couldn't create 'hashlog_production' database. Please check your configuration.
rails aborted!
Mysql2::Error::ConnectionError: Access denied for user 'hashlog'@'10.0.11.43' (using password: NO)
/var/www/hashlog/bin/rails:9:in `<top (required)>'
/var/www/hashlog/bin/spring:15:in `require'
/var/www/hashlog/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
Tasks: TOP => db:create
(See full trace by running task with --trace)

[USERNAME@ip-10-0-11-43 hashlog]$ vim config/database.yml
...
production:
  <<: *default
  database: hashlog_production
  username: hashlog  # 削除
  password: <%= ENV['HASHLOG_DATABASE_PASSWORD'] %>  # 削除
# defaultではなくこっちの設定が上書きされてしまっていたので削除

[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec rails db:create RAILS_ENV=production
Created database 'hashlog_production'

[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec rails db:migrate RAILS_ENV=production
```

やったぜ。

https://www.atmarkit.co.jp/flinux/rensai/linuxtips/a115makeerror.html

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

# よく使うコマンド

## .bash_profile

```
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
export PATH=$HOME/.nodebrew/current/bin:$PATH
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"

# source ~/.nvm/nvm.sh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

if [ -f ~/.nvm/nvm.sh ]; then
        . ~/.nvm/nvm.sh
fi

# nvm use をターミナル起動時に実行する
nvm use "v14.2.0"

# yarn

export PATH="$PATH:`yarn global bin`"
```

## nginxとpumaの再起動

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ ps aux | grep puma
[USERNAME@ip-10-0-11-43 hashlog]$ kill -9 プロセス番号
[USERNAME@ip-10-0-11-43 hashlog]$ bundle exec puma -C config/puma/production.rb -e production -d
[USERNAME@ip-10-0-11-43 hashlog]$ sudo nginx -s stop
[USERNAME@ip-10-0-11-43 hashlog]$ sudo service nginx start
```

## rails c

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ RAILS_ENV=production rails c
```

### デーモン管理のコマンド

https://qiita.com/sinsengumi/items/24d726ec6c761fc75cc9

```shell
$ systemctl list-unit-files --type=service

```
