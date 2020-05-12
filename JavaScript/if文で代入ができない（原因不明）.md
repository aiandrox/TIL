```
if (!!remindDay === false) {
  const editedRemindDay = 0
} else {
  const editedRemindDay = this.filter(remindDay)
}
```
↓
```
const editedRemindDay = !!remindDay === false ? 0 : this.filter(remindDay)
```

で代入ができた。
