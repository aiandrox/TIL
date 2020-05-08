```
<div id="app">
  // 文字列を渡す場合は""の中で''で囲む
  <my-component :message="'hoge'"></my-component>
  <my-component :message="'fuga'"></my-component>
  <my-component :message="'hogefuga'"></my-component>
</div>
<script>
  Vue.component("my-component", {
    props: ["message"],
    template: "<li>{{message}}</li>"
  })
</script>
```
