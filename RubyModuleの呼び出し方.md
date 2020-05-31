## module_function
- このメソッドは外から呼び出してもOKという登録のようなイメージ
- 呼び出す全てを定義する必要がある
- rails consoleで`ModuleName.method_name`で呼び出せた

## include
- 丸ごと取り込むイメージ
- なので、切り分けるときもそのまま持っていけた


`module_function`を使うと、`include`していなくても`ModuleName.method_name`で呼び出せる。  
ただし、ネスト？している場合は、使用する全てのメソッドを`module_function`の対象とする必要がある。privateに置いていても呼び出し不可

これは別にActiveConcern関係ないやつ
