## CSRFトークン

RailsにGET以外のリクエストを送るときはCSRFトークンが必要。
static_pages/topでヘッダーに埋め込まれている。

```js
document.getElementsByName('csrf-token')[0].getAttribute('content')
document.querySelector('meta[name="csrf-token"]').getAttribute('content')
```

どっちも同じ。ヘッダーから取ってきてる
