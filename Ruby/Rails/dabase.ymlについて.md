database.ymlの`host: localhost`の記述って「ローカルのMySQLに接続するよ！」って意味か。

`$ mysql -u usernameに設定したroot -h hostに設定したlocalhost -p （あれば）password`を実行して接続しているのと同じ感じ

```database.yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  charset: utf8mb4
  collation: utf8mb4_unicode_ci
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password:
  host: localhost
```
