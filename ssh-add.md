`ssh-add ~/.ssh/id_rsa`

秘密鍵をssh-agentに登録する。  
別サーバーAに入るときにはそのssh-agentを連れていけば、そのサーバーAに秘密鍵を置かなくても、Aから他のサーバー（例えばGitHub）へ間接的にssh接続できる。

https://qiita.com/naoki_mochizuki/items/93ee2643a4c6ab0a20f5
