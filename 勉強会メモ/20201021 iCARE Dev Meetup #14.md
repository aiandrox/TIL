https://icare.connpass.com/event/189356/

## 「GraphQL-Rubyの実戦投入から見る利便性」	yotuba_eng

APLLOってaxiosポジ？？  
GitHubのAPIがGraphQL

エンドポイントをいちいち作らなくていい
クライアント側の自由度が高い。欲しい情報だけ取れる

```
query {
  currentUser {
    email
    name
  }
}
```

ユーザー権限はどうする？？  
外部に出したくない情報

- Query　取得
　- visible?（Fieldを制限する）, authorized?（アクセス自体を制限）
　- クエリ自体を解析→条件によってraise（深さとかsize）
- Mutation　変更
　- 制限: ready?
- Subscription　変化検知


## 「Railsアプリの脆弱性パターン」	sinsoku

よい  
https://www.slideshare.net/ockeghem/ruby-on-rails-security-142250872

`current_user.articles.find(params[:article_id]).tags`  
（別解）手作業でnot_foundを返すようにする

テーブルのカラム数がバレるだけでもやばいのかあ  
jsonはシリアライズ必須（onlyを付けるとか。ホワイトリストにしたいよな）

filter_parameter_loggingでログに残すカラムを指定できる  
Sentryとかの外部サービスに対してもfilter推奨

「メールアドレスが存在しません」フィッシングに利用されかねない　ブルーとフォース攻撃  
↓  
「メールアドレスかパスワードが間違えています」

失敗回数（redis）でロック

パスワードスプレー（ドコモ？のやつ）  
パス固定でメールアドレスを変える→回数で制限できない！

入力スパム  
→display_noneが入ってるときに弾く（人は入力しないから）


## 「ActiveSupportのマイナーだけど便利なメソッド」	Issei Nakamura

```
1.in?([1,3])
=> true
1.in?([2,3])
=> false

[1,3].include?(1)
だっけ？

123456.to_s(;delimited)
=> '123,456'
:currency, :human_sizeなど

'hogehoge'.remove('ho')
=> 'gege'
gsubいらず

truncate_words(3)
truncate_words(3, separator: /[、。]./) +だったかも

inquery?
よくわからん
roleで判断？？

squish
戦闘、末尾、連続したスペースや改行を削除
生SQLなど

index_by
ブロックの値をkeyにする
重複に注意

'apple'.presence_in(['apple', 'banana'])
many?
2子以上あるか
ブロックの条件にあうものにたいしても使える
rubyではone?がある

deep_merge
なんとなく……わかる？

pluck
指定したkeyの配列
ActiveRecordのArray版

multiple_of?
引数の倍数かどうか
12.multiple_of?(3)
=> true

in_milliseconds
1.day.in_milliseconds

1.ordinalize
=> '1st'

定数とかも定義されている
１ヶ月の秒数とか
```
