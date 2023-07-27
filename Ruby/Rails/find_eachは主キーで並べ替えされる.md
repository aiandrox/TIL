```rb
users = User.order(:name).pluck(:name).limit(3)
users.pluck(:name)
=> ["aa", "bb", "cc"]

users.find_each do |user|
  p user.name
end
"bb"
"dd"
"ff"

users.order(:id).pluck(:name)
["bb", "dd", "ff", "cc", "aa"]
```

！！！？？？と思ったら、`find_each`は主キーで並べ替えて勝手にやるらしい。。。


https://github.com/rails/rails/blob/984c3ef2775781d47efa9f541ce570daa2434a80/activerecord/lib/active_record/relation/batches.rb#L40

