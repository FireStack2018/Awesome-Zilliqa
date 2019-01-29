# 挖矿指南
Zilliqa官方挖矿指南（中文版）首发，由FireStack团队翻译，同时向Hash1024社区表示感谢。

- [**一般信息**](#一般信息)
- [**推荐硬件配置**](#推荐硬件配置)
- **Zilliqa挖矿步骤**
   - **步骤 1:** [基础要求](#基础要求)
   - **步骤 2** (选项 1): [docker镜像配置本地挖矿](#本地搭建docker，仅适用于CPU或Nvidia的GPU)
   - **步骤 2** (选项 2): [docker镜像配置远程挖矿](#远程docker挖矿)
   - **步骤 2** (选项 3): [配置本地挖矿](#配置本地挖矿)
- [**交流渠道**](#交流渠道)


## 一般信息

### Epoch架构

![Zilliqa Epoch Architecture](https://i.imgur.com/Da4t6FW.png)
在每个DS Epoch开始时，所有候选人都将运行工作证明（Ethash算法）过程`150`秒窗口，以便竞争加入Zilliqa网络。

- 满足`DS_POW_DIFFICULTY`参数的节点将能够作为DS节点加入。
- 满足`POW_DIFFICULTY`参数的节点将作为分片节点加入。

每个DS Epoch（约2-3小时）内总共有`100`个TX时期（每个1-2分钟）。第100个TX时期被称为 **Vacuous epoch**。

Vacuous epoch会独立处理：

- 所有节点的coinbase奖励机制
- 升级机制（因为pBFT中没有分支）
- 持久状态存储（写入节点的DB而不是仅存储在内存中）。

在这个时期，网络不会处理任何交易。

### “工作证明”算法

Zilliqa的“工作证明”算法采用了[**Ethash**](https://github.com/ethereum/wiki/wiki/Ethash). 所以Zilliqa采用了 **DAG（有向无环图）**作为工作量证明算法，该算法以每个 **DS epoch**的增量率生成。自举DAG大小约为`1.02GB`。

### 网络难度

主网的自助最低难度等级为3。此难度级别是动态的，并根据每个DS epoch的提交目标值`1810`以`+/- 100`的幅度进行调整。

> 注意：难度级别是log2（难度）。

假设网络中有`1810`个席位但是有`1910`个PoW提交，则下一个DS epoch的难度等级将增加1。

假设网络中有`1810`个席位但是有`1710`个PoW提交，则下一个DS epoch的难度等级将减少1。

### 奖励机制

在Zilliqa网络中，奖励被分类为：

* 基础奖励 **[总数的25％]**，用于网络中的所有验证节点（DS / shard）
* 灵活奖励 **[总数的70％]**，基于节点在执行pBFT共识期间提交的有效和已接受签名（前2/3）的数量

基本奖励和灵活奖励对于所有DS和分片节点具有相同的权重。所有奖励都在整个DS epoch得到统一，并且仅在vacuous epoch分发。

例如，如果Zilliqa网络中总共有`2,400`个节点，并且每个DS Epoch的`COINBASE_REWARD`设置为`263698.630137`ZILs，则奖励分配将是：

 - 基本奖励：

```shell
263698.630137 * 0.25 / 2400
每个DS Epoch每个节点ZILs = 27.4686073059375
```
    
 - 灵活奖励:(以先到先得的方式）

```shell
263698.630137 * 0.70 /(2,400 * 2/3 [成功签名者] * 99 [TX区块]）
每个有效和接受的签名ZILs = 1.165334855403409 
```

## 推荐硬件配置

目前，Zilliqa客户端仅支持Ubuntu OS。

如果您希望双启动Windows和Ubuntu 16.04，请按照[此处](https://itsfoss.com/install-ubuntu-1404-dual-boot-mode-windows-8-81-uefi/)的步骤操作。

如果您希望使用在Windows上运行的挖矿平台，请使用docker镜像操作指南[**点击这里**](＃远程docker挖矿)进行远程挖矿。

PoW支持 **AMD**（使用OpenCL）和 **Nvidia**（使用OpenCL或CUDA）GPU。

Zilliqa挖矿节点的 **最低**配置是：

 - x64 Linux操作系统（例如Ubuntu 16.04.05）
 - 英特尔i5处理器或更高版本
 - 4GB DRR3 RAM或更高
 - NAT环境 **或**公共IP地址
 - 任何具有至少2 GB vRAM的GPU

## 基础要求

### 网络配置

> **注意：**如果您使用的是家用路由器，则很可能是在NAT环境中。

如果您在NAT环境中，您可以：

 - 使用 **选项1a**进行单端口转发。这应该是你的 **默认选项**。
 - 如果您的路由器支持UPnP，则使用 **选项1b**启用UPnP模式。

如果您有公共IP地址，则可以完全跳过此网络设置。

 - **（选项1a）**端口转发到端口“33333”，用于外部端口（端口范围）和内部端口（本地端口）。在端口转发时，您还必须在路由器菜单中选择 **BOTH** TCP和UDP协议选项。[**这里**](https://www.linksys.com/us/support-article?articleNum=136711)可以找到此过程的一个示例。在端口转发后，您可以使用此[**开放端口检查工具**](https://www.yougetsignal.com/tools/open-ports/)检查您是否已成功转发端口。

 - **（选项1b）**在家用路由器上启用UPnP模式。请谷歌如何访问您的家庭路由器设置以启用UPnP，[**这里**](https://routerguide.net/how-to-enable-upnp-for-rt-ac66u/)可以找到一个示例。您可以通过安装以下工具来检查是否已启用UPnP：

```shell
sudo apt-get install miniupnpc
```

   然后在命令行中键入以下内容：

```shell
upnpc -s
```

   您应该收到一条消息，显示：

- "List of UPNP devices found on the network : ..."

**或**
 
- "No IGD UPnP Device found on the network !".

第一个消息表示UPnP模式已成功启用，而后者表示启用UPnP模式失败。如果您收到后一条消息，请继续使用 **选项1a**。

### OpenCL驱动安装（AMD/Nvidia）

如果您希望使用支持OpenCL的GPU进行PoW，请运行以下代码来安装OpenCL开发人员包:

```shell
sudo apt install ocl-icd-opencl-dev
```

您可能需要重新启动PC才能使安装生效。重新启动后，使用以下命令检查驱动程序是否已正确安装：

```shell
clinfo
```

如果您遇到缺少OpenCL驱动程序等问题，请查阅此论坛指南[**点击这里**](https://forum.zilliqa.com/t/guide-to-setting-up-6-amd-gpus-on-ubuntu-16-04/180)。 （感谢@Speccles96）

### CUDA驱动安装（Nvidia）

如果您希望使用支持CUDA的GPU进行PoW，请从[NVIDIA官网](https://developer.nvidia.com/cuda-downloads)下载与安装CUDA软件包。您可能需要重新启动PC才能使安装生效。

### 多GPU配置

如果您希望同时运行多个AMD或Nvidia GPU。请编辑位于“join”文件夹中的 *constants.xml*文件中的`GPU_TO_USE`参数。

索引从`0`开始，您可以输入一个或多个GPU，以`,`分隔。

例如：

- 1个GPU为`0`
- 3个GPU为`0,1,2`或`0,2,4`。

请注意，最大的索引必须与您在采矿设备中物理上的GPU数量相对应。



## 本地搭建docker，仅适用于CPU或Nvidia的GPU

1. 按照[**这里**](http://releases.ubuntu.com/xenial/)的说明来安装Ubuntu 16.04.5操作系统。

2. 按照[**这里**](https://docs.docker.com/install/linux/docker-ce/ubuntu/)的说明为Ubuntu安装Docker CE。

3. 在桌面中创建一个新目录并将目录更改为：

   ```shell
   cd ~/Desktop && mkdir join && cd join
   ```

4. 在命令提示符下获取docker镜像：

      ```shell
      wget https://mainnet-join.zilliqa.com/configuration.tar.gz
      tar zxvf configuration.tar.gz
      ```

5. 在命令提示符中找出您当前的IP地址并记录下来。

	> **注意：**如果您使用 **选项1b**，如上面[**网络配置**](#网络配置)中所述，您可以跳过此步骤。

      ```shell
      curl https://ipinfo.io/ip
      ```

6. 在命令提示符下运行shell脚本以启动docker镜像。

	-  **（选项1）**对于使用CPU的挖矿，启动您的docker容器：

		```shell
		./launch_docker.sh
		```
	-  **（选项2）**对于使用Nvidia GPU进行挖矿，请先安装`nvidia-docker` [**点击这里**](https://github.com/NVIDIA/nvidia-docker#ubuntu-140416041804-Debian的jessiestretch)。然后，启动您的docker容器：

		```shell
		./launch_docker.sh --cuda
		```

	> **注意：**如果您希望同时运行多个Nvidia GPU，则需要按照上面的[**说明**](＃多GPU配置)修改 _** constants.xml**_ 文件。

7. 系统将提示您输入一些信息，如下所示：
	- `Assign a name to your container (default: zilliqa):`
		[如果使用默认值，按 **输入**跳过]

   - `Enter your IP address ('NAT' or *.*.*.*):` 
		[如果您使用选项1b，请输入您在步骤6中找到的IP地址 **或** NAT`]

   - `Enter your listening port (default: 33333):`
		[如果使用默认值，按 **输入**跳过]

8. 您现在是Zilliqa主网的矿工。 您可以使用以下方式监控进度：

      ```shell
      tail -f zilliqa-00001-log.txt
      ```

9. 要检查本地生成的公钥和私钥对，可以在命令提示符中输入以下内容：

      ```shell
      less mykey.txt
      ```
      
	第一个十六进制字符串是 **公钥**，第二个十六进制字符串是 **私钥**。

	> **注意：**此密钥对是在磁盘上本地生成的。请记住将私钥保存在安全的地方！	   

10. 要停止挖矿客户端，请停止正在运行的docker容器：

      ```
      sudo docker stop zilliqa
      ```

## 远程docker挖矿

配置内容架构如下图所示。 双方的所有通信都是通过JSON-RPC协议实现的。
![1-to-many](https://i.imgur.com/qReRpRx.jpg)

- CPU节点实例将运行 **Zilliqa客户端**并执行pBFT共识流程以获得奖励。
- GPU集群中的GPU装备将在单独的GPU集群上运行 **Zilminer**，以进行PoW挖矿并直接向CPU节点提供PoW解决方案。

要将GPU集群中的多个GPU装备连接到单个CPU节点，您需要执行以下步骤：

***

1. 按照[**说明**](http://releases.ubuntu.com/xenial/)安装Ubuntu 16.04 OS来创建单个本地/远程CPU节点实例。

2. 按照[**说明**](https://docs.docker.com/install/linux/docker-ce/ubuntu/)在CPU节点实例上为Ubuntu安装Docker CE。

3. 在桌面中创建一个新目录并将目录更改为：

      ```shell
      cd ~/Desktop && mkdir join && cd join
      ```

4. 获取加入配置文件：

      ```shell
      wget https://mainnet-join.zilliqa.com/configuration.tar.gz
      tar zxvf configuration.tar.gz
      ```

5. 在命令提示符中找出您当前的IP地址并记录下来：

      ```shell
      curl https://ipinfo.io/ip
      ```

6. 在配置文件夹中编辑 _constant.xml_ 文件：

	* 将`GETWORK_SERVER_MINE`设置为`true`。
	* 将`GETWORK_SERVER_PORT`设置为您将用于GetWork的端口。（默认为`4202`）
	* 将其他挖矿参数设置为`false`：

        ```shell
        <CUDA_GPU_MINE>false</CUDA_GPU_MINE>
        <FULL_DATASET_MINE>false</FULL_DATASET_MINE>
        <OPENCL_GPU_MINE>false</OPENCL_GPU_MINE>
        <REMOTE_MINE>false</REMOTE_MINE>
        ```

7. 在命令提示符下运行shell脚本以启动docker镜像。

      ```shell
      ./launch_docker.sh
      ```

8. 系统将提示您输入一些信息，如下所示：
	- `为容器指定一个名称（默认：zilliqa）：`
	[如果使用默认值，按 **输入**跳过]

	- `输入您的IP地址（'NAT'或*.*.*.*）：`
	[输入您在步骤5中找到的IP地址 **或**如果您在网络配置中选择了选项1b，则输入`NAT`]

	- `输入你的监听端口（默认值：33333）：`
	[如果使用默认值，按 **输入**跳过]
	
9. 一旦CPU Zilliqa节点运行，您就可以在单独的GPU平台上安装 **Zilminer**：

	- **对于Windows操作系统（使用CUDA 10.0+）：** [**点击下载**](https://github.com/DurianStallSingapore/ZILMiner/releases/download/v0.1.16/zilminer-0.1.16-cuda10.0-windows-64.zip)
	- **对于Ubuntu操作系统：** [**点击下载**](https://github.com/DurianStallSingapore/ZILMiner/releases/download/v0.1.16/zilminer-0.1.16-linux-x86_64.tar.gz)

10. 使用以下命令在单独的GPU装备上设置 **Zilminer**：

    ```shell
    zilminer --max-submit=1 --farm-recheck 10000 --work-timeout=7200 --farm-retries=10 --retry-delay=10 -P zil://wallet_address.worker_name@zil_node_ip:get_work_port
    ```
    > **注意：**您必须相应地更改 *wallet_address*，*worker_name*，*zil_node_ip*和*get_work_port*。

	- 对于`wallet_address`：如果你是单独挖矿，你可以输入任意Zilliqa地址。
	- 对于`worker_name`：您可以输入任何您想要的任意工作者名称。
	- 对于`zil_node_ip`：请输入您在步骤5中记下的Zilliqa节点的IP地址。
	- 对于`get_work_port`：请输入`GETWORK_SERVER_PORT`中使用的端口。默认为`4202`。

11. 您现在是Zilliqa主网的代理矿工。您可以使用以下方法监视CPU节点上的进度：

      ```shell
      tail -f zilliqa-00001-log.txt
      ```

12. 要检查本地生成的公钥和私钥对，可以在CPU节点的命令提示符中输入以下内容：

      ```shell
      less mykey.txt
      ```
	第一个十六进制字符串是 **公钥**，第二个十六进制字符串是 **私钥**。

	> **注意：**此密钥对是在磁盘上本地生成的。请记住将私钥保存在安全的地方！

13. 要停止挖矿客户端，请在CPU节点上停止正在运行的docker容器，并在GPU平台上终止 **Zilminer**进程：

      ```
      sudo docker stop zilliqa
      ```

## 配置本地挖矿

1. 为Zilliqa客户创建一个新目录：

      ```shell
      cd ~/Desktop && mkdir Zilliqa
      ```

2. 为Scilla二进制创建一个新目录：

      ```shell
      mkdir Scilla
      ```

3. 为join文件夹创建一个新目录：

      ```shell
      mkdir join
      ```

4. 克隆Scilla存储库并将目录更改为：

      ```shell
      git clone https://github.com/Zilliqa/Scilla.git Scilla && cd Scilla && git checkout v0.1.1
      ```

5. 找出您的Scilla目录路径并记录下来：

      ```shell
      pwd
      ```

6. 首先，按照[**说明**](https://github.com/Zilliqa/scilla/blob/master/INSTALL.md#ubuntu)下载Scilla二进制格式的Ubuntu依赖库。然后，构建Scilla二进制文件：

      ```shell
      make clean; make
      ```

7. 克隆Zilliqa存储库并将目录更改为：

      ```
      cd ~/Desktop && git clone https://github.com/Zilliqa/Zilliqa.git Zilliqa && cd Zilliqa && git checkout v4.0.0
      ```

8. 再次找出您的Zilliqa目录路径并写下来：

      ```
      pwd
      ```

9. 首先，下载Zilliqa客户端的依赖库。然后使用 **选项1**构建Zilliqa用于CPU挖矿，或使用 **选项2**/ **选项3**进行GPU挖矿。

    - 下载依赖库:

        ```shell
        sudo apt-get update
        sudo apt-get install git libboost-system-dev libboost-filesystem-dev libboost-test-dev \
        libssl-dev libleveldb-dev libjsoncpp-dev libsnappy-dev cmake libmicrohttpd-dev \
        libjsonrpccpp-dev build-essential pkg-config libevent-dev libminiupnpc-dev \
        libprotobuf-dev protobuf-compiler libcurl4-openssl-dev libboost-program-options-dev \
        libssl-dev
        ```

    - **(选项 1)** 构建Zilliqa用于CPU挖矿：

       ```shell
       ./build.sh
       ```
    - **(选项 2)** 使用CUDA支持为Nvidia GPU挖矿构建Zilliqa：

       ```shell
       ./build.sh cuda
       ```
    - **(选项 3)** 使用OpenCL支持为AMD或Nvidia GPU挖矿构建Zilliqa：

       ```shell
       ./build.sh opencl
       ```

10. 下载并解压缩压缩的连接配置文件：

    ```shell
    cd ../join && wget https://mainnet-join.zilliqa.com/configuration.tar.gz && tar zxvf configuration.tar.gz
    ```
    
11. 编辑 _join_ 文件夹中的 _constants.xml_ 以键入Scilla目录中的`SCILLA_ROOT`参数。 例子如下所示：

    ```shell
    <SCILLA_ROOT>/home/ubuntu/Scilla</SCILLA_ROOT>
    ```

12. **（可选）**如果您希望使用GPU，请继续编辑 _join_ 文件夹中 _constants.xml_ 文件中的以下参数：

	- **对于AMD GPU：**将`FULL_DATASET_MINE`参数从`false`更改为`true`。将`OPENCL_GPU_MINE`参数从`false`更改为`true`。
	- **对于Nvidia GPU：**将`FULL_DATASET_MINE`参数从`false`更改为`true`。将`CUDA_GPU_MINE`参数从`false`更改为`true`。

	> **注意：**如果您希望同时运行多个GPU，则需要按照上面的[**说明**](＃多GPU配置)修改 _**constants.xml**_ 文件。

13. 在命令提示符中找出您当前的IP地址并记录下来。

    > **注意：**如果您使用 **选项1b**，如上面[**网络配置**](#网络配置)中所述，您可以跳过此步骤。

    ```shell
    curl https://ipinfo.io/ip
    ```

14. 启动Zilliqa客户端:

    ```shell
    ./launch.sh
    ```

15. 系统将提示您输入以下详细信息：
	- `输入zilliqa源代码目录的完整路径：`
	[键入你在第8步找到的路径]
	- `输入您的IP地址（NAT或*.*.*.*）：`
	[输入您在步骤13中找到的IP地址 **或**如果您使用选项1b，则输入`NAT`]
	- `输入你的监听端口（默认值：33333）：`
	[如果使用默认值，按 **输入**跳过]

16. 您现在是Zilliqa主网的矿工。您可以使用以下方式监控进度：
    ```shell
    tail -f zilliqa-00001-log.txt
    ```

17. 要检查本地生成的公钥和私钥对，可以在命令提示符中输入：

    ```shell
    less mykey.txt
    ```
    第一个十六进制字符串是 **公钥**，第二个十六进制字符串是 **私钥**。

	> **注意：**密钥对是在磁盘上本地生成的。 请记住将私钥保存在安全的地方！

18. 要停止Zilliqa客户端：

    ```shell
    pkill zilliqa
    ```

## 交流渠道

### 渠道

加入我们的官方挖矿论坛：https://forum.zilliqa.com/c/Mining

加入社区管理的Telegram频道：https://t.me/zilliqaminer
