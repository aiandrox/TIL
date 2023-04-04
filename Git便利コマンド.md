### grepしたリストからspecファイルのパスを取得

```sh
git grep -l '# 後で消す' | xargs -I {} sh -c 'echo {} | sed "s/app\//spec\//;s/\.rb$/_spec.rb/"'
```

検索時にディレクトリを除外

```sh
git grep -l '# 後で消す' -- ':^transformer'
```

### ファイルを取得して一括削除

```sh
git grep -l '# 後で消す' | xargs rm
```

### そのブランチにマージしたPRのタイトルを取得する

```sh
git log --merges --since=6am --first-parent --reverse --author=k_end --pretty=format:"- %b"
```

[今日マージされたプルリクエストのタイトル一覧を表示するgitコマンド \- Qiita](https://qiita.com/mishimay/items/baa23f3b8c2cd62cf817)
