# ユーザの作成からDockerの設定まで

Windows PCからUbuntuサーバへのリモート接続は，Windows PowerShellを起動して以下のコマンドでできる．

```bash
ssh <ユーザ名>@<サーバIPアドレス>
```

## ユーザを作成する

### サーバ側のターミナルで作業

Sudo権限のある管理者ユーザでログインし，ユーザの追加を行う．

```bash
# ユーザの追加
sudo adduser <ユーザ名>
# sudo/dockerグループに追加
sudo usermod -aG sudo <ユーザ名>
sudo usermod -aG docker <ユーザ名>
sudo newgrp docker
```

一度ログアウトし，新たに作成したユーザでログインする．

```bash
# 正常にバージョンが表示されればOK
# permission deniedになる場合，再起動を試す．
docker --version
```

## SSH接続でサーバにログインできるようにする (公開鍵をサーバに登録する)

### Windows側で作業

現状でもユーザ名/パスワードの認証でSSH接続はできるが，作業性のために公開鍵でログインできるようにする．

公開鍵ペアの作成がまだの場合は，[Git環境の構築](https://rits-fujinolab.github.io/Github_rules/howto/git_setup.html)を参照してWindows上で公開鍵ペアの作成を行う．

Windows PowerShellを起動する．
```bash
cd ~
cd .ssh
# ファイル一覧から公開鍵(通常はid_rsa.pub)が存在することを確認する．
ls
# サーバにアップロードする
scp id_rsa.pub <ユーザ名>@<サーバIPアドレス>:/home/<ユーザ名>/.ssh/tmp
# -----
# ここで，
# /home/<ユーザ名>/.ssh No such file or directory
# と表示される場合は.sshフォルダが存在しないので，SSHでサーバに一度ログインして作成する．
# $ ssh <ユーザ名>@<IPアドレス>
# $ cd ~
# $ mkdir .ssh
# 終わったらもう一度上記のscpコマンドを試す．
# -----
# サーバに一度SSHでログインする．(ここではまだパスワードを聞かれる)
ssh <ユーザ名>@<サーバIPアドレス>
# 公開鍵をauthorized_keysに追加する
cat ~/.ssh/tmp >> ~/.ssh/authorized_keys
# 追加できたら元のファイルを消す
rm ~/.ssh/tmp
# Ctrl + Dでログアウトする．
# もう一度SSHでログインを試みて，パスワードを要求されなければ公開鍵の登録は完了．
ssh <ユーザ名>@<サーバIPアドレス>
```