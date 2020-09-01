知らなかったことを書いていく。

# 1章

**binstub**  
`bundle exec`を付けなくてもいい実行ファイルのこと。`./bin`内に入っている。

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
  - `rails runner 'puts Rails.env'`, `rails runner hoge.rb`てな感じで実行できる。

- `rails dbconsole`, `rails db`  
  - データベースの中に入る。

# 2章

## Query Interface

https://railsguides.jp/active_record_querying.html

「インターフェイス」は何かと何かをつなぐ間という概念。  
ここではSQLのクエリとRailsActiveRecordの間。

ActiveRecord::Relationに対して使うことのできるメソッドは例えば以下のものがある。

- **joins**
  - inner_joinでくくる  
- **left_outer_joins**
  - left outer joinでくくる

  
## クラスメソッドとスコープの違い

`find_by`を使ったときのnilの挙動に違いがある。  
クラスメソッドだと`nil`になるが、スコープだとそのスコープをなかったことにして再度探索する。

```rb
class Book < ApplicationRecord
  scope :written_about, ->(theme) { find_by("title LIKE ?", "%#{theme}%") }

  def self.written(theme)
     find_by("title LIKE ?", "%#{theme}%")
  end
end
```

```rb
irb(main):015:0> Book.written('ab')
  Book Load (0.2ms)  SELECT "books".* FROM "books" WHERE (title LIKE '%ab%') LIMIT ?  [["LIMIT", 1]]
=> nil
irb(main):016:0> Book.written_about('ab')
  Book Load (0.4ms)  SELECT "books".* FROM "books" WHERE (title LIKE '%ab%') LIMIT ?  [["LIMIT", 1]]
  Book Load (0.1ms)  SELECT "books".* FROM "books" LIMIT ?  [["LIMIT", 11]]
=> #<ActiveRecord::Relation [#<Book id: 1, title: "aaのついての本", created_at: "2020-09-01 05:20:51", updated_at: "2020-09-01 05:20:51">]>
```

`pluck`メソッドで必要なカラムの配列だけ取得できる。  
https://qiita.com/k-o-u/items/31e4a2f9f5d2a3c7867f

| コールバックポイント | create | update | destroy |
| :--------------| :------| :------| :-------|
| before_validation | ○ | ○ | × |
| after_validation  | ○ | ○ | × |
| before_save |  ○ | ○ | × |
| around_save | ○ | ○ | × |
| after_save  | ○ | ○ | × |
| before_create | ○ | × | × |
| around_create | ○ | × | × |
| after_create  | ○ | × | × |
| before_update | × | ○ | × |
| around_update | × | ○ | × |
| after_update  | × | ○ | × |
| before_destroy| × | × | ○ |
| around_destroy| × | × | ○ |
| after_destroy | × | × | ○ |

- `after_initialize`
  - モデルがインスタンス化された後に呼び出される。  
- `after_find`
  - `first`, `last`, `find`, `find_by`によってインスタンス化した後に呼び出される。

https://qiita.com/rtoya/items/29cef3e328299781a328


