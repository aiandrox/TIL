Proc今見たらなんとなくわかる気がしたので、自分なりの理解

ブロック`{これ}とかdo end`をオブジェクトにする=変数に入れられる！後から呼び出せる嬉しい

```ruby
p = Proc.new {|s| s.to_s}
p = Proc.new({|s| s.to_s})  # この書き方はダメ
```

ブロックごとProcオブジェクトにする
```ruby
[:aaa, :bbb, :ccc].map(&p)
```

`&変数`で`変数`を`to_proc`してから実行するという意味  
pは既にProcオブジェクトなので、to_procしても同じなんだけど

```ruby
[:aaa, :bbb, :ccc].map(&to_s)
```

期待する`to_s`は変数ではなくメソッドだけど、変数`to_s`をProc化しようとするので「そんなもんねーよ」案件。かと思ったら、変数としてはなんか定義されてた。

```ruby
[7] pry(main)> [:aaa, :bbb, :ccc].map(&to_s)
TypeError: wrong argument type String (expected Proc)
from (pry):7:in `<main>'
[8] pry(main)> to_s
=> "main"
```
変数`to_s`ってStringなんですね（なんだこれどこで定義されてんだ）

```ruby
irb(main):003:0> :to_s.to_proc
=> #<Proc:0x00007f84b99a9300(&:to_s)>
```

シンボルに対して`to_proc`をすると同じ名前のProcオブジェクトを返す

閑話休題

```ruby
irb(main):006:0> p = Proc.new{|s| s.to_s}
=> #<Proc:0x00007f84b992db88@(irb):6>
irb(main):007:0> p.call(123)
=> "123"
```

Procオブジェクトに対して`call(引数)`をすると、`|s|`に引数を渡してProcを実行することになる

つまり

```ruby
[1, 2, 3].map(&:to_s)
```
は、シンボル`:to_s`を`&`の効果でProc化（シンボルだからto_procすると同じ名前のProcオブジェクト（つまりto_sメソッド）を呼び出す）して実行するという意味


https://qiita.com/kasei-san/items/0392097791d3a5998216  
https://docs.ruby-lang.org/ja/2.2.0/doc/spec=2fcall.html#block  
