```js
const array = ['なんたらかんたら', 'うんたらかんたら', 'どうたらこうたら'];
array.join('\n');
```
の出力は

```
なんたらかんたら
うんたらかんたら
どうたらこうたら
```

になる。

```js
array.map(s => '・' + s).join('\n')
```

```
・なんたらかんたら
・うんたらかんたら
・どうたらこうたら
```
