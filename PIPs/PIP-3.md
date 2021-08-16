---
PIP: 3
Topic: PlatON网络优化升级提案
Author: alliswell
Status: Vote
Type: Text
Description: PlatON网络优化升级，修复已知问题
Created: 2021-06-01
---

# PIP-3：PlatON网络优化升级提案

## 目的

对PlatON底层节点网络进行升级，修复自主网启动以来影响用户体验且未解决的问题。

## 新特性

无

## 优化功能

- 优化交易传播策略，对于不直接广播交易的节点，发送交易hash值[issue-1780](https://github.com/PlatONnetwork/PlatON-Go/issues/1780)

- 支持RPC返回chainid的特性（参考EIP-695）

- 开放获取Slashing状态的getWaitSlashingNodeList接口[issue-1787](https://github.com/PlatONnetwork/PlatON-Go/issues/1787)

- 根据社区提议对Alaya网络随机性选举节点出块，累积二项分布函数期望值由30调整为43，候选节点增加洗牌以增加随机性，具体请参考[issue-1785](https://github.com/PlatONnetwork/PlatON-Go/issues/1785)，[讨论](https://forum.latticex.foundation/t/topic/4119)

## bug修复

- 修复fast同步没有完成的情况下重启节点失败[issue-1775](https://github.com/PlatONnetwork/PlatON-Go/issues/1775)

- 修复预估gas接口时，对于治理合约的预估，必须要传入gasPrice的问题[issue-1758](https://github.com/PlatONnetwork/PlatON-Go/issues/1758)

- 修复call调用偶现返回-32000错误码问题[issue-1769](https://github.com/PlatONnetwork/PlatON-Go/issues/1769)

- 修复节点fast同步失败后出现 `BAD BLOCK` 的问题[issue-1783](https://github.com/PlatONnetwork/PlatON-Go/issues/1783)

- 修复WASM跨合约调用时 `platon_caller` 值错误问题[issue-1779](https://github.com/PlatONnetwork/PlatON-Go/issues/1779)

- 同步修复以太坊txpool批量插入交易返回值错乱问题[ETH-21683](https://github.com/ethereum/go-ethereum/pull/21683)

## 说明

  本次升级将兼容历史数据，需要链上治理升级。详见讨论[链接](https://forum.latticex.foundation/t/topic/5113)

## 版本信息

本次升级的版本号为：1.1.0

Commit-ID: f86c9f8469105ab1969316aefe259b9b5ab3308e

