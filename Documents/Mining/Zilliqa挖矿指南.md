# 挖矿
Zilliqa官方挖矿指南（中文版）率先发布，由FireStack团队翻译。


# 挖矿指南

## 一般信息

欢迎来到代号为猫山王的Zilliqa测试网络-v3。我们邀请所有矿工成为公共节点，加入到猫山王的测试网络中。希望这次能让大家熟悉工作流程，并帮助我们在2019年1月底之前发现主网上线之前的潜在漏洞。我们还鼓励所有社区开发人员加入猫山王测试网，以便更好地了解Zilliqa的网络架构。

 - [推荐的硬件要求](#%E7%8C%AB%E5%B1%B1%E7%8E%8B%E6%B5%8B%E8%AF%95%E7%BD%91%E7%9A%84%E7%A1%AC%E4%BB%B6%E8%A6%81%E6%B1%82)
 - [使用docker挖矿的步骤](#%E4%BD%BF%E7%94%A8docker%E6%8C%96%E7%9F%BF%E7%9A%84%E6%AD%A5%E9%AA%A4%E4%BB%85%E9%80%82%E7%94%A8%E4%BA%8Ecpu%E6%88%96nvidia-gpu)
 - [本地挖矿的步骤](#%E6%9C%AC%E5%9C%B0%E6%8C%96%E7%9F%BF%E7%9A%84%E6%AD%A5%E9%AA%A4)



#### 测试网络难度

猫山王测试网的自助最低难度等级为3。此难度级别是动态的，并根据竞争加入Zilliqa网络的节点数量进行调整。

> 注意：难度级别是log2（难度）。


#### 测试网络Epoch架构

![测试网络 Epoch](https://github.com/FireStack2018/Awesome-Zilliqa/blob/master/Documents/img/Testnet%20Epoch.png)

在每个DS Epoch开始时，所有候选人都将运行工作证明（Ethash算法）过程`300`秒窗口，以便竞争加入Zilliqa网络。

然后，满足`DS_POW_DIFFICULTY`参数的节点将能够作为DS节点加入。同时，满足`POW_DIFFICULTY`参数的节点将作为分片节点加入。

每个DS Epoch（约1.5小时）内总共有`100`个TX时期（每个~1分钟）。第100个TX时期被称为**Vacuous时期**。

> 上图描绘了Zilliqa主网时期的架构。对于猫山王测试网络，我们在每个DS纪元中包含100个TX块用于测试。
>

空白时期会处理coinbase交易（奖励机制）、升级机制（因为pBFT中没有分支）和持久状态存储（写入节点的DB而不是仅存储在内存中）。在这个时期，网络不会处理任何常规交易。



#### 奖励机制

在Zilliqa网络中，奖励基于DS时期内节点完成的签名数量。由分片和DS节点提交的签名将获得相同的奖励。奖励被整合为一个DS时期，并在空白时期期间给出。

例如，如果Zilliqa网络中总共有`1,200`个节点，并且每个DS Epoch的`COINBASE_REWARD`设置为`10,000,000` ZIL，则每个签名分配的奖励将是：
```
10,000,000 /（1,200 * 2/3 [成功签名者] * 99 [TX块]）= 每个签名126.262626262626263 ZIL
```


## 猫山王测试网的硬件要求

目前，挖矿仅适用于Ubuntu 16.04 OS。如果您希望双启动Windows和Ubuntu 16.04，请按照[此处](https://itsfoss.com/install-ubuntu-1404-dual-boot-mode-windows-8-81-uefi/)的步骤操作。

我们目前支持AMD（使用OpenCL）和Nvidia（使用CUDA）GPU。

Zilliqa挖矿节点的建议要求是：

 -  x64 Linux操作系统，如Ubuntu 16.04.5
 -  英特尔i5处理器或更高版本
 -  8GB DRR3 RAM或更高
 -  **（可选）** 任何至少具有20 Mh / s的GPU卡[例如1 x GTX 1060,3GB专用RAM]

#### 对于OpenCL

如果您希望使用支持OpenCL的GPU进行PoW，请运行以下代码来安装OpenCL开发人员包。:
```
sudo apt install ocl-icd-opencl-dev
```

#### 对于CUDA

如果您希望使用支持CUDA的GPU进行PoW，请从[NVIDIA官网](https://developer.nvidia.com/cuda-downloads)下载与安装CUDA软件包。您可能需要重新启动PC才能使安装生效。

#### 对于多GPU

如果您有多个OpenCL或CUDA GPU，它们可以同时运作。请编辑位于“join”文件夹中的*constants.xml*文件中的`GPU_TO_USE`参数，以选择您希望使用的GPU数量。

索引从`0`开始，您可以选择一个或多个GPU。例如，1个GPU为`0`，3个GPU为`0,1,2`或`0,2,4`。确保最大的索引与您在挖矿设备中物理上的GPU数量相对应。



## 使用docker挖矿的步骤（仅适用于CPU或Nvidia GPU）

1. 按照以下说明安装Ubuntu 16.04.5操作系统：

   <http://releases.ubuntu.com/xenial/>

---

2. 按照以下说明为Ubuntu安装Docker CE：

   <https://docs.docker.com/install/linux/docker-ce/ubuntu/>

---

3. （可选）[如上所述](#%E5%AF%B9%E4%BA%8Ecuda)安装Nvidia CUDA驱动程序。如果使用CPU进行挖矿，则可以跳过此步骤。

---

4. 在桌面中创建一个新目录并将目录更改为：

   ```
   cd ~/Desktop && mkdir join && cd join
   ```

---

5. 在命令提示符下获取docker镜像：

   ```
   wget https://testnet-join.zilliqa.com/configuration.tar.gz
   tar zxvf configuration.tar.gz
   ```

---

6. 如果您在NAT环境中分别使用**选项1a**或**选项1b**，则启用UPnP**或**执行单端口转发。
   否则，如果您已经有公开的IP地址，请使用**选项2**查找当前的公共IP地址。

   > 注意：如果您使用的是家用路由器，则很可能是在NAT环境中并且可以启用UPnP。但是，如果UPnP不起作用，则可以执行端口转发。

   - **（选项1a）** 在家用路由器上启用UPnP模式。请谷歌你的家庭路由器设置，[这里](#%E5%AF%B9%E4%BA%8E%E5%A4%9Agpu)可以找到一个例子。您可以通过安装以下工具来检查是否已启用UPnP：

     ```
     sudo apt-get install miniupnpc
     ```

     然后在命令行后输入：

     ```
     upnpc -s
     ```

     您将收到一条消息 "List of UPNP devices found on the network :"**或**"No IGD UPnP Device found on the network !"。前者意味着UPnP模式已成功启用，而后者意味着UPnP模式存在问题。如果您属于后一种情况，请参阅下面的**选项1b**。

   - **（选项1b）** 单端口在路由器菜单中转发本地计算机IP。您可以在路由器菜单的TCP / UDP协议中同时将`30303`设置为外部端口（端口范围），`30303`设置为内部端口（本地端口），您可以在[此处](https://www.linksys.com/us/support-article?articleNum=136711)找到示例。然后，您可以使用命令提示符找出您的路由器IP地址：

     ```
     curl https://ipinfo.io/ip
     ```

   - **（选项2）** 如果您的命令提示符中已有公共IP地址，请直接查找您的公共IP地址：

     ```
     curl https://ipinfo.io/ip
     ```

---

7. 在命令提示符下运行shell脚本以启动docker镜像。

   - **（选项1）** 用于CPU挖矿：

     ```
     ./launch_docker.sh
     ```

   - **（选项2）** 对于Nvidia GPU挖矿，请先下载[nvidia-docker](https://github.com/NVIDIA/nvidia-docker)，然后：

     ```
     ./launch_docker.sh --cuda
     ```

     > 注意：如果您希望同时运行多个Nvidia GPU，则需要按照[此处](#%E5%AF%B9%E4%BA%8E%E5%A4%9Agpu)的说明修改*constants.xml*文件。

     > 注意：不幸的是，没有直接支持这种针对AMD GPU的docker构建。我们建议您按照以下[说明](#%E6%9C%AC%E5%9C%B0%E6%8C%96%E7%9F%BF%E7%9A%84%E6%AD%A5%E9%AA%A4)在本地构建Zilliqa，而不是使用docker。

---

8. 然后系统将提示您输入一些信息，如下所示：

   - `Assign a name to your container (default: zilliqa):`[*如果使用默认值，请按**Enter**跳过*]

   - `Enter your IP address ('NAT' or *.*.*.*):`[*键入**NAT**或您在步骤6中找到的公共IP地址*]

   - `Enter your listening port (default: 30303):`[*如果使用默认值，请按**Enter**跳过*]

---

9. 你现在是猫山王测试网络的一名矿工。您可以使用以下方法监控进度：

   ```
   tail -f zilliqa-00001-log.txt
   ```

   当您成为网络中的分片/ DS节点时，如果您设法在DS纪元开始时赢得PoW进程，您将在日志中收到通知。


---

10. 要检查本地生成的公钥和私钥对，可以在命令提示符中输入：

    ```
    less mykey.txt
    ```

    第一个十六进制字符串是您的**公钥**，第二个十六进制字符串是您的**私钥**。

    > 注意：密钥对是在磁盘上本地生成的。务必记住将私钥保存在安全的地方！

---

11. 停止通过 docker 挖矿，[DOCKER NAME]是你的 Docker 名称:

    ```
    sudo docker stop [DOCKER NAME]
    ```

## 本地挖矿的步骤

1. 为Zilliqa创建一个新目录：

   ```
   cd ~/Desktop && mkdir Zilliqa
   ```

---

2. 为Scilla创建一个新目录：

   ```
   mkdir Scilla
   ```

---

3. 创建一个新的加入目录：

   ```
   mkdir join
   ```

---

4. 克隆Scilla存储库并将目录更改为：

   ```
   git clone https://github.com/Zilliqa/Scilla.git Scilla && cd Scilla && git checkout v0.0.4
   ```

---

5. 找出当前目录路径并将其写下：

   ```
   pwd
   ```

---

6. 按照[这里](https://github.com/Zilliqa/scilla/blob/master/INSTALL.md#ubuntu)的说明下载Ubuntu的Scilla二进制依赖项。然后构建Scilla二进制文件：

   ```
   make clean; make
   ```

---

7. 克隆Zilliqa存储库并将目录更改为：

   ```
   cd .. && git clone https://github.com/Zilliqa/Zilliqa.git Zilliqa && cd Zilliqa && git checkout v3.4.1
   ```

---

8. 再次找出当前目录路径并将其写下：

   ```
   pwd
   ```

---

9. > **备注:** 如果你用 CPU 进行挖矿，请跳过此步骤.

   **(可选)** 为 Nvidia GPUs 安装 CUDA 驱动，参看 [对于 CUDA 部分](#%E5%AF%B9%E4%BA%8Ecuda)

   **(可选)** 为 AMD GPU 安装 OpenCL 驱动， 参看 [对于 OpenCL 部分](#%E5%AF%B9%E4%BA%8Eopencl).

---

10. 首先下载Zilliqa依赖项，然后构建Zilliqa用于CPU挖矿**或**GPU挖矿。

   - 首先，下载依赖项：

     ```
     sudo apt-get update
     sudo apt-get install git libboost-system-dev libboost-filesystem-dev libboost-test-dev \
     libssl-dev libleveldb-dev libjsoncpp-dev libsnappy-dev cmake libmicrohttpd-dev \
     libjsonrpccpp-dev build-essential pkg-config libevent-dev libminiupnpc-dev \
     libprotobuf-dev protobuf-compiler libcurl4-openssl-dev
     ```

   - **（选项1）** 构建Zilliqa用于CPU挖矿

     ```
     ./build.sh
     ```

   - **（选项2）** 使用CUDA为Nvidia GPU挖矿构建Zilliqa

     ```
     ./build.sh cuda
     ```

   - **（选项3）** 使用OpenCL构建用于AMD GPU挖矿的Zilliqa

     ```
     ./build.sh opencl
     ```

---

11. 下载压缩的加入配置文件：

    ```
    cd ../join && wget https://testnet-join.zilliqa.com/configuration.tar.gz
    ```

---

12. 解压缩压缩文件：

    ```
    tar zxvf configuration.tar.gz
    ```

---

13. 编辑*constants.xml*并将`SCILLA_ROOT`参数更改为Scilla源目录的完整路径，如**步骤5**中所示。

---

14. **（可选）** 如果您希望使用GPU，请编辑*constants.xml*并更改以下内容：

    - **对于AMD GPU**：将`FULL_DATASET_MINE`参数从`false`更改为`true`。将`OPENCL_GPU_MINE`参数从`false`更改为`true`。

    - **对于Nvidia GPU**：将`FULL_DATASET_MINE`参数从`false`更改为`true`。 将`CUDA_GPU_MINE`参数从`false`更改为`true`。

      > 注意：如果您希望同时运行多个GPU，则需要按照[此处](#%E5%AF%B9%E4%BA%8E%E5%A4%9Agpu)的说明修改*constants.xml*文件。

---

15. 如果您在NAT环境中分别使用**选项1a**或**选项1b**，则启用UPnP**或**执行单端口转发。

    否则，如果您已经有公开的IP地址，请使用**选项2**查找当前的公共IP地址。

    > 注意：如果您使用的是家用路由器，则很可能是在NAT环境中并且可以启用UPnP。但是，如果UPnP不起作用，则可以执行端口转发。

    - **（选项1a）** 在家用路由器上启用UPnP模式。请谷歌你的家庭路由器设置，在[这里](https://routerguide.net/how-to-enable-upnp-for-rt-ac66u/)可以找到一个例子。您可以通过安装以下工具来检查是否已启用UPnP：

      ```
      sudo apt-get install miniupnpc
      ```

       然后在命令行中输入：

      ```
      upnpc -s
      ```

      您将收到一条消息“网络上找到的UPNP设备列表：**”或**“网络上找不到IGD UPnP设备！”。前者意味着UPnP模式已成功启用，而后者意味着UPnP模式存在问题。如果您属于后一种情况，请参阅下面的**选项1b**。

    - **（选项1b）** 单端口在路由器菜单中转发本地计算机IP。您可以在路由器菜单的TCP / UDP协议中同时将`30303`设置为外部端口（端口范围），`30903`设置为内部端口（本地端口），您可以在[此处](https://www.linksys.com/us/support-article?articleNum=136711)找到示例。然后，您可以使用命令提示符找出您的路由器IP地址：

      ```
      curl https://ipinfo.io/ip
      ```

    - **（选项2）** 如果您的命令提示符中已有公共IP地址，请直接查找您的公共IP地址：

      ```
      curl https://ipinfo.io/ip
      ```

---

16. 使用以下命令加入Zilliqa测试网络：

    ```
    ./launch.sh
    ```

---

17. 系统将提示您输入以下详细信息：

    - `Enter the full path of your zilliqa source code directory: `*[键入您找到的路径第8步]*

    - `Enter your IP address (NAT or *.*.*.*): `*[键入**NAT**或您在步骤14中找到的IP地址]*

    - `Enter your listening port (default: 30303): `*[如果使用默认值，请按**Enter**跳过]*

---

18. 你现在是猫山王测试网络的一名矿工。您可以使用以下方法监控进度：

    ```
    tail -f zilliqa-00001-log.txt
    ```

    如果您在DS纪元开始时设法赢得PoW进程，您将在日志中收到您已成为网络中的分片/ DS节点的通知。

---

19. 要检查本地生成的公钥和私钥对，可以在命令提示符中输入：

    ```
    less mykey.txt
    ```

    第一个十六进制字符串是您的**公钥**，第二个十六进制字符串是您的**私钥**。

    > 注意：密钥对是在磁盘上本地生成的。务必记住将私钥保存在安全的地方！

---

20. 停止本地挖矿，请输入:
    ```
    pkill zilliqa
    ```

## 讨论渠道和错误报告

#### 通道

加入我们的官方挖矿讨论Gitter频道：https://gitter.im/Zilliqa/Mining

加入社区管理的Telegram频道：https://t.me/zilliqaminer

#### 报告

如果您在加入*猫山王*测试网时遇到问题或错误，请将您的log.txt文件提交到此Google表单：https://goo.gl/forms/y21CZrSwotvyNoKY2。

我们会尽可能帮助您。
