---
pip: 13
title: 隐私代币合约标准
author:
status: Draft
type: Standards Track
category: ARC
created: 2020-12-7
---

## Simple Summary

一种实现隐私代币合约的标准接口

## Abstract

该标准意在帮助实现一系列标准接口，利用 WASM 智能合约来支持隐私代币。标准提供了铸币，销毁，与标准代币进行交互，实现将标准代币转账进行匿名化

## Motivation

目前的代币标准都是基于账户模型下进行代币的转移，代币的流转过程可以很容易的被追踪到。急需一套隐私代币标准来隐藏交易双方，dAPP 可以基于标准接口来实现相关应用。标准不仅可以实现新的的匿名代币，同时也可以将标准代币进行匿名化。在用户身份上也可摆脱 ECDSA 算法，利用更多密码算法实现新的用户身份。

目前隐私公链多数采用 [UTXO](https://en.wikipedia.org/wiki/Unspent_transaction_output) 模型，诸如 [Zcash](https://en.wikipedia.org/wiki/Zcash)，[Monero](https://en.wikipedia.org/wiki/Monero_(cryptocurrency))。零知识证明近几年发展非常迅速，如主流的 [Groth16](https://eprint.iacr.org/2016/260.pdf), [PLONK](https://eprint.iacr.org/2019/953.pdf)，[Bulletproof](https://eprint.iacr.org/2017/1066.pdf) 等等，具备证明小，验证时间短，适用于在智能合约中使用。那么 用 UTXO+ZK 两者结合来实现智能合约上的隐私代币是较为理想的方式。以下接口规范将以 UTXO+ZK 的基础进行设计。

## Specification

### 接口规范

#### Transfer

``` C++
ACTION bool Transfer(const bytes &proof)

```

**Brief:** 隐私交易转账操作

**Param:** proof 转账证明

**Return:** 成功返回 true, 失败返回 false

#### Name

``` C++
CONST std::string Name()
```

**Brief:** 隐私代币名称

**Param:** 无

**Return:** 代币名称

**OPTIONAL** 推荐实现该接口来提升钱包及交易所的可用性，但其它合约不能依赖其接口的存在

#### Symbol

``` C++
CONST std::string Symbol()

```

**Brief:** 隐私代币符号

**Param:** 无

**Return:** 符号名称

**OPTIONAL** 推荐实现该接口来提升钱包及交易所的可用性，但其它合约不能依赖其接口的存在

#### ScalingFactor

``` C++
CONST u128 ScalingFactor()
```

**Brief:** 与标准代币兑换比例

**Param:** 无

**Return:** 兑换比例

**OPTIONAL** 推荐实现该接口来提升钱包及交易所的可用性，但其它合约不能依赖其接口的存在

#### TotalSupply

``` C++
CONST u128 TotalSupply()

```

**Brief:** 代币供给总量

**Param:** 无

**Return:** 供给总量

**OPTIONAL** 推荐实现该接口来提升钱包及交易所的可用性，但其它合约不能依赖其接口的存在

#### UpdateMetaData

``` C++
ACTION bool UpdateMetaData(const h256 &note_hash, const bytes &meta_data,
                             const bytes &signature)
```

**Brief:** 每笔 UTXO 的 Output 更新元数据信息，由发送方进行更改

**Param:** note_hash UTXO 中 输出的 note

**Param:** meta_data 需要更新的元数据信息

**Param:** signature 签名，含有该 note 的转账交易的发送者签名

**Return:** 更新是否成功

**OPTIONAL** 推荐实现该接口来提升钱包及交易所的可用性，但其它合约不能依赖其接口的存在

#### Approve

``` C++
ACTION bool Approve(const bytes &data)

```

**Brief:** note 拥有者授权给第三方花费该 note

**Param:** data 授权信息

**Return:** 授权是否成功

**OPTIONAL** 授权由不同算法来确定是否拥有授权功能，其它合约不能依赖其接口的存在

#### GetApproval

``` C++
CONST bytes GetApproval(const h256 &note_hash)
```

**Brief:** 获取指定 note 的授权信息

**Param:** note_hash note 的 hash 值

**Return:** 返回授权信息

**OPTIONAL** 授权由不同算法来确定是否拥有授权功能，其它合约不能依赖其接口的存在

#### Mint

``` C++
ACTION bool Mint(const bytes &proof)

```

**Brief:** 铸币

**Param:** 铸币证明

**Return:** 返回是否成功

**OPTIONAL** 授权由不同代币需求来确定是否拥有授权功能，其它合约不能依赖其接口的存在

#### Burn

``` C++
ACTION bool Burn(const bytes &proof)
```

**Brief:** 销毁代币

**Param:** 销毁证明

**Return:** 返回是否成功

**OPTIONAL** 授权由不同代币需求来确定是否拥有授权功能，其它合约不能依赖其接口的存

### 事件

#### CreateNoteEvent

``` C++
PLATON_EVENT2(CreateNoteEvent, const bytes &owner_index, const h256 &note_hash_index, const bytes &owner)

```

**Brief:** 每笔交易新增的 output 都会触发该事件

**Param:** owner_index owner 索引

**Param:** note_hash_index note hash 值索引

**Param:** owner 根据不同密码算法来确定 owner，owner 可能是一把公钥或者多把公钥的组合

#### DestroyNoteEvent

``` C++
PLATON_EVENT2(DestroyNoteEvent, const bytes &owner_index, const h256 &note_hash_index, const bytes &owner)
```

**Brief:** 每笔交易花费的 input 都会触发该事件

**Param:** owner_index owner 索引

**Param:** note_hash_index note hash 值索引

**Param:** owner 根据不同密码算法来确定 owner，owner 可能是一把公钥或者多把公钥的组合

#### ApproveEvent

``` C++
PLATON_EVENT1(ApproveEvent, const h256 &note_hash_index, const bytes &data)

```

**Brief:** 每笔成功的授权交易都会触发该事件

**Param:** note_hash_index note hash 值索引

**Param:** data 授权数据

#### MetaDataEvent

``` C++
PLATON_EVENT1(MetaDataEvent, const h256 &note_hash_index, const bytes &data)
```

**Brief:** 每笔更新元数据交易都会触发该事件

**Param:** note_hash_index note hash 值索引

**Param:** data 元数据

#### MintEvent

``` C++
PLATON_EVENT0(MintEvent, const h256 &mint_hash, u128 value);

```

**Brief:** 每笔铸币交易都会触发该事件

**Param:** mint_hash 铸币交易的 hash 值

**Param:** value 铸币金额

#### BurnEvent

``` C++
PLATON_EVENT0(BurnEvent, const h256 &burn_hash, u128);
```

**Brief:** 每笔销毁交易都会触发该事件

**Param:** burn_hash 销毁交易的 hash 值

**Param:** value 销毁金额

## Implementation

### 合约架构

隐私代币的合约涉及到密码算法，采用算法与存储分离架构。做到密码算法的可升级，避免算法漏洞带来的资金损失。具体架构如下：

```
                                           +---------+
                                           |Validator|
    +---------+                            +----+----+
    |  Token1 +------->+---------+              ^
    +---------+        |         |              |
    |  Token2 +------->+   ACL   +--------------+
    +---------+        |         |              |
    |  Token3 +------->+-+--+--+-+              v
    +---------+          |  |  |           +----+----+
                         |  |  |           | Storage |
                         |  |  |           +---------+
                         |  |  |
         +---------------+  |  +------------+
         |                  |               |
         v                  v               v
+--------+-------+  +-------+--------+  +---+------------+
| StandardToken1 |  | StandardToken2 |  | StandardToken3 |
+----------------+  +----------------+  +----------------+

```

**Token1, Token2, Token3** 为隐私代币，实现隐私代币规范接口，每个隐私代币都可与标准代币进行对应，实现标准代币向隐私代币进行存取操作。隐私代币也可以不与标准代币进行互通，独立进行发行

**StandardToken1, StandardToken2, StandardToken3** 为标准代币，实现标准代币规范接口

**ACL** 访问控制合约，实现隐私代币的公共逻辑，作为代币与其它模块的桥梁。主要提供转账公共逻辑，验证算法与存储模块的版本控制及升级

**Validator** 验证合约，实现对各种算法的验证功能，每种验证合约可以根据算法自定义账户地址

**Storage** 存储合约负责存储每笔转账的必要信息，如 UTXO 中 output 的 owner 和 hash 值

### 协议格式

考虑到不同算法，证明结构的不同，协议中没有对交易证明进行严格格式限定。下面是一种协议格式，供参考：

``` C++
struct ProofData {
  uint32_t version;
  platon::bytes zk_proof;
  platon::bytes extra_data;
};

struct Proof {
  ProofData proof_data;
  platon::bytes signature;
};

```

**Proof** 交易证明， proof_data 证明数据，signature 签名用来保证 proof_data 完整性与不配篡改

**ProofData** version 用来标识版本，zk_proof 零知识证明所需要信息， extra_data 交易中非证明数据信息

#### 验证合约

验证合约定义了协议接口来扩展算法。

``` C++
class ValidatorInterface {
  virtual platon::bytes ValidateProof(const platon::bytes& proof) = 0;
  virtual bool ValidateSignature(const platon::bytes& pk, const platon::h256& hash, const platon::bytes& signature) = 0;
};
```

**ValidateProof** 验证证明并返回验证结果，具体的交易类型，有验证合约解析字节流获得具体类型

**ValidateSignature** 验证签名，通常用作更新信息。签名是对消息的 hash 进行签名

#### 验证结果

目前交易验证类型有：

**Transfer** 表示转账操作，同时拥有 input 和 output，只是隐私代币的内部转账，配平关系 input = output

**Deposit** 表示存款操作，由标准代币向隐私代币进行存入操作，存款的配平关系为 input =  output - deposit

**Withdraw** 表示提取操作，由隐私代币向标准代币进行提取操作，提取的配平关系为 input = output + withdraw

**Mint** 表示铸币操作

**Burn** 表示销毁操作

**Approve** 表示授权操作

##### TransferResult

```
struct TransferResult {
  InputNotes inputs;
  OutputNotes outputs;
  platon::Address public_owner;
  i128 public_value;
  platon::bytes sender;
};
```

代表了 Transfer， Deposit， Withdraw 三种类型验证结果。其中 inputs 与 outputs 为输入输出，如果是 Deposit， Withdraw 两种类型，那么 public_owner 与 public_value 会被写入具体的地址和金额。 sender 表示了该笔交易的发送方

##### MintResult

```
struct MintResult {
  platon::h256 old_mint_hash;
  platon::h256 new_mint_hash;
  platon::u128 total_mint;
  OutputNotes outputs;
  platon::bytes sender;
};
```

铸币类型验证结果，old_mint_hash 上一次铸币交易的 hash 值，new_mint_hash 本次铸币交易的 hash 值
total_mint 本次铸币的总量，outputs 可花费的 note， sender 表示了该笔交易的发送方

##### BurnResult

```
struct BurnResult {
  platon::h256 old_burn_hash;
  platon::h256 new_burn_hash;
  platon::u128 total_burn;
  InputNotes inputs;
  platon::bytes sender;
};
```

销毁类型验证结果，old_burn_hash 上一次销毁交易的 hash 值， new_burn_hash 本次销毁交易的 hash 值
total_burn 本次销毁代币的总量，inputs 销毁的 note， sender 表示该笔交易的发送方

##### ApproveResult

```
struct ApproveResult {
  platon::h256 note_hash;
  platon::bytes approve_data;
  platon::bytes sender;
};
```

授权类型验证结果，note_hash 指定的可花费 note 的 hash 值，approve_data 授权数据， sender 交易发送方

#### 存储合约

``` C++
class StorageInterface {

    virtual void Transfer(const platon::bytes &result) = 0;
    virtual void Mint(const platon::bytes &result) = 0;
    virtual void Burn(const platon::bytes &result) = 0;
    virtual void Approve(const platon::bytes &result) = 0;
    virtual platon::bytes GetApproval(const platon::h256 &note_hash) = 0;
    virtual NoteStatus GetNote(const platon::h256 &note_hash) = 0;

};

```

**Transfer，Mint，Burn，Approve** 分别对应上述的交易类型

**GetApproval，GetNote** 则是根据 hash 进行授权数据与 note 信息的查询接口

#### ACL

控制合约实现了各个功能的核心逻辑，对接各个代币。ACL 有如下核心接口

``` C++
class AclInterface {
    virtual bool CreateRegistry(uint32_t validator_version, uint32_t storage_version, u128 scaling_factor, const platon::Address& token_address, bool can_mint_burn) = 0;
    virtual platon::bytes Transfer(const platon::bytes& proof) = 0;
    virtual platon::bytes Mint(const platon::bytes& proof) = 0;
    virtual platon::bytes Burn(const platon::bytes& proof) = 0;
    virtual platon::bytes Approve(const platon::bytes& proof) = 0;
    virtual void UpdateNotes(const platon::bytes& proof) = 0;
};
```

**CreateRegistry** 隐私代币合约通过此接口向 ACL 注册，validator_version 为验证合约版本，storage_version 为存储合约版本，scaling_factor 为与标准代币兑换比例，token_address 为标准代币的地址，can_mint_burn 是否允许铸币与销毁操作

**Transfer** 实现了 Transfer， Deposit， Withdraw 3 种类型交易

**Mint, Burn, Approve** 分别实现铸币，销毁，授权

**UpdateNotes** 实现了更新 note 元数据

#### 注册调用顺序

```
+--------+   1     +--------+
|        +-------->+        |       2
| Issuer |         |  Token +---------------+         3        +------------+
|        +<--------+----+---+               |    +------------>+  Validator |
+--------+   6          |                   |    |             +------------+
                        |                   v    |
                        |       5       +---+----++
                        +---------------+   ACL   |
                                        +-------+-+
                                                |
                                                |      4        +---------+
                                                +-------------->+ Storage |
                                                                +---------+
```

1. 发行方根据需求算法选择相应的验证合约与存储合约版本，进行代币的部署

2. 隐私代币合约调用 ACL 合约 CreateRegistry 接口进行注册

3. ACL 根据对应版本进行验证合约

4. ACL 根据对应版本进行存储合约的部署

4. ACL 返回给隐私代币注册状态

5. 发行方通过链上获取隐私代币注册状态

#### 合约调用顺序

```
+--------+       1
|  Token +---------------+         2        +------------+
+----+---+               |    +------------>+  Validator |
     ^                   |    |             +------------+
     |                   v    |
     |       5       +---+----++
     +---------------+   ACL   |
                     +---+---+-+
                         |   |
                         |   |      3        +---------+
                         |   +-------------->+ Storage |
                       4 |                   +---------+
                         |
                         v
                    +----+----------+
                    | StandardToken |
                    +---------------+
```

1. 隐私代币合约将证明传入到 ACL 合约

2. ACL 找到 Token 对应的验证合约进行验证

3. 验证成功，调用存储合约，将验证结果保存

4. 如果是 Deposit，Withdraw 交易则调用标准代币合约进行转账

5. ACL 将验证结果返回给隐私代币合约

### 升级

ACL 作为控制合约，提供验证合约与存储合约的版本管理与升级的功能。增强隐私代币的安全性，可扩展性。

#### ACL 合约管理接口

``` C++
class AclInterface {
  virtual bool UpdateValidator(uint32_t version) = 0;
  virtual bool CreateValidator(uint32_t version, const platon:: Address& addr, const std::string& description) = 0;
  virtual bool UpdateValidatorVersion(uint32_t version, const platon:: Address& addr, const std::string& description) = 0;
  virtual uint32_t ValidatorLatest(uint8_t name) = 0;
  virtual uint32_t ValidatorLatestMinor(uint8_t name, uint8_t major_version) = 0;

  virtual bool UpdateStorage(uint32_t version) = 0;
  virtual bool CreateStorage(uint32_t version, const platon:: Address& addr, const std::string& description) = 0;
  virtual bool UpdateStorageVersion(uint32_t version, const platon:: Address& addr, const std::string& description) = 0;
  virtual uint32_t StorageLatest(uint8_t name) = 0;
  virtual uint32_t StorageLatestMinor(uint8_t name, uint8_t major_version) = 0;
};

```

**UpdateValidator** 用于隐私代币验证合约更新版本，version 为已经在 ACL 合约中注册的合约版本。 由隐私代币合约来调用进行升级

**CreateValidator** 用于创建新算法验证合约

**UpdateValidatorVersion** 用于更新验证合约算法

**ValidatorLatest** 用来获取指定算法的最新版本

**ValidatorLatestMinor** 用来获取指定验证算法主版本的最新小版本

**UpdateStorage** 用于隐私代币存储合约更新版本，version 为已经在 ACL 合约中注册的合约版本。 由隐私代币合约来调用进行升级

**CreateStorage** 用于创建新存储类型合约

**UpdateStorageVersion** 用于更新存储合约

**StorageLatest** 用来获取指定存储合约类型的最新版本

**StorageLatestMinor** 用来获取指定存储合约主版本的最新小版本

#### 合约升级流程

```

                     +----------+
                     |  Admin   |
                     +-----+----+
                           |
+--------+      2          |1
|  Token +-------------+   |                 +------------+
+----+---+             |   |          +------+ Validator1 |
     ^                 |   |          |      +-----+------+
     |                 v   v          |            |
     |      4        +-+---+---+  3   |            |
     +---------------+   ACL   +------+            |
                     +---------+                   |
                                             +-----+------+
                                             | Validator2 |
                                             +------------+
```

1. 管理员（可能是发行方，也可能是多签合约地址）注册新版本验证合约到 ACL

2. Token 传入指定版本，调用 ACL 合约进行升级

3. ACL 调用当前验证合约升级接口，由验证合约自行迁移到新版本

4. ACL 返回给 Token 合约升级是否成功
