# デプロイのコマンド記録

## 環境

- Ruby 2.6.6
- Rails 5.2.4.3
- node 14.4.0
- MySQL 8.0.19（EC2は 15.1 Distrib 5.5.64-MariaDB）
- Bundler 2.1.4


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
[ec2-user@ip-10-0-11-43 ~]$ sudo useradd username

[ec2-user@ip-10-0-11-43 ~]$ sudo passwd username
ユーザー username のパスワードを変更。
新しいパスワード: # 自分で決める
新しいパスワードを再入力してください:
passwd: すべての認証トークンが正しく更新できました。

[ec2-user@ip-10-0-11-43 ~]$ sudo visudo

vimで
...
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
username  ALL=(ALL)       NOPASSWD: ALL # 追加
...

[ec2-user@ip-10-0-11-43 ~]$ cat ~/.ssh/authorized_keys
# コピー

[ec2-user@ip-10-0-11-43 ~]$ su - username
パスワード:

[username@ip-10-0-11-43 ~]$ mkdir .ssh
[username@ip-10-0-11-43 ~]$ chmod 700 .ssh
[username@ip-10-0-11-43 ~]$ touch ~/.ssh/authorized_keys
[username@ip-10-0-11-43 ~]$ chmod 600 ~/.ssh/authorized_keys
[username@ip-10-0-11-43 ~]$ vim ~/.ssh/authorized_keys
# ペースト
```

`$ ssh -v -i "aws-key.pem" username@IPアドレス`でEC2に入れるようになったよヤッター

## 初期設定インストール

```shell
[username@ip-10-0-11-43 ~]$ sudo yum install \
git make gcc-c++ patch \
openssl-devel \
libyaml-devel libffi-devel libicu-devel \
libxml2 libxslt libxml2-devel libxslt-devel \
zlib-devel readline-devel \
mysql mysql-server mysql-devel \
ImageMagick ImageMagick-devel \
epel-release

[username@ip-10-0-11-43 ~]$ sudo amazon-linux-extras install epel

[username@ip-10-0-11-43 ~]$ curl -L git.io/nodebrew | perl - setup
...
========================================
Export a path to nodebrew:

export PATH=$HOME/.nodebrew/current/bin:$PATH
========================================

[username@ip-10-0-11-43 ~]$ echo 'export PATH=$HOME/.nodebrew/current/bin:$PATH' >> ~/.bash_profile

[username@ip-10-0-11-43 ~]$ source ~/.bash_profile

[username@ip-10-0-11-43 ~]$ nodebrew install-binary latest
Fetching: https://nodejs.org/dist/v14.4.0/node-v14.4.0-linux-x64.tar.gz
######################################################################### 100.0%
Installed successfully

[username@ip-10-0-11-43 ~]$ nodebrew list
v14.4.0
current: none
# ローカルは14.2.0だからまあいいかな

[username@ip-10-0-11-43 ~]$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
[username@ip-10-0-11-43 ~]$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
[username@ip-10-0-11-43 ~]$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
[username@ip-10-0-11-43 ~]$ source ~/.bash_profile
[username@ip-10-0-11-43 ~]$ rbenv install -v 2.6.6  # ローカルに合わせる
# めっちゃ長い
[username@ip-10-0-11-43 ~]$ rbenv global 2.6.6
[username@ip-10-0-11-43 ~]$ rbenv rehash
[username@ip-10-0-11-43 ~]$ ruby -v
ruby 2.6.6p146 (2020-03-31 revision 67876) [x86_64-linux]
```

## GitHubとの連携、クローン

```shell
[username@ip-10-0-11-43 ~]$ cd .ssh  # さっき.sshディレクトリを作っているはず

[username@ip-10-0-11-43 .ssh]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa):  # エンター
Enter passphrase (empty for no passphrase):  # エンター
Enter same passphrase again:  # エンター
Your identification has been saved in /home/username/.ssh/id_rsa.
Your public key has been saved in /home/username/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ZCbswiAYHY9AohfuoT+TgHdG4IcCwpMkN0+mjQ4P4UU username@ip-10-0-11-43.ap-northeast-1.compute.internal
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

[username@ip-10-0-11-43 .ssh]$ cat id_rsa.pub
# コピーしてGitHubに登録

[username@ip-10-0-11-43 .ssh]$ cd /
[username@ip-10-0-11-43 /]$ sudo chown username var
[username@ip-10-0-11-43 var]$ sudo mkdir www
[username@ip-10-0-11-43 var]$ sudo chown username www
[username@ip-10-0-11-43 var]$ cd www
[username@ip-10-0-11-43 www]$ git clone git@github.com:username/hashlog.git
```

## Railsアプリの初期設定

```shell
[username@ip-10-0-11-43 www]$ cd hashlog/config/

[username@ip-10-0-11-43 config]$ vim master.key
# ローカルのmaster.keyをコピペ

[username@ip-10-0-11-43 config]$ vim database.yml

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

[username@ip-10-0-11-43 config]$ cd /var/www/hashlog/
[username@ip-10-0-11-43 hashlog]$ gem install bundler:2.1.4  # bundle installが通らなかったときに指示された。ローカルも2.1.4だった。
[username@ip-10-0-11-43 hashlog]$ bundle install
```

### MySQLの設定

```
[aiandrox@ip-10-0-11-43 hashlog]$ bundle exec rails db:create RAILS_ENV=production
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

[aiandrox@ip-10-0-11-43 hashlog]$ mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)

[aiandrox@ip-10-0-11-43 hashlog]$ cat /etc/my.cnf
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

[aiandrox@ip-10-0-11-43 hashlog]$ cd /var/lib

[aiandrox@ip-10-0-11-43 lib]$ ls -a
.             chrony    gssproxy       misc       postfix    stateless
..            cloud     hibinit-agent  mlocate    rpcbind    systemd
alternatives  dbus      initramfs      nfs        rpm        update-motd
amazon        dhclient  logrotate      os-prober  rpm-state  xfsdump
authconfig    games     machines       plymouth   rsyslog    yum

[aiandrox@ip-10-0-11-43 lib]$ mkdir mysql
mkdir: ディレクトリ `mysql' を作成できません: Permission denied

[aiandrox@ip-10-0-11-43 lib]$ sudo mkdir mysql
[aiandrox@ip-10-0-11-43 lib]$ cd mysql/
[aiandrox@ip-10-0-11-43 mysql]$ sudo touch mysql.sock

[aiandrox@ip-10-0-11-43 hashlog]$ mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (13)

```
