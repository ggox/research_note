### fabric 折腾笔记

#### 环境安装

* 安装git：

  ```bash
  apt install git
  ```

* 安装curl：

  ```bash
  apt intall curl
  ```

* 安装docker：

  ```bash
  apt install docker.io
  ```

* 安装docker-compose：

  ```bash
  apt install docker-compose
  ```

* 安装golang：

  * 获取最新软件包源并添加至apt库：

    ```bash
    add-apt-repository ppa:longsleep/golang-backports
    ```

  * 更新：

    ```bash
    apt-get update
    ```

  * 安装最新版go：

    ```bash
    apt-get install golang-go
    ```

* 配置环境变量：

  ```bash
  export GOPATH=$HOME/go
  export GOROOT=/usr/local/go
  export PATH=$GOROOT/bin:$PATH
  ```

  

* 安装NVM与NPM

  1. ```bash
     curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash
     ```

  2. ```bash
     nvm install v8.11.1
     ```

* 安装Hyperledger Fabric

  * 新建目录并执行初始化脚本

    ```bash
    mkdir hyfa && cd hyfa
    vim bootstrap.sh
    chmod	+x bootstrap.sh
    ./bootstrap.sh 1.2.0
    ```

  * 添加环境变量：

    ````bash
    export PATH=$HOME/hyfa/fabric-samples/bin:$PATH
    ````

* 构建fabric网络环境：

  1. 使用脚本方式：byfn.sh
  2. 手动实现

* 测试网络环境：使用byfn.sh命令