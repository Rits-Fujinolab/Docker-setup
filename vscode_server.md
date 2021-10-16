# VSCodeでリモートサーバのDockerコンテナに接続する

## 拡張機能をインストールする
	- `Remote Development (ms-vscode-remote.vscode-remote-extensionpack)`
	- `Docker (ms-azuretools.vscode-docker)`

## SSHの設定をする
	`Remote-SSH: Open SSH Configuration File`

	以下を追記
	```
	Host <識別名>
	HostName <Remote PCのIPアドレス>
	User <ユーザ名>
	IdentityFile <Local PCに保存したSSH_key(秘密鍵ファイル)の絶対パス>
	IdentitiesOnly yes
	```	

    `IdentityFile`，`ItentitiesOnly`は[公開鍵の設定](./adduser.md)を行った場合のみ．

## Dockerの設定をする

- VSCodeの設定をする (Local PC)
	`settings -> Docker.host`に`ssh://<user_name>@<Remote IP address>`

- Remote PCにVSCodeの`Remote-SSH:Connect to Host`で接続する

- Remote PC上で動作するVSCode serverに拡張機能`Docker (ms-azuretools.vscode-docker)`をインストールする

- Remote PC上でDockerコマンドがSudoつけなくても動くようにする

確認する
```
$ docker run hello-world
```

もし，これでもSudoユーザが必要と言われたら，PCを再起動する


- これで，Remote-SSHでログインした状態でDocker拡張機能が使えるようになった．
