コールバックを使わないパターン

```rb
FactoryBot.define do
  factory :article do
    user
    plans { create_list(:plan, 3) }
  end
end
```

これだとCIでは落ちる。
