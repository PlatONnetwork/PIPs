---
PIP: 7
Topic: PlatON网络对双ChaiID的支持
Author: benbaley
Status: Draft 
Type: Upgrade
Description: 提议在PlatON主网络对ChainID `100`和`210425`同时进行支持和适配
Created: 2022-02-11
---

## 背景

PlatON主网当前的ChainID值是100，在[ethereum-lists/chains](https://github.com/ethereum-lists/chains) 登记的链ID中与其他网络冲突了，为了更好的对DApp（如MetaMask）进行支持，需要再确定一个在EVM框架下由统一的第三方平台注册过唯一的链ID。因PlatON主网已运行近一年，已有的生态工具（钱包、浏览器等）不能立即迁移或适配新的链ID，因此提议在一段时期内PlatON主网络同时支持双ChainID。

## 概述

当前PlatON主网络ChainID值和其他网络冲突，存在以下问题：

1. 存在不同链上的交易在PlatON网络上重放风险
2. 如MetaMask之类的第三方应用不能唯一适配到PlatON网络上

因此，我们的最终目标是将主网络的ChainID由当前的`100`更新为`210425`，为达成这个目标需要分2步进行：

- 支持新ChainID`210425`

可以通过PlatON特有的升级提案先在主网络确保对新链ID`210425`的支持，同时兼容旧ChainID`100`，这样既可以很好的适配诸如MetaMask之类的工具，方便新DApp的快速开发，又可以同时兼容当前已经在运行中的应用。

- 未来择机停止对ChainID`100`的支持

在完成第一阶段升级后，所有新的应用将直接使用新ChainID开发和运行，对于旧的DApp来说（只针对使用了ChainID的应用），可以有充足的时间进行适配和升级，在经历充分的适配调整时间以后，需要再发起新的升级提案，停止使用旧ChainID。

## 原理

在PlatON网络中ChainID主要有以下作用：

1. 防止交易重放

交易签名加入了ChainID，用于区分不同的链，避免将其他链的交易在PlatON网络中执行，因v值除了记录ChainID外还用于标记椭圆曲线y值的奇偶性，因此只要两个ChainID不是连续整数则可以根据v值获取到用户签名时填入的ChainID。

2. 防止异形攻击

P2P握手消息中加入ChainID，用于验证节点是否是属于同一个链，可以通过判断握手消息中携带的ChainID是否为PlatON所支持的2个ChainID中的一个来适配。

## 实现

1. 交易验证

v = v - 2*100（原ChainID）
v = v - 8
if v > 28 
then
  v = v - (新ChainID - 原ChainID)
  判断v是否为27或28
  是
     使用新新ChainID进行判断
  否
     签名非法
else
  使用原ChainID进行判断

2. P2P握手
handle处理ping、pong、findnode、neighbors消息时，直接对Rest字段判断，为两个ChainID其一者则验证通过。

3. opCode(0x46)

EVM执行CHAINID指令时，根据大版本号判断，提案前使用旧ChainID， 提案升级成功后按新ChainID返回。

## 影响分析

底层实现本提案后，将有以下影响：

1. 需要链上治理升级

意味着需要分叉， 新客户端将兼容旧的交易的ChainID和旧节点， 但旧客户端无法支持新ChainID的交易和新节点。

2. P2P链接的影响

由于新节点使用新ChainID，在节点发现时（UDP消息，对应Ping、Pong、Findnode、Neighbors）新节点消息带新ChainID，该消息会被旧节点认为是不属于当前链，新节点可以发现旧节点，旧节点也发现不了新节点，且旧节点可以链上新节点，但新节点只能链新节点。

3. DApp需要对ChainID更新适配

由于新版本客户端ChainID更新为`210425`,EVM获取ChaiID的指令需要根据版本号（链上）来判断返回新值或是旧值，因此对于DApp中合约逻辑使用了ChainID的应用，需要重新进行适配，对于在合约中硬编码了ChainID逻辑的应用，只能通过重新部署新合约后将旧合约中数据迁移致新约的方式进行适配，对于在应用层（链下）使用ChainID的情形，需要在应用层代码逻辑中自行适配。

## 链接

1. [Chain Agnostic Improvement Proposals](https://github.com/ChainAgnostic/CAIPs)
2. Ethereum-lists注册[PlatON主网络ChainID](https://github.com/ethereum-lists/chains/blob/master/_data/chains/eip155-210425.json)
