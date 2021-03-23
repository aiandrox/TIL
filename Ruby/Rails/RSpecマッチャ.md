```rb
expect(HogeService).to receive(:call).with(
  message: include("メッセージの一部"),
  hoge: 'foo',
)
```

こんな書き方できるのかあ。

あとこれが便利そうだった。  
[RSpecの標準matchers\(マッチャー\)一覧 \| 酒と涙とRubyとRailsと](https://morizyun.github.io/blog/rspec-builtin-matcher-rails/index.html)
