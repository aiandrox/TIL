```js
12.toString()
VM49617:1 Uncaught SyntaxError: Invalid or unexpected token

const n = 12
-> undefined
n.toString()
-> "12"
(12).toString()
-> "12"
```

`12`の段階だとプリミティブ型だから`toString`メソッドを持たない
手動でオブジェクトにする必要がある。
