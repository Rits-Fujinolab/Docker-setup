# Samba の設定
Linux server のファイルを Windows explorer から叩けるようになる．

## Server 構築時にやること
### Samba のインストール
```bash
sudo apt install -y samba
```
### smb.confの設定
/etc/samba/smb.comf 内の[global]行以下に追記
```
unix extensions = no
wide links = yes
```

### Samba の自動起動設定
```bash
sudo systemctl enable smbd nmbd
```


## User 追加時にやること
### Samba user の作成
この時，linux ユーザと同じ名前にしておく．
Pass も同じにしておくと無難．
(USERNAMEのところに自分の名前を入れてね！)
```bash
sudo smbpasswd -a USERNAME
```
### smb.conf の設定
/etc/samba/smb.comfの行末に追記する．   
[hogehoge] は任意，[share_{linux_username}]にしておくと無難です．
pathには共有したい場所を記入してください．   
大体の人は /home/{linux_username}だと思います．(echo $HOME で確認できる)
```
[hogehoge]
path = /home/name
writable = yes
printable = no
browseable = no
read only = no
```
### Samba の再起動
```bash
sudo systemctl restart smbd nmbd
```

## Windows explorer からのアクセス方法
Windows explorer の Address bar に以下を入力し，設定したSamba userでログインすることでアクセスが可能となる．
```bash
\\{linux serverのip address}\{任意の文字列(上の場合だとhogehoge)}
```
