## Vue.js 3.0
### コンポジションAPI

コンポーネントにsetupメソッドが追加される。propsとcontextが引数。  
attrs, slots, parent, root, listeners, emitとかをsetupで用意する  
なんか書き方がTSっぽい？型を指定している感じがする  
初期化する、というのがキーワードっぽい  

値洋服している場合はミックスインで共通化するのが定石だった。これは名前空間のデメリットがあるから結局コピペになったりしている  
→コンポジションAPIで解決できる！！

まずは別で共通化。setとかを準備しておく  
setupで共通化した「動作部分のコード」を取ってこれる  
あとは値を渡して値をwatchするだけ

注意：setupのときのみ使用できる＝onMountedとかでは使えない？？？実例がイメージできないのでちょっとわからん。  
setupで呼び出した後、そのものが消えたり変化することはないよってことっぽい

conposition function: useなんとかという命名をしよう

setupフック呼び出し後にref, reactiveは作成可  
呼び出した後コンポーネントに後付けできるって意味かな？でもあんまりよくないっぽい？  
見た目と動き・データで別々に切り分けて、まとめてコンポーネントとしてexportしてるのか？

## Rails 6.1の新機能

https://speakerdeck.com/willnet/rails6-dot-1dexin-sikuru-ruji-neng-nituite

- viewでrenderがHTMLで見れる？これちょっとよくわからんかった
- routesを別ファイルに切り出し可能になった。
- トークン付きのidを付けるメソッド signed_id
  - パスワードリセットなどで使える
- strict loadingを使うと、強制eager loadさせる（してないとエラーが出る）
  - アソシエーションでデフォルトでこれをonにもできる
  - モデル単位でも可能（俺を呼ぶときはstrict loadingでよろしく）
  - strict_loading(false)で外すこともできる
  - 関連モデルのオブジェクトを無駄に作成するデメリットも
- 複数DBのsharding対応（恩恵はよくわからん）
- Deprecation warningで例外を発生させる機能
  - 指定方法は色々ある（スライド参照）
- ActiveStrageでアップロードしたファイルに直接アクセスできるURLを使える
  - 元々はリダイレクトだったんだ知らんかった
- サムネの管理方法
  - サムネが作成されているかどうかを監理するテーブルを作るので、一々ファイルを参照しに行かない
- ActiveModel::Errorオブジェクトができた（ActiveModel::ErrorsはErrorの集合になる）
  - これは直感的に書けて嬉しい。ハッシュは操作しづらかった
- クエリメソッドmissingの追加
  - アソシエーション先が1件もないとき
- enumのdefaultを設定できるオプション（アツい）

他にもいい機能たくさん！
