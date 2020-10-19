```rb
  def previous_charge
    @previous_charge ||= Charge.find_by(
      plan_id: plan_id,
      user_id: user_id,
    )
  end
```

`@previous_charge`がnil→再度取得しに行くからダメ！！！

```rb
  def previous_charge
    return @previous_charge if defined?(@previous_charge)
    @previous_charge = Charge.find_by(
      plan_id: plan_id,
      user_id: user_id,
    )
  end
```
