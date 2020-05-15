## 問題

axiosのレスポンスを `console.log()` で表示した時  
`.then` のレスポンスはうまくオブジェクトとして取れるのに  
`.catch` のレスポンスは下記のように表示されてしまう

```
Error: Request failed with status code 422
    at createError (createError.js:16)
    at settle (settle.js:17)
    at XMLHttpRequest.handleLoad (xhr.js:59)
```

## 結論

catchのerrorオブジェクト？が文字列として表示されてしまっている。  
なのでerror.responseと指定すればOK

```javascript
axios.post('/test/create', this.formData)
  .then(response => { 
	console.log(response)
  })
  .catch(error => {
    console.log(error.response)
  });
```

or 

```javascript
axios.post('/test/create', this.formData)
  .then(response => { 
	console.log(response)
  })
  .catch(({response}) => {
    console.log(response)
  });
```


### 参考
https://katuo-ai.com/axios-error/
https://github.com/axios/axios/issues/960

https://qiita.com/HorikawaTokiya/items/a18d59c864d1d1e1baf1
