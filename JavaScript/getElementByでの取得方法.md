```js
document.getElementsByClassName('add-form')
HTMLCollection [form.add-form]

document.getElementsByClassName('add-form').insertAdjacentHTML
undefined

document.getElementById('add-form')
<form id="add-form" class="add-form">...</form>
```

classは複数あるからHTMLCollectionになってしまう。ので、`insertAdjacentHTML`が引っ張れない。<br>
idにすれば問題ない（idが一致するものはページに一つしかないので）

```js
document.getElementsByClassName('add-form')[0]
<form id="add-form" class="add-form">​…​</form>
```

`[0]`を付けて順番を指定したら、ちゃんと取得できた！
