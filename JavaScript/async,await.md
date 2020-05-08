AxiosはPromissオブジェクトで返ってきてくれる。
チェーンな書き方

```
axios.get(url)
  .then(response => {
    // 正常系
})
  .catch(response => {
    // 異常系
})
  .finally(() => {
    // どちらにせよ
})
```

直列で実行したいときってどういうこと？<br>
APIを叩いて200レスポンスだったらさらに別のAPI叩くみたいなこと。

async/await

```
qsync function methodKorewoYobuyo() {
  try {
    const res = await axios.get(url);
    // 取得したresを使ってデーターほげほげする
  } catch (error) {
    // 例外処理
  }
}
```

使うときには`methodKorewoYobuyo()`として実行する。

```
def hoge
  # 処理
rescue StandardError => e
  # 例外処理
end
```

っぽい記法。
