## カラムの追加

`$ rails generate migration AddPartNumberToProducts part_number:string`  
**Addカラム名Toテーブル名**

```
class AddPartNumberToProducts < ActiveRecord::Migration[5.0]
  def change
    add_column :products, :part_number, :string
    add_column :テーブル名, :カラム名, :型
  end
end
```

## カラムの削除

`$ rails generate migration RemovePartNumberFromProducts part_number:string`  
**Removeカラム名Fromテーブル名**

```
class RemovePartNumberFromProducts < ActiveRecord::Migration[5.0]
  def change
    remove_column :products, :part_number, :string
    remove_column :テーブル名, :カラム名, :型
  end
end
```
rollbackできるようにするためにremveでも型を指定する。


## カラム名の変更

```
class RenamePiblishedColumnToBooks < ActiveRecord::Migration
  def change
    rename_column :books, :titre, :title
    rename_column :テーブル名, :元のカラム名, :新しいカラム名
  end
end
```


## インデックス、リファレンスを追加

`$ rails generate migration AddPartNumberToProducts part_number:string:index user:references`

```
class AddPartNumberToProducts < ActiveRecord::Migration[5.0]
  def change
    add_column :products, :part_number, :string
    add_index :products, :part_number
    add_reference :products, :user, foreign_key: true
  end
end
```


## その他

外部キー
```
add_foreign_key :articles, :authors
```
