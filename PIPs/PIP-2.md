---
PIP: 2
Topic: 兼容以太坊 (Compatible with Ethereum)
Author: clearly
Status: Draft 
Type: Text
Description: 底层全面兼容以太坊的生态 (The underlying technology shall be fully compatible with Ethereum's ecosystem)
Created: 2021-05-28
---

# PIP-2：关于全面兼容以太坊的提案

## 摘要
为吸引更多开发者参与 PlatON 网络中来，降低开发者将 Dapp 应用从以太坊迁移到 PlatON 上的开发成本，PlatON 应该在如 SDK、EVM、RPC 接口、solidity 语言等方面兼容以太坊。

## 动机

以太坊作为世界上第二大市值的区块链网络，也是最大的智能合约平台，有着众多的开发者和成熟的社区，许多开发者是通过以太坊和他们的智能合约进入区块链世界的，他们对使用以太坊智能合约以及与开发 Dapp 相关的工具、SDK 等都很熟练，但对 PlatON 不熟悉，因此如果能在 Dapp 开发层面实现对以太坊的兼容，无疑将很大程度上提升 PlatON 对开发者的吸引力，生态也会逐渐壮大起来。

当前 PlatON 和以太坊不兼容的地方:

- 地址格式不同，PlatON 采用 bech32 的地址格式，以太坊采用的是 EIP55 的地址格式
- Token 单位不同，PlatON 的 Token 单位为 lat/von，以太坊的为 ether/wei
- 部分函数在 PlatON 中没有实现或者实现不同。如 block.difficulty、miner.setEtherbase 等函数，使用到这些函数的以太坊合约可能不适合在 PlatON 上运行，需要进行相应调整，不过这些函数很少使用
- 区块头时间戳不同，以太网为秒，PlatON 为毫秒

由以上差异可知，PlatON 只需要极小的改动就可以实现对以太坊的兼容。

## 实现说明

### RPC 接口兼容

#### 1. 接口格式兼容
扩展 JSON-RPC 2.0，对 request 请求对象做出以下修改:  
  - 增加 bech32 字段，Booleans 类型。bech32 为 true 表示此次 rpc 调用中地址部分的编解码格式为 bech32，默认为 EIP55。
  - 优化 method 字段，在命名空间中增加对 eth 的支持，eth 等同于 platon。

接收 request 的流程做如下优化：
1. 优先通过 method 中是否包含 eth/platon 来区分是以太坊 / PlatON 调用
    ```
    // PlatON 调用
    curl -X POST --data '{"jsonrpc":"2.0","method":"platon_getBalance","params":["lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqp7pn3ep", "latest"],"id":1}'
    // 以太坊调用
    curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1", "latest"],"id":1}'
    ```
2. 如果 method 相同，通过 request 中 bech32 的值来区分是以太坊 / PlatON 调用
    ```
    // PlatON 调用
    curl -X POST --data '{"jsonrpc":"2.0","bech32":true, "method": "txpool_contents", "params": [], "id": 1}'
    // 以太坊调用
    curl -X POST --data '{"jsonrpc":"2.0", "method": "txpool_contents", "params": [], "id": 1}'
    ```

对于来自以太坊的调用的响应对象，地址的编解码格式为 EIP55。对于来自 PlatON 的调用的响应对象，地址的编解码格式为 bech32。示例:
```
 // PlatON 调用
 {"jsonrpc":"2.0","id":1,"result":["lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqp7pn3ep"]}
 // 以太坊调用
 {"jsonrpc":"2.0","id":1,"result":["0x407d73d8a49eeb85d32cf465507dd71d507100c1"]}
```

#### 2. 需要新增或修改的接口
以太坊中的一些 RPC 接口 PlatON 中没有或实现不同，我们需要对此进行适配。

1. miner.setEtherbase
因为 PlatON 中区块提议人都已明确制定固定的收益地址，因此 miner.setEtherbase 接口无需在 PlatON 中实现。

### solidity 合约兼容

#### 1.底层指令兼容

EVM底层指令Alaya和以太坊已经基本兼容,不兼容的指令有:
- TIMESTAMP
  Alaya返回的是单位是ms，以太坊返回的单位是s，经开发者充分[讨论](https://github.com/PlatONnetwork/PlatON-Go/issues/1816)，由于出块间隔不同，该指令应该与以太坊有所区分，暂不修改。
- DIFFICULTY
  由于共识机制不同，该指令在Alaya没用，但虚机底层以及编译器都做了支持，无需修改。

#### 2. 编译器兼容

编译器兼容以太坊主要体现在 solidity 合约中的地址格式，当前 PlatON 的合约编译器只支持 bech32 个，为实现对以太坊合约的兼容，只需在每个 solidity 大版本的最新版 (0.4.26，0.5.17，0.6.12，0.7.6 和 0.8.4）实现对 EIP55 地址格式的兼容即可。

对于 Token 单位来说，为提示用户或开发者当前是在 PlatON 网络中使用合约，因此不建议对以太坊 Token 单位 `ether/wei` 实现兼容。

## 说明
本次升级将兼容历史数据，需链上治理升级。详见讨论 [链接](https://forum.latticex.foundation/t/topic/4636)


[English translation]
# PIP-2：The PIP of fully Compatible with Ethereum

## Abstract
In order to attract more and more developers to the PlatON network and ecosystem,  and reduce the development cost of moving dApps from Ethereum to PlatON, we should be Ethereum compatible in aspects such as SDK, EVM, RPC interface, solidity language, etc.

## Motivation

As a chunk of the world's second largest market value blockchain network, Ethereum is also the biggest platform for the smart contracts, with many developers and mature community. Many developers evolving to blockchain practitioners are through the Ethereum smart contracts, who are skilled with the smart contracts, the tools for the developement of DApps, SDKs of Ethereum, but not familiar with PlatON. So if we can make it compatible with Ethereum, it will definitely make PlatON more attractive to developers, and it is conducive to the development of ecosystems.

The key points PlatON and Ethereum are currently incompatible:


- The address format is different. PlatON uses the address format of Bech32, while Ethereum uses the address format of Eip55
- The Token units are different. PlatON's Token units are LAT/VON and Ethereum's are Ether/WEI
- Some functions are not implemented in PlatON or are implemented differently. Functions such as block.difficulty, miner. SetEtherbase, etc., the Ethereum contracts using these functions may not work on PlatON and may need to be adjusted accordingly, but these functions are rarely used.
- The block header timestamp is different, Ethereum in seconds and PlatON in milliseconds

From these differences, PlatON can be made compatible with Ethereum with minimal changes.

## Implementation notes

### Compatible with RPC interface

#### 1.Compatible interface format
PlatON's current address encoding format is Bech32, while Ethereum's current address encoding format is Eip55, so we need to make compatibility in address format.
Parts of the PlatON RPC interface have a namespace of 'PlatON', such as 'platon_getBalance'. Some of the RPC interfaces on Ethereum have a namespace of 'eth', such as' eth_getBalance '. Both functions have the same functionality, so we need namespace-compatibility.

To do this, we distinguish interface calls in different formats by version numbers, where 1.0 is the RPC interface for PlatON and 2.0 is the RPC interface for Ethereum.

Example:
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

The version of RPC compatible with the Ethereum interface, which is the same as the existing Ethereum RPC version, is 2.0, as shown below:
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

#### 2.Interfaces that need to be added or modified
Some RPC interfaces in Ethereum are not or are implemented differently in PlatON, and we need to adapt them.

1. miner.setEtherbase
The miner. SetEtherBase interface does not need to be implemented in PlatON because the block proposer in PlatON has a fixed income address explicitly specified.

### Solidity Contract Compatibility

#### 1.Compatibility of underlying instructions
After checking, PlatON and Ethereum are basically compatible with EVM underlying instructions. Incompatible instructions include:
- TIMESTAMP
  PlatON returns the unit of ms, Ethereum returns the unit of s. The return unit of this instruction needs to be changed to s.

#### 2.Compiler compatibility

Compiler compatibility with Ethereum is mainly embodied in the address format of solidity contracts. The current PlatON contract compiler only supports Bech32 contracts. To achieve compatibility with Ethereum contracts, simply using the latest version of each large solidity release (0.4.26, 0.5.17, 0.6.12, 0.7.6 and 0.8.4) can be compatible with the EIP55 address format.

For Token units, compatibility with Ethereum Token units `ether/wei` is not recommended as a reminder that the user or developer is currently using the contract in the PlatON network.

## Description
This upgrade will be compatible with historical data and requires an on-chain governance upgrade. See the discussion [Link](https://forum.latticex.foundation/t/topic/4636)
