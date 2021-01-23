```rb
Answer.includes([:user, { review: [:evaluation, :reviewer, :approver] }, { question: [:subject, { course: :certificate }] }])
```

こんな感じ。
