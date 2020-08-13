【あっているかわからない】

Q. サーバーがauth_tokenを返すのは、ユーザーIDを返すんじゃだめ？  
JWTにユーザーIDを格納している  
JWTはrevokeできない＝サーバー側で無効化できないから、あまり重要な情報は入れたくない  
ユーザーIDを使わない目的としては、auth_tokenを変えることで無効化するため？？（できるかは知らんが）

A. ユーザーIDではないのは、推測できないようにしたかったからという考え。  
IDがuuidじゃなかったから

Q. APIの戻り値としてはuser.auth_tokenと有効期限をJWT化したものを格納したjsonを返却するはauth_tokenに対する有効期限？  
A. これはJWTの有効期限

リフレッシュトークンを使って有効期限を再設定するのがセオリーぽい  
→あれそれJWTに入れてたら、リフレッシュトークン取られときに永久的に取られるからまずくね？  
→それってクライアントでも保持しないといけない

トークンはBearerヘッダに入れる  
Wifiつなぎ直したとき

has_oneは新しいものを作ったら上書きされる仕様！？  
 dependent: :destroyをつけたらいけるっぽい。これがなかったら「先に消せ」ってエラーが出るっぽい。newした時点で消される？？？

AWSのコグニトを使うと二段階認証が楽らしい。[アンプリファイのチュートリアル](http://educationhub-31789470-a146-11ea-85be-f18c4f5a36d8.s3-website-us-east-1.amazonaws.com/)で使ってるらしい

Q. ログイン情報の管理をJWTで監理するメリットは？

A. ログイン情報をステートレスで管理できることかな？同じサービスでサーバーが増えたときに、sessionだとログイン情報が消える。  
JWTにログインしてますよ情報が入っているから、サーバーに依存しない。まあデータベース？に一々検証しに行くんだけど、サーバーが増えてもいいよー  
JWTはステートレス、の意味はredisにデータを照合せずに済むって意味らしい

Q. あれ、そもそもログイン情報ってサーバーで持ってなくね？  
redisに送っているデータ「〇〇サーバーのこの名前＆ユーザーIDがログインしてるやで」

[SorceryとJWTを使用したRails基本API認証](https://translate.google.com/translate?hl=ja&sl=en&tl=ja&u=http%3A%2F%2Ftangosource.com%2Fblog%2Frails-basic-api-authentication-with-sorcery-and-jwt%2F)
https://creators-note.chatwork.com/entry/2018/09/25/132218
