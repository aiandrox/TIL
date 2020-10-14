```rb
    rescue ActiveRecord::RecordInvalid => e
      errors.add(e.record.class.to_s.downcase, e.record.errors.full_messages.join)
      false
    end
```
