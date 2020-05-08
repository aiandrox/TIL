```
def initialize(name, age)
  @name = name
  @age = age
end
```

`User.new(name: 'NAME', age: 10)`とかで設定される、インスタンス生成時の初期値。増えてくると面倒になるとかなんとか……

```
def initialize(name, age)
  @name, @age = name, age
end
```

も可。

```
class Client
  def initialize(user, tag_name)
    @user = user
    @tag_name = tag_name
  end
end
```

だと`Client.new(user, 'ハッシュタグ')`じゃないとデータ格納できない。<br>
`Client.new(user: user, tag_name: 'ハッシュタグ')`ってしたかったら

```
def initialize(user:, tag_name:)
  @user = user
  @tag_name = tag_name
end
```
↑<br>
これはキーワード引数。<br>

これが必要＝引数が多くなっているということなので、先にリファクタした方がいい。
