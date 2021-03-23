```sh
bundle exec rake hoge:foo[91811]
zsh: no matches found: hoge:foo[91811]
```

でも本番では使えているっぽい。。なぜ〜〜〜〜？？  
と思ったら、zshではこうしないといけないんだった。

```sh
bundle exec rake 'hoge:foo[91811]'
```
