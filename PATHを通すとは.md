## PATHを通すとは？

```
$ vim .zshrc

export PATH="$HOME/.rbenv/shims:$PATH"
```

`.zshrc`にこういう記述がある。`.zshrc`や`.zprofile`（zsh）`.bashrc`や`.bash_profile`（bash）に書くことがパスを通すということ。

```
$ cd ~/.rbenv/shims
$ ls
略
faker                rake                 update_rubygems
```

いろいろある。これの中身を実行している。

Q. 直打ちでもコマンドが実行できるのか？？

```
$ ~/.rbenv/shims/rake
rake aborted!
No Rakefile found (looking for: rakefile, Rakefile, rakefile.rb, Rakefile.rb)

(See full trace by running task with --trace)
```

できた！！
