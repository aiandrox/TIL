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

- `joins`
  - inner_joinでくくる  
- `left_outer_joins`
  - left outer joinでくくる

  
## クラスメソッドとスコープの違い

`find_by`を使ったときのnilの挙動に違いがある。  
クラスメソッドだと`nil`になるが、スコープだとそのスコープをなかったことにして再度実行する。  
`where`だとnilではないので、どちらも`#<ActiveRecord::Relation []>`になる。

```rb
class Book < ApplicationRecord
  scope :written_about_scope, ->(theme) { find_by("title LIKE ?", "%#{theme}%") }

  def self.written_about_class(theme)
     find_by("title LIKE ?", "%#{theme}%")
  end
end
```

```rb
irb(main):016:0> Book.written_about_scope('ab')
  Book Load (0.4ms)  SELECT "books".* FROM "books" WHERE (title LIKE '%ab%') LIMIT ?  [["LIMIT", 1]]
  Book Load (0.1ms)  SELECT "books".* FROM "books" LIMIT ?  [["LIMIT", 11]]
=> #<ActiveRecord::Relation [#<Book id: 1, title: "aaのついての本", created_at: "2020-09-01 05:20:51", updated_at: "2020-09-01 05:20:51">]>

irb(main):017:0> Book.written_about_class('ab')
  Book Load (0.2ms)  SELECT "books".* FROM "books" WHERE (title LIKE '%ab%') LIMIT ?  [["LIMIT", 1]]
=> nil
```

`pluck`メソッドで必要なカラムの配列だけ取得できる。  
https://qiita.com/k-o-u/items/31e4a2f9f5d2a3c7867f

## コールバック

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

順番などは以下を参照  
https://qiita.com/rtoya/items/29cef3e328299781a328

## enum

`book.enum_column_before_type_cast`のように`_before_type_cast`を付けると、でDBに保存されている値（integer）を取得できる。

`Book.published`でwhere絞り込み検索、`Book.not_published`のようにnotを付けるとnot検索ができる。

enumerizeというgemもあるのでオススメ

## コントローラ

- before_action
- around_action
  - アクションの前後で実行
- after_action

`around_action`の場合、アクションを囲むような感じになるので、該当アクションを`yield`とする。

```rb
around_action :hoge

def hoge
  logger.info 'この後アクションがあるやで'
  yield
  logger.info 'アクションが実行されたやで'
end
```

## ビュー

### variantsによるテンプレート切り替え

```rb
class ApplicationController < ActionController::Base
  before_action :detect_mobile_variant
  
  private
  
  def detect_mobile_variant
    request.variant = :mobile if request.user_agent =~ /iPhine/
  end
end
```

この場合、UserAgentがiPhoneだったときは`show.html+mobile.erb`を表示する。

### ヘルパーメソッド

#### `url_for`

URLパスを文字列にする。

引数
- `root_path` => `"/"`
- `result_url` => `"http://www.example.com/result"`
- `controller: :hoge, action: :edit` => `"/hoge/edit"`  
- `controller: :hoge, action: :edit, id: 1234, detailed: 'true'` => `"/hoge/edit?detailed=true&id=1234"`

#### `time_ago_in_words`

現在時刻と引数の差をざっくり文字列として返す。  
i18nに対応する。

```rb
irb(main):010:0> helper.time_ago_in_words(Time.now)
=> "less than a minute"
irb(main):011:0> helper.time_ago_in_words(Time.now - 1.hours)
=> "about 1 hour"
irb(main):012:0> helper.time_ago_in_words(Time.now + 1.hours)
=> "about 1 hour"
irb(main):013:0> helper.time_ago_in_words(Time.now + 1.hours + 1.minutes)
=> "about 1 hour"
irb(main):014:0> helper.time_ago_in_words(Time.now + 1.minutes)
=> "1 minute"
```

#### `number_with_delimiter`

引数の数字を3桁ごとに`,`で区切った文字列を返す。  
delimiterで間の文字を指定することもできる（誰得？）。

```rb
irb(main):015:0> helper.number_with_delimiter(11123444321)
=> "11,123,444,321"
irb(main):016:0> helper.number_with_delimiter(11123444321, delimiter: '@')
=> "11@123@444@321"
irb(main):017:0> helper.number_with_delimiter(11123444321, delimiter: 'おりゃ')
=> "11おりゃ123おりゃ444おりゃ321"
```


### コンソールで呼び出すとき

- `app.result_path`
- `helper.time_ago_in_words`

#### 余談

特定のメソッドの引数を確認する。

```rb
irb(main):025:0> Book.method(:new).parameters
=> [[:opt, :attributes], [:block, :block]]
```

https://qiita.com/sonots/items/f18cf3ca8c76bfb4d6f0


### エスケープ処理

クロスサイトスクリプティング（XXS）を防ぐために、ただの文字列はHTMLにせずにエスケープするようになっている。

```
<%= raw "<script>alert('sample');</script>" %>
<%= "<script>alert('sample');</script>".html_safe %>
```

上記の処理によって`ActiveSupport::SafeBuffer`オブジェクトになり、jsが実行される。  
（rawメソッドは内部で`html_safe`を呼んでいる）

`ActiveSupport::SafeBuffer`は`String`のサブクラス。そして、安全な文字列という意味なので、これに対して`html_safe`をしてもエスケープ処理をしない。
つまり、何度同じ処理をしても多重エスケープの問題を起こさない！
