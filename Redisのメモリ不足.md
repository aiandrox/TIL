```エラーログ
Enqueued HogeJob (Job ID: c48f466f-20aa-4b5e-a8b7-ac7e2bf32d68) to Sidekiq(default) with arguments: "User", 74272
Redis::CommandError: MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
from /Users/k-endo/sight-visit/shikaku-rails/vendor/bundle/ruby/2.7.0/gems/redis-3.3.5/lib/redis/client.rb:199:in `call_pipelined'
```

一旦ログを消すみたいなやり方↓

```sh
brew services stop redis
brew services start redis
```
