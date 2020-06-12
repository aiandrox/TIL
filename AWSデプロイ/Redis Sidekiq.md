## redisのインストール（以前していたよ？？）

結局やめた。

```
wget http://download.redis.io/releases/redis-6.0.5.tar.gz
tar xzf redis-6.0.5.tar.gz
cd redis-6.0.5
make
中に入って色々やってから出る。

[aiandrox@ip-10-0-11-43 redis-6.0.5]$ src/redis-server
8717:C 11 Jun 2020 17:43:27.704 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
8717:C 11 Jun 2020 17:43:27.704 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=8717, just started
8717:C 11 Jun 2020 17:43:27.704 # Warning: no config file specified, using the default config. In order to specify a config file use src/redis-server /path/to/redis.conf
8717:M 11 Jun 2020 17:43:27.705 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
8717:M 11 Jun 2020 17:43:27.705 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
8717:M 11 Jun 2020 17:43:27.705 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
8717:M 11 Jun 2020 17:43:27.705 # Could not create server TCP listening socket *:6379: bind: Address already in use
```

```shell
[aiandrox@ip-10-0-11-43 ~]$ sudo mkdir -p /var/lib/redis/6379
[aiandrox@ip-10-0-11-43 ~]$ mkdir -p /var/log/redis
[aiandrox@ip-10-0-11-43 ~]$ sudo mkdir -p /etc/redis
[aiandrox@ip-10-0-11-43 ~]$ sudo chown -R aiandrox /var/log/redis /var/lib/redis
[aiandrox@ip-10-0-11-43 redis]$ sudo vi /etc/redis/6379.conf
# 新規ファイルに以下の記述
daemonize no
port 6379
save 900 1
save 300 10
save 60 10000
dir /var/lib/redis/6379/
dbfilename dump.rdb
logfile /var/log/redis/redis_6379.log
supervised systemd 
```

```shell
[aiandrox@ip-10-0-11-43 redis]$ sudo vim /etc/systemd/system/redis.service
# 以下を記述
[Unit]
Description=Redis

[Service]
Type=notify
ExecStart=/usr/local/bin/redis-server /etc/redis/6379.conf
ExecStop=/usr/local/bin/redis-cli -p 6379 shutdown
User=redis
Group=redis

[Install]
WantedBy=multi-user.target

[aiandrox@ip-10-0-11-43 ~]$ sudo systemctl daemon-reload

[aiandrox@ip-10-0-11-43 ~]$ systemctl status redis.service
● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since 火 2020-06-09 21:32:04 JST; 2 days ago
 Main PID: 3653 (redis-server)
   Status: "Ready to accept connections"
   CGroup: /system.slice/redis.service
           └─3653 /usr/bin/redis-server 127.0.0.1:6379

[aiandrox@ip-10-0-11-43 ~]$ sudo reboot
           
[aiandrox@ip-10-0-11-43 ~]$ systemctl --failed
  UNIT           LOAD   ACTIVE SUB    DESCRIPTION
● mysqld.service loaded failed failed MySQL Server
● redis.service  loaded failed failed Redis persistent key-value database

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

2 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```

```shell
[aiandrox@ip-10-0-11-43 ~]$ ~/redis-6.0.5/src/redis-server
3791:C 12 Jun 2020 06:59:09.625 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
3791:C 12 Jun 2020 06:59:09.625 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=3791, just started
3791:C 12 Jun 2020 06:59:09.625 # Warning: no config file specified, using the default config. In order to specify a config file use redis-6.0.5/src/redis-server /path/to/redis.conf
3791:M 12 Jun 2020 06:59:09.626 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
3791:M 12 Jun 2020 06:59:09.626 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
3791:M 12 Jun 2020 06:59:09.626 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.0.5 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 3791
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

3791:M 12 Jun 2020 06:59:09.628 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
3791:M 12 Jun 2020 06:59:09.628 # Server initialized
3791:M 12 Jun 2020 06:59:09.628 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
3791:M 12 Jun 2020 06:59:09.628 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
3791:M 12 Jun 2020 06:59:09.628 * Ready to accept connections

# ctr+cで終了。デーモンとして起動はできていない……
```

```shell
vim /etc/systemd/system/redis.service.d/limit.conf

以下の記述がしてあった。


# If you need to change max open file limit
# for example, when you change maxclient in configuration
# you can change the LimitNOFILE value below.
# See "man systemd.exec" for more information.

# Slave nodes on large system may take lot of time to start.
# You may need to uncomment TimeoutStartSec and TimeoutStopSec
# directives below and raise their value.
# See "man systemd.service" for more information.

[Service]
LimitNOFILE=10240
#TimeoutStartSec=90s
#TimeoutStopSec=90s
```

https://yomon.hatenablog.com/entry/2016/02/20/162648
https://redis.io/download


### なかったことにしました。

```shell
[aiandrox@ip-10-0-11-43 ~]$ rm -rf redis-6.0.5
[aiandrox@ip-10-0-11-43 ~]$ rm -rf redis-6.0.5.tar.gz
```

他のファイルは残ったままですが、、、

### AWSに作る

[![Image from Gyazo](https://i.gyazo.com/83d49b4133e858e850bdff8ec3c9774a.gif)](https://gyazo.com/83d49b4133e858e850bdff8ec3c9774a)

画像の設定 + セキュリティグループは`hashlog-redis-sg`を作成した。設定は以下の記事を参照。  
https://qiita.com/mt2/items/af916608dfb8c4fc72c8

実際はEC2のwebのセキュリティグループからのアクセスにしたほうがいい。
[![Image from Gyazo](https://i.gyazo.com/2bde2a55350d552b78cb2c65e148a3d3.png)](https://gyazo.com/2bde2a55350d552b78cb2c65e148a3d3)


**プライマリエンドポイント:クラスターのプライマリエンドポイント**をURLに設定する。

```rb
# config/initializers/sidekiq.rb

Sidekiq.configure_server do |config|
  config.redis = { url: Settings[:sidekiq][:url], namespace: 'sidekiq' }
end

Sidekiq.configure_client do |config|
  config.redis = { url: Settings[:sidekiq][:url], namespace: 'sidekiq' }
end
```

```
# config/settings/production.rb

sidekiq:
  url: 'redis://hashlog-redis.fxhyxf.ng.0001.apne1.cache.amazonaws.com:6379'
```
## sidekiqのデーモン化

```
[aiandrox@ip-10-0-11-43 hashlog]$ bundle exec sidekiq --environment production -d --logfile log/sidekiq.log
WARNING: Daemonization mode will be removed in Sidekiq 6.0, see #4045. Please use a proper process supervisor to start and manage your services
WARNING: Logfile redirection will be removed in Sidekiq 6.0, see #4045. Sidekiq will only log to STDOUT
```
