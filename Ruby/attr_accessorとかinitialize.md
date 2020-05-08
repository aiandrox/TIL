```
attr_accessor :name
↓
def name
  @name
end

def name=(val)
  @name = val
end
```

 
```
attr_reader :name
↓
def name
  @name
end
```

```
attr_writer :name
↓
def name=(val)
  @name = val
end
```

---

```
def initialize(name, age)
  @name = name
  @age = age
end
```

`User.new(name: 'NAME', age: 10)`とかで設定される、インスタンス生成時の初期値。増えてくると面倒になるとかなんとか……

```
def initialize(name, age)
  @name, @age = name, age
end
```

も可 
