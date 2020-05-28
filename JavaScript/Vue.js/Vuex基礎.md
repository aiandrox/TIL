**プロパティスタイルアクセス**
```js
gettersNoMethod: (state, getters) => {
```
第二引数がgettersなんやね

```js
    getters: {
      // doneTodosを定義してください
      doneTodos: state => {
        return state.todos.filter(todo => todo.done)
      },
略
    computed: {
      // doneTodosを実行してください
      doneTodos () {
        return this.$store.getters.doneTodos
      },
```
なんでアロー関数と混ぜてるの？  
`doneTodos(state) { `でもちゃんと表示されるのに、なんで統一しない？？

**メソッドスタイルアクセス**というやつで使うかららしい。
```js
getTodoById: (state) => (id) => {
```
プロパティスタイルアクセスではできないらしい 


## getters

```js
   computed: {
     ...mapGetters(['doneTodos'])
   }
```
`this.doneTodos`と、あたかも「うちのcomputedですけど？」みたいな呼び出し方ができる。  
引数が必要な場合は別途computedを定義してその中で$storeのgettersを`this.getTodoById(2)`みたいな感じで呼び出して使う（これはちょっと二度手間感はある）


## なんか似ている

```js
 actions: {
  search_frameworks({ commit, state }, val) {
```
```js
getters {
  method: (state, getters) => {
```

## スプレッド演算子の書き方

```js
computed: {
  ...mapStates({ message: 'message', name: 'name' }), // コンポーネント側の変数: 'store.state側'
  ...mapGetters(['doneTodos', 'hogeTodo'])
},
methods: {
  ...mapMutations(['increment']),
  ...mapActions(['search_frameworks']) // 試してない＆書いてなかったけど、多分こういう書き方をするはず
}
```

> `dispatch` Actionsを実行（API叩いたり）  
`commit` Mutationsを実行（stateの更新）  
`plugin` 初期時に読み込む  
`subcribe` mutation, stateを引数にもつ

```js
store.dispatch('a/increment')
store.commit('a/increment')
store.getters['a/doneTodos']
```
