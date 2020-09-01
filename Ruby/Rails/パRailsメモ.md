# 1章

**binstub**  
`bundle exec`を付けなくてもいい。`./bin`内に実行ファイルが入っている。こういうファイルのこと。

```
$ rails stats
+----------------------+--------+--------+---------+---------+-----+-------+
| Name                 |  Lines |    LOC | Classes | Methods | M/C | LOC/M |
+----------------------+--------+--------+---------+---------+-----+-------+
| Controllers          |    556 |    297 |      21 |      34 |   1 |     6 |
| Helpers              |      2 |      2 |       0 |       0 |   0 |     0 |
| Models               |    222 |    174 |       6 |      20 |   3 |     6 |
| Mailers              |      4 |      4 |       1 |       0 |   0 |     0 |
| Channels             |      8 |      8 |       2 |       0 |   0 |     0 |
| Libraries            |     16 |     15 |       0 |       0 |   0 |     0 |
| Loyalty specs        |    275 |    257 |       0 |       0 |   0 |     0 |
| Model specs          |    618 |    586 |       0 |       0 |   0 |     0 |
| Request specs        |   1117 |   1105 |       0 |       0 |   0 |     0 |
| Serializer specs     |     29 |     29 |       0 |       0 |   0 |     0 |
| Service specs        |    201 |    195 |       0 |       0 |   0 |     0 |
+----------------------+--------+--------+---------+---------+-----+-------+
| Total                |   3048 |   2672 |      30 |      54 |   1 |    47 |
+----------------------+--------+--------+---------+---------+-----+-------+
  Code LOC: 500     Test LOC: 2172     Code to Test Ratio: 1:4.3
```

- `rails runner`  
`rails runner 'puts Rails.env'`, `rails runner hoge.rb`てな感じで実行できる。

- `rails dbconsole`, `rails db`  
データベースの中に入る。

# 2章

## Query Interface

https://railsguides.jp/active_record_querying.html

「インターフェイス」は何かと何かをつなぐ間という概念。  
ここではSQLのクエリとRailsActiveRecordの間。

ActiveRecord::Relationに対して使うことのできるメソッドは例えば以下のものがある。

- **joins**
  inner_joinでくくる  
- **left_outer_joins**
  left outer joinでくくる

  
