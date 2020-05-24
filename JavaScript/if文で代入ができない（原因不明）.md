```js
if (!!remindDay === false) {
  const editedRemindDay = 0
} else {
  const editedRemindDay = this.filter(remindDay)
}
```

<img width="728" alt="スクリーンショット 2020-05-12 20 52 14" src="https://user-images.githubusercontent.com/44717752/81685138-cafdfb00-9492-11ea-8e0e-b12b1a9bb077.png">

↓

```js
const editedRemindDay = !!remindDay === false ? 0 : this.filter(remindDay)
```

で代入ができた。
