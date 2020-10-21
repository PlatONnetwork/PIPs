[toc]

>版本信息
>
>ChainID：201018
>
>Version: 0.13.2
>
>Git Commit: da6a7b338fa4b0ae2f11a070610d5bee48b30b92
>
>[PlatON二进制下载](http://download.alaya.network/alaya/platon/0.13.2/platon-ubuntu-amd64-Alaya_0.13.2.tar.gz )
>
>Mtool及Aton相关下载参考[官网开发者文档](https://www.platon.network/developer/?lang=zh#aton)

### 0、节点关键数据备份

​		**节点相关keys、质押及收益地址等重要信息，请妥善保管和备份；若未备份，一旦丢失，无法找回！**

### 1、安装指南

​		节点的全新安装，请参考开发者文档：

- **PPA**

  [官方PPA安装文档](https://devdocs.alaya.network/alaya-devdocs/zh-CN/Install_Node/)

- **源码编译**

  [官方源码编译文档](https://devdocs.alaya.network/alaya-devdocs/zh-CN/Install_Alaya/)

### 2、原拉力赛节点切换Alaya网络指南

**注意1**：原拉力赛节点如果已有质押，请先解除质押，否则无法申请Alaya网质押；

**注意2**：请一定按照安装文档完成**NTP**配置，否则可能因时钟不同步导致无法正常同步区块；

**注意3**：原启动脚本中的“--testnet”参数调整为“--alaya”；

**注意4**:   原拉力赛相关钱包请妥善保管，作为后续拉力赛奖励兑换的依据。

​		此次切换操作需要清理节点数据，**以节点base目录在/opt/platon-node/为例**，首先停止platon进程，然后清理数据操作如下：

```bash
# 解除质押命令：
$ mtool-client unstaking --keystore $MTOOLDIR/keystore/staking.json --config $MTOOLDIR/validator/validator_config.json
# NTP检查命令
$ ntpq -4c rv | grep leap_none
# 查看要清理的文件和目录
$ ls /opt/platon-node/data/platon
chaindata  evidence  LOCK  nodes  snapshotdb  transactions.rlp  wal
# 停止platon进程,参考命令
$ ps -ef | grep platon | grep datadir | grep -v grep | awk '{print $2}' | xargs kill
# 进入相应的目录
$ cd /opt/platon-node/data
# 删除platon目录，节点相关keys可以利旧
$ rm -rf platon 
```

​		以下步骤以 Ubuntu18.04 系统为例，更新操作分为三种方式：**PPA、源码编译、直接使用二进制**，请严格按照以下步骤操作升级，如有需要帮助请联系客服。

- **PPA**   注意：Alaya网对应PPA项目为alaya

  ```bash
  # 卸载当前安装版本
  $ sudo apt remove `apt search platon|awk -F/ '/installed/{print $1}'` --purge -y  
  # 添加alaya项目并更新
  $ sudo add-apt-repository ppa:ppatwo/alaya  && sudo apt-get update 
  # 安装platon0.13.2
  $ sudo apt install -y platon0.13.2
  # 对照commitid检查版本是否正确
  $ platon version
  # 根据各自的管理方式，启动platon进程，注意修改启动参数
  $ nohup ......   & (再次提醒：修改--testnet参数为--alaya！！！)
  # 查看块高是否正常
  $ platon attach http://127.0.0.1:6789 -exec platon.blockNumber
  ```
  
- **源码编译**（本指南针对之前已成功编译过的环境，全新编译请参考官网[源码安装Alaya](https://devdocs.alaya.network/alaya-devdocs/zh-CN/Install_Alaya/)）

  ```bash
  $ mkdir PlatON-Go-Alaya && cd PlatON-Go-Alaya
  $ git clone -b alaya-develop https://github.com/PlatONnetwork/PlatON-Go.git --recursive
  $ cd PlatON-Go && make all
  # 此处可能报错，如果报错，请确认go版本，要求1.13+
  # 使用PlatON-Go-Alaya/build/bin/目录下的platon文件替换旧的platon
  $ chown +x platon
  # 请根据实际部署环境，使用新的platon二进制替换旧的platon二进制!!!
  # 对照commitid检查版本是否正确
  $ ./platon version
  # 根据各自的管理方式，启动platon进程，注意修改启动参数
  $ nohup ......   & (再次提醒：修改--testnet参数为--alaya！！！)
  # 查看块高是否正常
  $ ./platon attach http://127.0.0.1:6789 -exec platon.blockNumber
  ```
  
- **直接用二进制**

  ```bash
  # 下载最新的二进制
  $ wget http://download.alaya.network/alaya/platon/0.13.2/platon-ubuntu-amd64-Alaya_0.13.2.tar.gz
  # 解压,使用新的platon二进制替换旧的platon二进制
  $ tar -xvf platon-ubuntu-amd64-Alaya_0.13.2.tar.gz
  $ cd platon-ubuntu-amd64-Alaya_0.13.2 && chmod +x platon
  # 请根据实际部署环境，使用新的platon二进制替换旧的platon二进制!!!
  # 对照commitid检查版本是否正确
  $ ./platon version
  # 根据各自的管理方式，启动platon进程，注意修改启动参数
  $ nohup ......   & (再次提醒：修改--testnet参数为--alaya！！！)
  # 查看块高是否正常
  $ ./platon attach http://127.0.0.1:6789 -exec platon.blockNumber
  ```

### 3、mtool升级、钱包及验证人配置文件重建

```bash
# 备份老版mtool目录，删除原mtool-client.zip，下载并解压新版mtool
$ mv mtool-client mtool-client.bak && rm -f mtool-client.zip
$ wget http://download.alaya.network/alaya/mtool/linux/0.13.2/mtool-client.zip
$ unzip mtool-client.zip
# 重新创建质押和收益钱包
$ mtool-client account new staking
$ mtool-client account new reward
# 重新下载nginx脚本并执行，原拉力赛节点已部署nginx也需要执行，否则会导致后续validator_config.json生成的certificate路径错误
$ cd $MTOOLDIR && wget http://download.alaya.network/opensource/scripts/nginx_conf.sh
$ chmod +x nginx_conf.sh && ./nginx_conf.sh
# 下载validator_conf.sh并重新生成validator_config.json
$ cd $MTOOLDIR && wget http://download.alaya.network/opensource/scripts/validator_conf.sh 
$ chmod +x validator_conf.sh && ./validator_conf.sh
# 查询质押钱包余额进行验证，请注意：下面是一条命令！！
$ mtool-client account balance staking.json --config $MTOOLDIR/validator/validator_config.json
```

### 4、反馈

如何成为验证人或其他问题请参考：节点招募计划，也可通过以下渠道反馈：

1. 发送至 [gitter room](https://gitter.im/PlatON_Network/Welcome)
2. 验证人微信群
3. 邮箱 validator@platon.network
