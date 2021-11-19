```rb
redirect_to root_path, notice: "ほげほげ"
```
↓
```rb
redirect_back(fallback_location: root_path, notice: "ほげほげ")
```

[redirect\_back \| Railsドキュメント](https://railsdoc.com/page/redirect_back)
