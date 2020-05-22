```
FactoryBot.define do
  factory :registered_tag do
    user
    tag
    sequence(:created_at) { |n| Time.current + n.minute }
  end
end
```

CIさんは結構作成速度ガバいから、落ちるときは指定する！
