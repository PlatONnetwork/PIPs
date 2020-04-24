---
PIP:  8
Topic: 0.12.0升级提案
Author: alliswell
Status: Final 
Type: Upgrade
Description: 存储优化和节点出块顺序随机化功能升级
Created: 2020-04-24
---

# PIP-8：PlatON版本升级-0.12.0

## 目的

近期[关于节点出块顺序随机化的提案](https://github.com/PlatONnetwork/PIPs/blob/master/TestNet/PIP-7.md)经过重复讨论和论证，建议合理且方法可行，本次升级将支持这个提案，同时针对原0.11.0版本临时通过引用计数修改的存储问题在本版本从根本上进行了修复和优化。另外，由于在同一个区块节点质押后立即对其委托会导致委托详情不准确问题，本次升级一并进行优化。


## 新特性

[节点出块顺序随机化](https://github.com/PlatONnetwork/PIPs/blob/master/TestNet/PIP-7.md)

## 优化功能

- Storage存储优化

  将上个版本中对value增加引用计数的方式修改为value中增加Keccake256(address+key)以区分不同的key，防止value被优化机制清理掉

- 某一节点发起的质押交易和对该节点的委托交易不能出现在同一个区块

  因节点可以在由于期内质押后立即解质押，当质押、解质押和再质押发生在同一个区块时会导致委托在第一次质押上的详情（其实是失效的）不准确，且此种场景不符合实际情况（正常情况下质押节点交易要先上链后才会对其委托），故在程序中对质押后立即委托进行限制。

## bug修复

修复了经济模型的配置参数没有参与计算创世区块hash，导致不同参数初始化后创世区块相同的问题[#1333](https://github.com/PlatONnetwork/PlatON-Go/pull/1333)

## 说明

本次升级将重置网络，不对历史数据做兼容，不走链上升级，无需投票。详见[链接](https://forum.latticex.foundation/t/topic/2430)

## 版本信息

本次升级的版本号为：0.12.0

Commit-ID: f0b54edeb8b0e0ddd6cf4f46bbc4b90695bae701
