## 別のモデルに`errors.messages`を持たせる

```
registered_tag.errors.messages
=> {:tag_id=>["はすでに存在します"]}
```
のとき
```
tag.errors.messages.merge!(registered_tag.errors.messages)
```
によって`tag`に同じエラーメッセージを渡せる。

```
tag.errors.messages
=> {:tag_id=>["はすでに存在します"]}
```

## i18nの対応

このままだと、`tag.errors.full_messages`が日本語化されずに`["Tagはすでに存在します"]`になる。  
`registered_tag.errors.full_messages`は`["ハッシュタグはすでに存在します"]`になるけど。

i18nはこのとき`tag.tag_id`を参照するので、
```
ja:
  activerecord:
    attributes:
      tag:
        tag_id: ハッシュタグ
```
こうしないといけない。
