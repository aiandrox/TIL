orderで並べると`NULL`が一番上にきちゃう。

```rb
eager_load(:category).order(Arel.sql('category_id IS NULL, categories.position'), :position)
```
