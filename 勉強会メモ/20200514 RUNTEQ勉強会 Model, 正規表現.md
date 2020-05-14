## Modelに入れるもの入れないもの

helperのメソッドはグローバルなので、メソッド名が被ってしまうことがある  
なので、helperのメソッドの命名規則注意

あれ、helperってmodule？？

```
# app/helpers/application_helper.rb
module ApplicationHelper
end
```

そうだったーーーーー！！！！

### ActiveSupport::Concern

大きいプロジェクトだと危ない。結局モジュールだから、名前がかぶる可能性

- `git grep`で確認する
- 汎用的になりすぎない命名。一意になるようにする


## Rubyの正規表現活用例

`sub`  最初のものだけ置き換え

`gsub`  すべて置換

```
str.sub(A, B)  # AをBに置換
```

`scan`  合致するものを抜き出して配列に格納する


```
.scan(/@([0-9]+)+/)

irb(main):001:0> "@144 and @150".scan(/@([0-9]+)+/)
=> [["144"], ["150"]]
```

`()`によってさらに内側を取得できる。配列がネストされるけども
