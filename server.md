# リモート共用サーバ(Ubuntu)を立ち上げ，SSHで接続する

## Ubuntuのインストール
- Keyboardレイアウト
	
	`日本語/日本語-日本語(OADG 109A)`

- ネットワーク接続

	- IP: 172.24.162.xxx
	- Mask: 255.255.255.128
	- Default GW: 172.24.162.254
	- DNS: 172.24.32.1, 172.26.100.1

- ソフトウェアインストール

	`$ sudo apt update`  
	`$ sudo apt upgrade`  
	`$ sudo apt install vim ntp ssh`  

- NTPサーバの設定

	`$ sudo vim /etc/ntp.conf`  
	以下のように書き換え
	```
	# Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
	# on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
	# more information.
	#pool 0.ubuntu.pool.ntp.org iburst
	#pool 1.ubuntu.pool.ntp.org iburst
	#pool 2.ubuntu.pool.ntp.org iburst
	#pool 3.ubuntu.pool.ntp.org iburst
	server ntp.ritsumei.ac.jp iburst
	```
	`$ sudo service ntp restart`  
	NTPサーバが更新されていることを確認  
	`$ ntpq -p`

- ユーザを追加し，Sudo権限をつける (Optional)

	`$ sudo adduser <user_name>`  
	`$ sudo gpasswd -a <user_name> sudo`


## ローカルPCでの作業

リモートPCへのログインに公開鍵認証を使用する．

不要な場合はスキップして，ユーザ名/パスワードでのログインとすることもできる．

1. Windowsキーを押し，検索窓に`cmd`と打ち込み，コマンドプロンプトを起動する．
1. `$ ssh-keygen`と打ち込み，実行する．
1. `Enter file in which to save the key (C:\Users\<ユーザ名>/.ssh/id_rsa):`は保存先を聞かれているので，基本的には何も書かずにEnterを押す．
1. `Enter passphrase (empty for no passphrase):`は秘密鍵をほどくためのパスフレーズを聞かれているので，任意のパスワードを設定する．何も入力せずにEnterを押すと省略できる(秘密鍵が暗号化されない)．
1. `Enter same passphrase again:`上記のパスワードを再入力する．何も入力しなかった場合はそのままEnterを押す．
1. `C:\Users\<ユーザ名>\.ssh\`に`id_rsa`(秘密鍵)と`id_rsa.pub`(公開鍵)が作成されている．

`C:\Users\<ユーザ名>\.ssh\`へのアクセスは，エクスプローラから`PC`->`ローカルディスク(C:)`->`ユーザー`->`<Windowsユーザ名>`->`.ssh`である．

コマンドプロンプトより以下の操作を行い，公開鍵をサーバにアップロードする．

```bash
$ # ----- ここからローカルPCのコマンドプロンプト ----- #
$ ssh <username>@<remote ip>  # 例: $ ssh fujinolab@172.24.162.xxx
$ # ----- ここからリモート接続先のPC ----- #
$ ls -all  # .ssh フォルダがあるか確認する
$ mkdir .ssh  # なければ作る
$ # Ctrl+D で切断する
$ # ----- ここからローカルPC ----- #
$ scp \Users\<ユーザ名>\.ssh\id_rsa.pub <username>@<remote ip>:~/.ssh/authorized_keys  # 公開鍵をauthorized_keysという名前でアップロード
```

SSH接続を試して，パスワードレスでログインできれば成功．

ただし，秘密鍵にパスフレーズを設定した場合，秘密鍵を復号するためのパスワードを聞かれるので注意．

```bash
$ ssh <username>@<remote ip>
```

## 