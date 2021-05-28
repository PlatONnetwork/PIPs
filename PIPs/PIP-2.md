---
PIP: 2
Topic: 兼容以太坊
Author: clearly
Status: Draft 
Type: Text
Description: 底层全面兼容以太坊的生态
Created: 2021-05-28
---

# PIP-2：关于全面兼容以太坊的提案

## 摘要
为吸引更多开发者参与PlatON网络中来，降低开发者将Dapp应用从以太坊迁移到PlatON上的开发成本，PlatON应该在如SDK、EVM、RPC接口、solidity语言等方面兼容以太坊。

## 动机

以太坊作为世界上第二大市值的区块链网络，也是最大的智能合约平台，有着众多的开发者和成熟的社区，许多开发者是通过以太坊和他们的智能合约进入区块链世界的，他们对使用以太坊智能合约以及与开发Dapp相关的工具、SDK等都很熟练，但对PlatON不熟悉，因此如果能在Dapp开发层面实现对以太坊的兼容，无疑将很大程度上提升PlatON对开发者的吸引力，生态也会逐渐壮大起来。

当前PlatON和以太坊不兼容的地方:

- 地址格式不同，PlatON采用bech32的地址格式，以太坊采用的是EIP55的地址格式
- Token单位不同，PlatON的Token单位为lat/von，以太坊的为ether/wei
- 部分函数在PlatON中没有实现或者实现不同。如block.difficlty、miner.setEtherbase等函数，使用到这些函数的以太坊合约可能不适合在PlatON上运行，需要进行相应调整，不过这些函数很少使用
- 区块头时间戳不同，以太网为秒，PlatON为毫秒

由以上差异可知，PlatON只需要极小的改动就可以实现对以太坊的兼容。

## 实现说明

### RPC接口兼容

#### 1.接口格式兼容
PlatON目前的地址编码格式为bech32，以太坊目前的地址编码格式为EIP55，因此我们要在地址格式上做兼容。
PlatON部分RPC接口的命名空间为'PlatON'，如’PlatON_getBalance’。以太坊部分RPC接口的命名空间为'eth'，如‘eth_getBalance’。两个函数的功能一致，因此我们要在命名空间上兼容。

为此，我们通过版本号来区分不同格式的接口调用，其中1.0为PlatON的RPC接口，2.0为以太坊的RPC接口。

示例:
```
// Request 
curl -X POST --data '{"jsonrpc":"1.0","method":"platon_getBalance","params":["lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqp7pn3ep", "latest"],"id":1}'

// Result
{
  "id":1,
  "jsonrpc": "1.0",
  "result": "0x0234c8a3397aab58" // 158972490234375000
}

// Result
curl -X POST --data '{"jsonrpc":"1.0", "method": "platon_getStorageAt", "params": ["0x295a70b2de5e3953354a6a8344e616ed314d7251", "0x0", "latest"], "id": 1}' localhost:8545

// Result
{"jsonrpc":"1.0","id":1,"result":"lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqp7pn3ep"}

```

兼容以太坊接口的rpc版本,与以太坊现有rpc版本保持一致，均为2.0,示例如下:
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1", "latest"],"id":1}'

// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x0234c8a3397aab58" // 158972490234375000
}


curl -X POST --data '{"jsonrpc":"2.0", "method": "eth_getStorageAt", "params": ["0x295a70b2de5e3953354a6a8344e616ed314d7251", "0x0", "latest"], "id": 1}' localhost:8545

{"jsonrpc":"2.0","id":1,"result":"0x00000000000000000000000000000000000000000000000000000000000004d2"}

```

#### 2.需要新增或修改的接口
以太坊中的一些RPC接口PlatON中没有或实现不同，我们需要对此进行适配。

1. ChainId获取接口
以太坊在EIP-695中增加该接口的支持，在PlatON中同样需要添加该接口的支持

2. miner.setEtherbase
因为PlatON中区块提议人都已明确制定固定的收益地址，因此miner.setEtherbase接口无需在PlatON中实现。

### solidity合约兼容

#### 1.底层指令兼容
经排查，EVM底层指令PlatON和以太坊已经基本兼容,不兼容的指令有:
- TIMESTAMP
  PlatON返回的是单位是ms，以太坊返回的单位是s.该指令的返回单位需要统一改为s。

#### 2.编译器兼容

编译器兼容以太坊主要体现在solidity合约中的地址格式，当前PlatON的合约编译器只支持bech32个，为实现对以太坊合约的兼容，只需在每个solidity大版本的最新版(0.4.26，0.5.17，0.6.12，0.7.6和0.8.4）实现对EIP55地址格式的兼容即可。

对于Token单位来说，为提示用户或开发者当前是在PlatON网络中使用合约，因此不建议对以太坊Token单位 `ether/wei` 实现兼容。

## 说明
本次升级将兼容历史数据，需链上治理升级。详见讨论[链接](https://forum.latticex.foundation/t/topic/4636)

