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

### mariaDBをインストールしてアンインストールした

```shell
[USERNAME@ip-10-0-11-43 /]$ sudo yum install mariadb-server
読み込んだプラグイン:extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                               | 3.7 kB     00:00
amzn2extra-docker                                        | 3.0 kB     00:00
amzn2extra-epel                                          | 1.7 kB     00:00
epel/x86_64/metalink                                     | 8.2 kB     00:00
192 packages excluded due to repository priority protections
...
インストール:
  mariadb-server.x86_64 1:5.5.64-1.amzn2

依存性関連をインストールしました:
  perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.amzn2.0.2
  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.amzn2.0.2
  perl-DBD-MySQL.x86_64 0:4.023-6.amzn2
  perl-DBI.x86_64 0:1.627-4.amzn2.0.2
  perl-Data-Dumper.x86_64 0:2.145-3.amzn2.0.2
  perl-IO-Compress.noarch 0:2.061-2.amzn2
  perl-Net-Daemon.noarch 0:0.48-5.amzn2
  perl-PlRPC.noarch 0:0.2020-14.amzn2

完了しました!

[USERNAME@ip-10-0-11-43 /]$ sudo systemctl start mariadb
Job for mariadb.service failed because the control process exited with error code. See "systemctl status mariadb.service" and "journalctl -xe" for details.

[USERNAME@ip-10-0-11-43 /]$ systemctl status mariadb.service
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since 月 2020-06-08 22:04:11 UTC; 19s ago
  Process: 22582 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=1/FAILURE)

[USERNAME@ip-10-0-11-43 mysql]$ journalctl -xe

-- Logs begin at 月 2020-06-08 13:00:01 UTC, end at 月 2020-06-08 13:06:25 UTC.
 6月 08 13:00:01 ip-10-0-11-43.ap-northeast-1.compute.internal su[12488]: pam_unix(su:auth): authentication failure; logname=ec2-user uid=1001 euid=0 tty=pts/1 ruser=USERNAME rhost=  user=ec2-user
 6月 08 13:00:03 ip-10-0-11-43.ap-northeast-1.compute.internal su[12488]: FAILED SU (to ec2-user) ec2-user on pts/1
 6月 08 13:06:25 ip-10-0-11-43.ap-northeast-1.compute.internal sshd[12641]: Received disconnect from 202.208.137.136 port 62766:11: disconnected by user
 6月 08 13:06:25 ip-10-0-11-43.ap-northeast-1.compute.internal sshd[12641]: Disconnected from 202.208.137.136 port 62766

[USERNAME@ip-10-0-11-43 ~]$ sudo yum remove mariadb-server

[USERNAME@ip-10-0-11-43 ~]$ sudo yum remove mariadb-libs -y
読み込んだプラグイン:extras_suggestions, langpacks, priorities, update-motd
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ mariadb-libs.x86_64 1:5.5.64-1.amzn2 を 削除
...
削除しました:
  mariadb-libs.x86_64 1:5.5.64-1.amzn2

依存性の削除をしました:
  mariadb.x86_64 1:5.5.64-1.amzn2        mariadb-devel.x86_64 1:5.5.64-1.amzn2
  perl-DBD-MySQL.x86_64 0:4.023-6.amzn2  postfix.x86_64 2:2.10.1-6.amzn2.0.3

完了しました!
```

### mysql-serverのインストール

```shell
[USERNAME@ip-10-0-11-43 hashlog]$ mysql.server start
-bash: mysql.server: コマンドが見つかりません
[USERNAME@ip-10-0-11-43 ~]$ systemctl list-unit-files --type=service | grep mysql
# なにもない
```

https://dev.mysql.com/downloads/repo/yum/  
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

[aiandrox@ip-10-0-11-43 hashlog]$ vim config/database.yml
...
production:
  <<: *default
  database: hashlog_production
  username: hashlog  # 削除
  password: <%= ENV['HASHLOG_DATABASE_PASSWORD'] %>  # 削除
# こっちが適用されてしまっていたので

[aiandrox@ip-10-0-11-43 hashlog]$ bundle exec rails db:create RAILS_ENV=production
Created database 'hashlog_production'

[aiandrox@ip-10-0-11-43 hashlog]$ bundle exec rails db:migrate RAILS_ENV=production
```

やったぜ。

https://www.atmarkit.co.jp/flinux/rensai/linuxtips/a115makeerror.html



### デーモン管理のコマンド

https://qiita.com/sinsengumi/items/24d726ec6c761fc75cc9

```shell
$ systemctl list-unit-files --type=service
```

https://qiita.com/ymasaoka/items/7dc131dc98ba10a39854  
https://blog.apar.jp/linux/9868/
