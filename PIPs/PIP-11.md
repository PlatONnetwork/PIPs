---
PIP:  11
Topic: 1.4.0升级提案
Author: alliswell
Status: Draft
Type: Upgrade
Description: 轻节点协议支持、pre-eip155交易的兼容
Created: 2023-02-17
---

# PIP-11：PlatON版本升级-1.4.0

## 目的

通过对区块和交易信息中增加对map protocol、eip-2470等合理化提案的适配，兼容EVM的同时让PlatON能够更灵活的适配不同应用，进一步促进生态繁荣。

## 新特性

1. Block.Header.Extra字段中增加验证人摘要

PlatON是基于权益证明（PoS）共识的公链，根据轻节点客户端协议Map protocol的开发需求，根据当前的区块header信息无法对PlatON共识validators进行验证，本提案通过在扩展字段Extra中加入对应共识轮validators的摘要（hash）信息来满足对共识验证人list验证的需求。

- 具体规则：

block.extra: 前32字节存储验证人list的hash值

写入hash时间：每共识轮最后一个区块写入

hash含义： 下一个共识轮节点列表的rlp后的hash值。

hash规则：

```
// 对共识轮节点列表做rlp编码
rlpValue := rlp.encode([{pubKey1,blsPubKey2},{pubKey1,blsPubKey2},...])
// 对编码后的数据，算hash
hash := Keccak256(rlpValue)
```

- 新增RPC接口：

GetValidatorByBlockNumber

通过`RPC`调用，参数和返回值如下：

> 参数

| 字段          | 类型     | 描述   |
| ----------- | ------ | ---- |
| blockNumber | uint64 | 区块高度 |

> 返回数据
> 
> 类型：集合，按出块顺序排列

| 字段        | 类型     | 描述      |
| --------- | ------ | ------- |
| address   | 20byte | 节点地址    |
| pubKey    | 64byte | 节点公钥    |
| blsPubKey | 96byte | 节点bls公钥 |

2. 对pre-eip155交易的兼容

根据[EIP-2470](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2470.md)中关于DApps需要在不同链中使用同一个instance需求的解决方案，链底层需要对无链ID签名（pre-eip155）交易兼容才能实现。为进一步促进PlatON生态繁荣，应该兼容pre-eip155交易。

- 具体规则：

启动参数：同步以太坊v1.10.0中关于[non eip155 replay protected tx](https://github.com/ethereum/go-ethereum/pull/22339)内容，增加启动参数`rpc.allow-unprotected-txs`允许rpc提交pre-eip155类型的交易

交易的执行：允许pre-eip155类型的交易被打包到区块中

## 影响说明

1. 本提案涉及区块结构信息的变更，提案通过后旧版本客户端无法继续同步新区块

2. PlatON的交易签名中支持100和210425两个chainid，同时也支持不带chainid，通过托管在节点keystore中的钱包unlock方式发送交易将默认使用210425签名交易

## 版本信息

本次升级的版本号为：1.4.0

Commit-ID: *[待定]*
