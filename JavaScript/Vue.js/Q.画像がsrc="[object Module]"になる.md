url-loaderとかfile-loaderの設定がうまく出来ていればいいが、そうでない場合

無理な場合は相対パスでこんな風に書けば行ける  
`<img :src="require('../images/logo.png').default"/>`

https://github.com/vuejs/vue-loader/issues/1612
