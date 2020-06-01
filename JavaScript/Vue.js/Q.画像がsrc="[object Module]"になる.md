### Rails + Vue + Webpacker

`<img :src="../images/logo.png"/>`だと`src="[object Module]"`になる。  
url-loaderとかfile-loaderの設定がうまく出来ていればちゃんと読み込めるのだけど……。


1. 絶対パスなら書けるので、`<img :src="/images/logo.png"/>`にして、画像を/public/images配下に置く。

2. 無理な場合は相対パスでこんな風に書けば行ける  
`<img :src="require('../images/logo.png').default"/>`

https://qiita.com/coppieee/items/4260bd0af246aebf5557
https://github.com/vuejs/vue-loader/issues/1612
