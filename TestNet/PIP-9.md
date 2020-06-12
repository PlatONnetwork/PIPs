---
PIP:  9
Topic: 关于对PlatON地址格式变更的提议
Author: alliswell
Status: Draft 
Type: Requirement
Discussions-to: https://forum.latticex.foundation/t/topic/2529
Description: 提议PlatON地址格式支持bech32编码规范
Created: 2020-05-19
---

# PIP-9：提议PlatON地址支持bech32编码

## 背景

PlatON底层最初基于ETH开发，地址格式沿用了ETH的hex+EIP55规范，这种地址格式不能直观的判断具体是属于ETH还是PlatON的地址，不利于用户区分和使用。

## 目的

为了能让用户能直观的判断PlatON地址，提议PlatON支持带固定前缀的bech32地址编码规范。

## 内容

### 对外地址修改
PlatON底层使用byte的形式存储地址，当地址需要显示给用户时，当前的规则是将地址转成16进制字符串，然后采用EIP55的标准对所得字符串中的部分字母转大写，得到最终地址字符串，例如:
 0x7cB0f5D66D07472A835C97e288931BE2518f7b21
这种字符串虽然可以通过计算机校验（是否符合EIP55规范），但对用户来说辨识度不高。

在对各主流公链的地址生成进行充分调研后，决定采用bech32的方式对内部地址编解码成字符串来对外显示，长度与原编码方式长度保持一致,均为42，hrp（human readable part)为lat（主网）和lax（其他网络），例如：
 lax10jc0t4ndqarj4q6ujl3g3ycmufgc77epxg02lt

```
公钥:aa396ac2ebb0fe6c8addde0ee65a53b38ec5d976f1af737b0ad85b039fa3b451d8ec6d4d27ff06b4db32da3f49fbe0d66cd80fe1a86ae87f6e626a1eed5e0b31
公钥->内部地址(byte) = Keccak256(公钥.byte[1:])[12:] 
// 原地址字符串,长度42
对外地址(string) =  EIP55(hex(内部地址)) =0x7cB0f5D66D07472A835C97e288931BE2518f7b21
// 新版的地址字符串,长度42
对外地址(string) = bech32("lax",内部地址) =lax10jc0t4ndqarj4q6ujl3g3ycmufgc77epxg02lt
```

注：
  - 主网前缀为`lat`，其他网络前缀为`lax`
  - bech32 基于base32的一种编码方式，可以快速交易生成的地址并且生成的地址具有辨识度，官方文档说明 https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki  
  - bech32 各语言实现，https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki#reference-implementations   , 注意要去掉带version的实现，python sdk和js sdk 都踩过坑，注意避免 

go版本参考实现如下

bech32编码步骤: 
1. 将内部地址使用base32进行编码data
2. 使用bech32.Encode(前缀，data)进行编码

`部分实现中带version，如bech32.Encode(前缀，version，data），注意不要使用这种实现`

```
package bech32

import (
	"github.com/btcsuite/btcutil/bech32"
	"github.com/pkg/errors"
)

//ConvertAndEncode converts from a base64 encoded byte string to base32 encoded byte string and then to bech32
func ConvertAndEncode(hrp string, data []byte) (string, error) {
    //1.此处是进行base32转换
	converted, err := bech32.ConvertBits(data, 8, 5, true)
	if err != nil {
		return "", errors.Wrap(err, "encoding bech32 failed")
	}
	2. 使用bech32进行编码
	return bech32.Encode(hrp, converted)

}

//DecodeAndConvert decodes a bech32 encoded string and converts to base64 encoded bytes
func DecodeAndConvert(bech string) (string, []byte, error) {
	hrp, data, err := bech32.Decode(bech)
	if err != nil {
		return "", nil, errors.Wrap(err, "decoding bech32 failed")
	}
	converted, err := bech32.ConvertBits(data, 5, 8, false)
	if err != nil {
		return "", nil, errors.Wrap(err, "decoding bech32 failed")
	}
	return hrp, converted, nil
}

```


### 底层keystore修改

更改了keystore生成的地址文件里面的的address这一项，增加主网和测试网的地址

```
//文件名不变  UTC--2020-04-22T10-34-50.846182000Z--fc21fcfe23faea9e3776882f372ed65a7f8e1b64
原keystore内容
{
    "address": "fc21fcfe23faea9e3776882f372ed65a7f8e1b64",
    "crypto": {
        "cipher": "aes-128-ctr",
        "ciphertext": "a912aae85664e08bb8f083fcd5a4e3e2c9567d00d9dd47c489787c24994f5e69",
        "cipherparams": {
            "iv": "161dd177d37091f0f5e61496661a93b1"
        },
        "kdf": "scrypt",
        "kdfparams": {
            "dklen": 32,
            "n": 262144,
            "p": 1,
            "r": 8,
            "salt": "a24f437747d9c671d0e42ac041fd5c4bdc8b23a1ec6f32054628e1ca19a1e945"
        },
        "mac": "e6c9238aa563afb33cd6ce9d87daa47ebb9c329b41d9ebacedd10d1a2ef49063"
    },
    "id": "c46cceeb-66ec-42d5-b69f-24e766bc2e36",
    "version": 3
}


现keystore
{
    "address": {
        "mainnet": "lat1fzpmtwrlmdv925xk8e9n8v9k6f68wl5lc88wzy",
        "testnet": "lax1lsslel3rlt4fudmk3qhnwtkktflcuxmyu0eku7"
    },
    "crypto": {
        "cipher": "aes-128-ctr",
        "ciphertext": "a912aae85664e08bb8f083fcd5a4e3e2c9567d00d9dd47c489787c24994f5e69",
        "cipherparams": {
            "iv": "161dd177d37091f0f5e61496661a93b1"
        },
        "kdf": "scrypt",
        "kdfparams": {
            "dklen": 32,
            "n": 262144,
            "p": 1,
            "r": 8,
            "salt": "a24f437747d9c671d0e42ac041fd5c4bdc8b23a1ec6f32054628e1ca19a1e945"
        },
        "mac": "e6c9238aa563afb33cd6ce9d87daa47ebb9c329b41d9ebacedd10d1a2ef49063"
    },
    "id": "c46cceeb-66ec-42d5-b69f-24e766bc2e36",
    "version": 3
}

```


注: 
1. 根据此次修改，主网和测试网的地址的密码相同
2. 同一个keysotre文件可以供主网和测试网同时使用

### 对外接口修改


- sendRawTransaction接口，to地址需要使用新格式地址，底层在验证交易内容时根据当前网络chainid校验to地址是否为合法地址（除chainid==100外其他都需要lax格式）。

- sendTransaction接口（代理签名交易接口）和call调用，from和to都要按新地址规则生成

- platon account new子命令和personal.newAccount() rpc接口入参不变，返回参数中同时返回2个地址，即mainnet和testnet

- 所有rpc查询接口，入参全部改为新地址格式，返回值地址改为新地址格式

- 各内置合约地址变更如下：

  | 内置合约 |原地址| 新地址                                                         |
  | -------- |----| ------------------------------------------------------------ |
  | 锁仓     |0x1000000000000000000000000000000000000001| 主网：lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqp7pn3ep                                                         测试网：lax1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqp3yp7hw |
  | Staking  |0x1000000000000000000000000000000000000002| 主网： lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqzsjx8h7                                                        测试网：lax1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqzlh5ge3 |
  | 奖励     |0x1000000000000000000000000000000000000003| 主网： lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqrdyjj2v                                                        测试网： lax1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqrzpqayr|
  | 处罚     |0x1000000000000000000000000000000000000004| 主网：    lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqyva9ztf                                                     测试网：lax1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqyrchd9x |
  | 治理     |0x1000000000000000000000000000000000000005| 主网： lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq93t3hkm                                                        测试网：lax1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq97wrcc5 |
  | 委托奖励 |0x1000000000000000000000000000000000000006| 主网：lat1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqxlcypcy                                                         测试网：lax1zqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqxsakwkt |

