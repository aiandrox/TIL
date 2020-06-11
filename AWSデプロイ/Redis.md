## redisのインストール

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

https://redis.io/download
