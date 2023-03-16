---
PIP:  10
Topic: 1.3.0升级提案
Author: alliswell
Status: Final
Type: Upgrade
Description: 支持委托锁定，EVM获取链ID返回PlatON新链ID 210425
Created: 2022-08-08
---

# PIP-10：PlatON版本升级-1.3.0

## 目的

关于PlatON支持委托锁定的提案经过反复讨论和论证，提案合理且方法可行，本次升级的主要内容包括了该提案的实施，同时关于PlatON支持新链ID的提案PIP-7第二阶段已经于两个月前实施完毕，第三阶段将在本次提案中完成。

## 新特性

- [PlatON对委托锁定提案的支持实施](https://github.com/PlatONnetwork/PIPs/blob/master/PIPs/PIP-6.md)
- [PlatON支持新链ID的第三阶段实施](https://github.com/PlatONnetwork/PIPs/blob/master/PIPs/PIP-7.md)

## 优化内容

本次升级同时对以太坊1.9.12以前的版本优化内容做了更新。

## 影响说明

### 委托相关

1. 撤销已生效的委托时,Token不是立即到账的

升级后，被撤销委托的Token将被锁定56个结算周期（默认值，可治理），锁定期结束后需要由委托用户主动发起“赎回”操作才可以到账。

2. 处于锁定期的Token，仍可以继续用于委托

用于委托的Token因此包含3种类型：自由金额（用户的balance）、用户在锁仓计划中的待释放金额和处于锁定期的金额。

3. 重新委托的锁定Token，仍要经历犹豫期

处于锁定期的Token在被重新委托后的第一个结算周期仍旧是“犹豫期”，“犹豫期”过后才进入生效期，处于“犹豫期”的委托，撤销后仍要被锁定，切锁定期是重新计算的。

### 链ID相关

Solidity中使用assembly`chainid`指令或者`block.chainid`指令获取到的链ID将由旧值100调整为新值210425

在Solidity中使用过以上指令的应用，需要从应用层重新适配，以保证业务能正常运行。

## 版本信息

本次升级的版本号为：1.3.0

Commit-ID: *8ddcea2d05d9b45efb6aa71a1ec49b006c957df3*
