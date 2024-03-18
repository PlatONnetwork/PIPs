---
PIP:  13
Topic: 1.5.0升级提案
Author: alliswell
Status: Draft
Type: Upgrade
Description: 移除对旧链ID（100）的支持，适配以太坊柏林升级和伦敦升级
Created: 2023-12-08
---

# PIP-13：PlatON版本升级-1.5.0

## 目的

`PlatON`自[PIP-7](https://github.com/PlatONnetwork/PIPs/blob/master/PIPs/PIP-7.md)提案开始支持双链ID已经过了三个`Minor`版本的过渡期，当前绝大部分应用都已经调整到使用新的链ID（210425），故此，本次升级将按计划实施[PIP-7](https://github.com/PlatONnetwork/PIPs/blob/master/PIPs/PIP-7.md)提案的第四阶段：停止对旧链ID的支持。

除此之外，本次升级`PlatON`将启用对以太坊[柏林升级](https://ethereum.org/en/history/#berlin)和[伦敦升级](https://ethereum.org/en/history/#london)的适配，开始支持`EIP-2718`、`EIP-2930`、`EIP-1559`等类型的交易。

## 新特性

- PIP-7提案的第四阶段（[停止对旧链ID的支持](https://github.com/PlatONnetwork/PIPs/blob/master/PIPs/PIP-7.md)）
- 启用对柏林升级以及伦敦升级的适配

## 优化内容

- 本次升级同时对以太坊`1.11.0`以前的版本优化内容做了更新
- fast同步将保持交易回执信息，gc模式不再删除交易回执

## 影响说明

### 链ID相关

升级后PlatON网络将不再支持交易签名中带链ID值为100的交易，仍在使用旧链ID的应用需要在升级前完成旧链ID到新链ID的迁移，以保证业务能正常运行。

## 版本信息

本次升级的版本号为：1.5.0

Commit-ID: *8a960a30aeae89e01e39a732ecbbf3888a260a7a*
