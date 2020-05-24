```js
const numbers = [1, 2, 3, 4, 5];

numbers.find(function(number) {
  return number < 3;
})

numbers.find((number) => {
  return number < 3;
})

numbers.find(number => {
  return number < 3;
})

numbers.find(number => number < 3); 
```

これ全部一緒！！<br>
厳密には少し違う。`this`の扱い。
