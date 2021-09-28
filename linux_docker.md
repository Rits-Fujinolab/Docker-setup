# Ubuntu上にDocker環境を構築する

[Nvidia container toolkitのドキュメント](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)に従ってインストールを進める

アップデートによって手順が変更されている可能性があるため，ソースの確認を推奨．

## Dockerを入れる

	[公式の手順](https://docs.docker.com/engine/install/)に従ってインストールする
	
	手順例 (実際にインストールする場合は，公式をちゃんと参照すること)
	```
	$ sudo apt-get update
	$ sudo apt-get install \
	    apt-transport-https \
	    ca-certificates \
	    curl \
	    gnupg \
	    lsb-release
	$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
	$ echo \
	  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
	  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	$ sudo apt-get update
	$ sudo apt-get install docker-ce docker-ce-cli containerd.io
	```

	確認する  
	`$ docker --version`

## Docker-composeを入れる

	[公式の手順](https://docs.docker.com/compose/install/)に従ってインストールを進める

	手順例 (実際にインストールする場合は，公式をちゃんと参照すること)
	```
	$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
	$ sudo chmod +x /usr/local/bin/docker-compose
	$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
	```

	確認する  
	`$ docker-compose --version`

## Nvidia-container-toolkitを入れる (GPU搭載サーバのみ)

	[公式の手順](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)に従ってインストールを進める

	手順例  
	```
	$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
	   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
	   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
	$ curl -s -L https://nvidia.github.io/nvidia-container-runtime/experimental/$distribution/nvidia-container-runtime.list | sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
	$ sudo apt-get update
	$ sudo apt-get install -y nvidia-docker2
	$ sudo systemctl restart docker
	```

	確認する  
	`$ sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi`