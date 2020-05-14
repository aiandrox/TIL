## フォーム外の値による動的なバリデーション

``:rules="`${isRemind ? 'remindDay|maxRemindDay' : ''}`"``のような書き方で条件によってバリデーションを付けることができる。

サンプルコード

```
  <v-checkbox
    v-model="isRemind"
  />
  <validation-provider
    v-slot="{ errors }"
    :rules="`${isRemind ? 'remindDay|maxRemindDay' : ''}`"
  >
    <v-text-field
      v-show="isRemind"
      v-model="registeredTag.remindDay"
      :error-messages="errors"
    />
  </validation-provider>
```
