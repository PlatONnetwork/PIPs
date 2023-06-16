---
PIP: 13
Topic: PlatON网络BLS公钥更新提案
Author: TraceBundy、benbaley
Status: Draft 
Type: Upgrade
Description: 在PlatON主网络将BLS签名以及验签算法适配
Created: 2023-06-15
---

## 背景

PlatON最早于2018年9月开始基于BLS聚合签名设计Giskard共识协议，核心开发团队最初基于[herumi](https://github.com/herumi/bls)实现的bls签名算法开发，使用的是`default`方式（即PublicKey->G2,Signature->G1）,在以太坊团队提出[EIP-2537](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2537.md)后，越来越多的应用开始逐渐倾向于使用ETH2方式（即PublicKey->G1,Signature->G2）支持BLS，为了保持和EVM生态的兼容性，PlatON应当适配ETH2方式。

## 概述

PlatON的Giskard算法要求验证人公开BLS的PubKey，新算法的PubKey使用G1，因此升级需要验证人重新声明新的公钥，这样才能保证Giskard能正确的适配新BLS算法。

## 原理

| Cyclic Group | PlatON    | ETH2      | Length   |
| ------------ | --------- | --------- | -------- |
| Fr           | SecretKey | SecretKey | 32 bytes |
| G1           | Signature | PublicKey | 48 bytes |
| G2           | PublicKey | Signature | 96 bytes |

ETH2算法中PublicKey是48字节，而且在没有私钥的情况下新公钥无法从旧公钥推导出来，因此要保证Giskard不间断运行，需要在算法升级前先将各验证人新的公钥收集到合约中，收集完成后才能在一个统一的区块高度切换新算法。

## 实现

- 现状
  
  | 名称  | PlatON | ETH2 |
  | --- | ------ | ---- |
  | 私钥  | 小端 | 大端 |
  | 公钥  | 小端 | 大端 |
  | 签名  | 小端 | 大端 |

- 目标

封装bls相关的接口：

```golang

// SecretKey represents a BLS secret or private key.
type SecretKey interface {
	PublicKey() PublicKey
	//P2() []byte
	Sign(msg []byte) Signature
	Marshal() []byte
	SetLittleEndian(buf []byte) error
	GetLittleEndian() []byte
	MakeSchnorrNIZKP() SchnorrProof
}

// PublicKey represents a BLS public key.
type PublicKey interface {
	Marshal() []byte
	Copy() PublicKey
	Aggregate(p2 PublicKey) PublicKey
	IsInfinite() bool
	Equals(p2 PublicKey) bool
}

// Signature represents a BLS signature.
type Signature interface {
	Verify(pubKey PublicKey, msg []byte) bool
	// Deprecated: Use FastAggregateVerify or use this method in spectests only.
	AggregateVerify(pubKeys []PublicKey, msgs [][32]byte) bool
	FastAggregateVerify(pubKeys []PublicKey, msg [32]byte) bool
	Eth2FastAggregateVerify(pubKeys []PublicKey, msg [32]byte) bool
	Marshal() []byte
	Copy() Signature
}

type SchnorrProof interface {
	Marshal() []byte
	Unmarshal([]byte) error
	Verify(key PublicKey) error
}

```

对于同一个私钥对应的swpPubKey和ethPubkey，使用以下方法转换：

```golang

func (s *SecretKey) createKey() {
	if s.key == nil {
		switch BlsVersion {
		case Bls12381:
			s.key = &eth.SecretKey{}
		case Bls12381Swap:
			s.key = &swap.SecretKey{}
		}
	}
}

func (s *SecretKey) ToSwapKey() *SecretKey {
	key := &swap.SecretKey{}
	key.SetLittleEndian(s.GetLittleEndian())
	return &SecretKey{
		key: key,
	}
}

func (s *SecretKey) ToEthKey() *SecretKey {
	key := &eth.SecretKey{}
	key.SetLittleEndian(s.GetLittleEndian())
	return &SecretKey{
		key: key,
	}
}

```

使用key和验证signature根据具体的场景创建不同的实例，动态调用。


## 计划

完成BLS算法更新需要2个阶段：

阶段1：
在1.5.0中启用新版本声明和提案投票交易中声明新PublicKey。

阶段2：
在1.6.0升级过程中通过提案投票或版本声明收集各个验证人新的BLS公钥，提案通过后启用新BLS算法。

## 影响分析

本提案共分2次链上升级，除验证人节点需要重新声明新PubKey外无其他影响，阶段2完成升级后新BLS签名和验签算法和以太坊2.0兼容。

## 链接

- [IETF BLS signature draft standard v4](https://tools.ietf.org/html/draft-irtf-cfrg-bls-signature-04)
