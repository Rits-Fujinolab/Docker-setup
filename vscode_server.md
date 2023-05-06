# VSCodeでリモートサーバのDockerコンテナに接続する

## 拡張機能をインストールする

- `Remote Development (ms-vscode-remote.vscode-remote-extensionpack)`
- `Docker (ms-azuretools.vscode-docker)`

## SSHの設定をする

VSCodeにてコマンドパレットから`Remote-SSH: Open SSH Configuration File`を実行し，以下を追記。

```ini
Host <識別名>
HostName <Remote PCのIPアドレス>
User <ユーザ名>
IdentityFile <Local PCに保存したSSH_key(秘密鍵ファイル)の絶対パス>
IdentitiesOnly yes
```

`IdentityFile`，`ItentitiesOnly`は[公開鍵の設定](./adduser.md)を行った場合のみ．

※ GitHubのSSHの関係でconfigファイルを作成している場合はそこに追記しても良い

## Dockerの設定をする

1. Remote PCにVSCodeの`Remote-SSH:Connect to Host`で接続する
2. Remote PC上で動作するVSCode serverに拡張機能`Docker (ms-azuretools.vscode-docker)`をインストールする
3. Remote PC上でDockerコマンドがsudoつけなくても動くようにする [[参考](https://docs.docker.com/engine/install/linux-postinstall/)]
4. 設定が終わったらDockerデーモンを再起動し，ターミナルを開き直す  
   `sudo systemctl restart docker`

※ Rootless Docker推奨だが後のdevcontainerで対応可能になるので上記の通りとする

ターミナルを開き，下のコマンドで実行を確認する

```bash
docker run hello-world
```

もし，これでもsudoユーザが必要と言われたら，PCを再起動する

## Devcontainerの設定をする

### 概要

Devcontainerについて起動の手順はどらぜみにて解説した．[[参考](https://tinyurl.com/2ajmuv37)]  
ここでは (VSCodeユーザ以外を見捨てない) Devcontainerに対応する`Dockerfile`と`docker-compose.yaml`，そして`devcontainer.json`について説明する．  
今回は例として，NVIDIA-TensorflowをJupyter Notebookで実行する環境を通して，各々のファイルの例とその配置について見る．  
Devcontainerは大前提としてVSCodeユーザ向けである点に注意．  
(Devcontainer CLIが良い感じになったら更新してください)  
なお利用するメリット・デメリットについては割愛するが，とにかく同一UID・GIDを持った一般ユーザで実行できる点が強み．(rootユーザ・ダメ絶対)．

### 各ファイルの配置について

ファイルと<フォルダ>については以下の通り.

```txt
<project folder>
├── <.devcontainer>
│   └── devcontainer.json
├── Dockerfile
├── docker-compose.yaml
├── requirements.txt
└── <workspace>
    ├── tf.code-workspace
    ├── <datasets>
    └── sample.ipynb
```

### Dockerfileについて

まず`Dockerfile`の例を次に示す．  

```Dockerfile
FROM nvcr.io/nvidia/tensorflow:23.03-tf2-py3

RUN useradd -m -d /home/fujinolab -s /bin/bash fujinolab
WORKDIR /home/fujinolab/workspace

COPY requirements.txt /home/fujinolab/workspace
RUN pip install --upgrade pip &&\
  pip install black flake8 isort &&\
  pip install -r requirements.txt
```

特筆すべき点は`RUN useradd`と`WORKDIR`で，前者にてコンテナ内に一般ユーザを作成し，そのホームディレクトリを作業フォルダとしている．  
これにより作成されるファイルやマウント時に権限を弄る必要が無くなる．  
注意点として一部のイメージでは初めから一般ユーザを設定しているものがあるので，その場合はそのユーザをそのまま使えばよい．  
(例えば，SCAのどらぜみで使用したJupyter公式イメージでは，一般ユーザ "Jovyan" が予め用意されている)

### docker-compose.yamlについて

続いて`docker-compose.yaml`の例を次に示す．  

```yaml
version: "3.8"

services:
  main:
    # Names
    image: tf-gpu
    container_name: tf-container
    hostname: tf-server
    # Property
    build: .
    ports:
      - '8888:8888'
    ipc: host
    ulimits:
      memlock: -1
      stack: -1
    shm_size: 16g
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: [ '0' ]
              capabilities: [ gpu ]
    volumes:
      - type: bind
        source: ./workspace
        target: /home/fujinolab/workspace
    working_dir: /home/fujinolab/workspace
    tty: true
    stdin_open: true
```

冒頭にもある通り既存のcompose設定を生かせるのが強みで，一般的なcomposeファイルと同じでよい．  
特筆すべき点としては`working_dir`以下の設定で，作業ディレクトリは一致させておく必要がある．  
`Names`の部分は同一のGPUサーバを利用する場合でも共通で問題ない (はず)．  
途中の`devives_id`はGPUサーバを割り当てられた場合に弄る必要があり, 特に指定しないと全てのGPUをマウントする．  
ボリュームは基本バインドマウントを使用すること．(よく用いられる省略記法と同じ)

### devcontainer.jsonについて

今回の本命`devcontainer.json`の例を以下に示す．

```json
{
  "name": "tf-gpu",
  "dockerComposeFile": [
    "../docker-compose.yaml"
  ],
  "service": "main",
  "workspaceFolder": "/home/fujinolab/workspace/tf.code-workspace",
  "remoteUser": "fujinolab",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.flake8",
        "ms-python.isort"
      ]
    }
  }
}
```

詳細はネット検索に任せるが，`workspaceFolder`はworkspaceフォルダ内のcode-workspaceにすると便利である．  
特に次の[おまけ](#おまけ-tfcode-workspaceについて)に示すリンター設定などを共通化できるので，フォーマットを統一することができる．  
また`extensions`ではコンテナ内での拡張機能を自動でインストールすることができ，環境構築を楽にしてくれる．  

### おまけ: tf.code-workspaceについて

VSCodeの設定はほとんどが好みによるものだが，一つの例として次に示す．  

```json
{
  "folders": [
    {
      "path": "."
    }
  ],
  "settings": {
    "python.formatting.provider": "black", // フォーマッターをblackに設定
    "python.linting.pylintEnabled": false, // pyLintをオフに
    "python.linting.mypyEnabled": false, // mypyは好みで
    "python.linting.flake8Enabled": true, // flake8をオンに
    "python.linting.enabled": true, // リンターを有効に
    "python.linting.lintOnSave": true, // 保存時にリンターが効くように
    "editor.formatOnSave": true, // 保存時にフォーマットをオンに
    "editor.formatOnType": true, // タイピング時にフォーマットをオンに
    "editor.formatOnPaste": false, // 貼り付け時のフォーマットはオフに
    "python.terminal.activateEnvironment": false,
    "python.formatting.blackArgs": [
      "--line-length",
      "120"
    ],
    "python.linting.flake8Args": [
      "--max-line-length",
      "120",
      "--extend-ignore",
      "E203, F403, F405, E741, E731, W503, E402, W504, E501",
    ],
  }
}

```
