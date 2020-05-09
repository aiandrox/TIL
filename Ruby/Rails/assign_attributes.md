## `assign_attributes`

```
  def update
    user = User.find(params[:id])
    user.assign_attributes(user_params)
    user.privacy = User.provacies_i18n.invert(user_params[:privacy])
    if user.save
      render user
    else
      error = {
        'status' => '422',
        'title' => '登録内容が適切ではありません。',
        'detail' => '登録内容を確認してください。'
      }
      render json: { 'errors': [error] }, status: 422
    end
  end
```

`new(user_params)`と同じ感覚で、インスタンスにデータを入れるだけで保存はしていない状態になる。<br>
ストロングパラメータ以外の値を入れるときに使う。
