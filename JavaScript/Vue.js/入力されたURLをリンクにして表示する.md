## URL置き換えの処理

```
<template>
  <div
    class="body-1"
    style="white-space: pre-line;word-wrap: break-word"
    v-html="$sanitize(replacedDescription)"
  />
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      default: () => {}
    }
  },
  computed: {
    replacedDescription() {
      const url = new RegExp("(https?://[a-zA-Z0-9!-/:-@¥[-`{-~]*)")
      const replaced = this.user.description.replace(
        url,
        "<a href='$1' target='_blank'>$1</a>"
      )
      return replaced
    }
  }
}
</script>
```

- 正規表現は`()`でくくらないと`$1`で抽出できない

ネットにあったURLの正規表現

```
https?://[\w!?/\+\-_~=;\.,*&@#$%\(\)\'\[\]]+
```

## サニタイズ

application.js
```
import Vue from "vue"
import App from "../App.vue"
import sanitizeHTML from "sanitize-html"
Vue.prototype.$sanitize = sanitizeHTML

document.addEventListener("DOMContentLoaded", () => {
  const app = new Vue({
    el: "#app",
    router,
    vuetify,
    render: h => h(App)
  })
  app.$mount("#app")
})
```

https://qiita.com/tnemotox/items/b4b8f0f627e23dd62447

## ちなみに。改行反映させるやつ

```
style="white-space: pre-line;word-wrap: break-word"
```

https://qiita.com/koji77/items/435c2410d9fd7d622f4c
