

>版本信息
>
>ChainID: 201018
>
>Version: 0.14.0
>
>Git Commit: 16aced7d20f7caec781cc0a69c0cfea1a8089ecb
>
>PlatON二进制下载: http://download.alaya.network/alaya/platon/0.14.0/platon
>

### 1、安装指南

​		若已安装了Alaya节点，请跳到**第2点**进行更新操作；

​		未安装节点，请严格按照[官方文档](https://devdocs.alaya.network/alaya-devdocs/zh-CN/)安装，如有需要帮助请联系客服。

### 2、更新指南（先更新，暂不重启）

​		以下步骤以 Ubuntu18.04 系统为例，更新操作分为三种方式：**PPA、源码编译、直接使用二进制**，请严格按照以下步骤操作升级，如有需要帮助请联系客服。

- **PPA**

  ```bash
  # 更新
  $ sudo apt update
  # 卸载当前安装版本
  $ sudo apt remove `apt search platon|awk -F/ '/installed/{print $1}'` --purge -y  
  # 安装新版platon
  $ sudo apt install -y platon0.14.0
  # 查看版本
  $ platon version
  ```

- **源码编译**（针对之前已成功编译过的环境，全新编译请参考[官网](https://devdocs.alaya.network/alaya-devdocs/zh-CN/Install_Alaya/)）

  ```bash
  $ cd PlatON-Go
  $ git fetch --all
  $ git checkout -b alaya-0.14.0 16aced7d20f7caec781cc0a69c0cfea1a8089ecb
  $ make clean
  $ make all
  # 使用PlatON-Go/build/bin/目录下的platon文件替换旧的platon
  $ chown +x platon
  # 查看版本
  $ ./platon version
  ```

- **直接用二进制**

  ```bash
  # 下载最新的二进制
  $ wget http://download.alaya.network/alaya/platon/0.14.0/platon
  # 使用新的platon二进制替换旧的platon二进制
  $ chmod u+x platon
  # 查看版本
  $ ./platon version
  ```

### **3、择机重启进程**

如果当前节点质押过，首先通过[区块链浏览器](https://scan.alaya.network/node)查看节点是否已经被选为验证人（共识中）
如果已经当选，请等待节点退出验证人（共识中转为活跃中）后再进行重启操作。

根据各自的管理方式，重启platon进程

### 4、提案处理

- 质押过的节点，请通过区块链浏览器查看当前节点是否是以下状态：

1. 出块中
2. 共识中
3. 活跃中

如果当前节点为上述3中状态的一种，请参考以下命令对版本0.14.0进行投票，提案id为<0x261cf6c0f518aeddffb2aa5536685af6f13f8ba763c77b42f12ce025ef7170ed>：

```bash
# 升级提案投票命令示例
$ mtool-client vote_versionproposal --proposalid 0x261cf6c0f518aeddffb2aa5536685af6f13f8ba763c77b42f12ce025ef7170ed --keystore <staking.json文件绝对路径> --config <validator_config.json文件绝对路径>
```

如果节点状态为“**候选中**”，则需要做版本声明，请参考以下命令进行版本声明。

```bash
#版本声明示例
$ mtool-client declare_version --keystore <staking.json文件绝对路径> --config <validator_config.json文件绝对路径>
```

- 未质押过的节点，跳过本步骤。

### 6、反馈

platon升级失败的，可通过以下渠道反馈。

2. 验证人微信群
3. 邮箱 validator@platon.network
