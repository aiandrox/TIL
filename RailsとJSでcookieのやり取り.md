Rails側で登録
```rb
cookies[:logged_in] = { value: 1, expires: 3.minute.from_now }
```

JS側で削除
```js
document.cookie = "logged_in=;path=/;max-age=0;"
```
