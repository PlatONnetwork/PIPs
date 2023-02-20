---
PIP: 8
Topic: PlatON网络对链上随机数的支持
Author: benbaley
Status: Final 
Type: Upgrade
Description: 在PlatON网络上支持链上获取可验证随机数
Created: 2022-03-28
---

## 背景

以NFT、`Loot drops`游戏等应用为代表的DApp极大的促进了区块链生态的繁荣，链上随机数可以为DApp提供防篡改的链上随机性，诸如NFT铸造和归属、draws、(PvP) Battles等都需要使用链上随机来产生公平的结果，为此，在PlatON网络上急需支持一套安全的、和应用具有良好兼容性的链上随机数解决方案。

## 概述

在公有链上通过共识算法独立产生不可预知的随机数是困难的，PlatON的共识算法[Giskard](https://devdocs.platon.network/docs/zh-CN/PlatON_Solution)中使用了VRF算法做验证人选取，其链上Nonce（VRF和证明）天然的具备安全、可验证以及随机性（不可预知）的特点，因此可以直接在PlatON中通过使用区块Header中的Nonce来满足链上（合约层）获取随机性的要求。

## 实现

由于solidity并未提供获取Nonce的指令，EVM类似的提案[EIP-4399](https://ethereum-magicians.org/t/eip-4399-supplant-difficulty-opcode-with-random/7368)也尚未进入实施阶段，为了保持和EVM良好的兼容性，PlatON上讲采用 PrecompiledContract 的方式在合约层支持用户获取随机数。

### 随机源

PlatON的链上随机数来源于区块header中的Nonce，该字段由父区块的Nonce做为种子，由当前区块的提议人私钥签名产生的随机数，天然具有可验证、随机性。使用时，取该字段的第[1,33]字节为可验证的随机数的随机源VRF。

PlatON网络中的所有验证人节点都将对区块Header中的Nonce字段做验证，如果该字段非法，则当前区块不能获取验证人的签名，无法被确认，因此该字段无需在合约层重复验证。

### 用户接口

- 预编译合约

系统通过PrecompiledContract的`VrfInnerContract`获取区块Header中的Nonce，合约地址为 `lat1xqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqpe9fgva` (`0x3000000000000000000000000000000000000001`)
调用时，用户需要传入产生随机数的个数， 该PrecompiledContract返回随机数数组。

- 调用

智能合约中可以使用`delegatecall`请求随机数，如：

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * 调用PlatON内置合约生成VRF随机数
 */
contract VRF {

  error InvalidRandomWords(uint32 numWords, uint256 returnValueLength);

  // VrfInnerContract 内置合约地址
  address vrfInnerContractAddr = 0x3000000000000000000000000000000000000001;

  // 32个字节表示uint256
  uint32 base = 32;

  /**
   * 调用 VrfInnerContract 内置合约生成VRF随机数
   * @param numWords 随机数的个数
   */
  function requestRandomWords(uint32 numWords) internal returns (uint256[] memory) {
    bytes memory data = abi.encode(numWords);
    bytes memory returnValue = assemblyCall(data, vrfInnerContractAddr);

    if (numWords * base != returnValue.length) {
        revert InvalidRandomWords(
            numWords,
            returnValue.length
        );
    }

    uint256[] memory randomWords = new uint256[](numWords);
    for(uint i = 0; i < numWords; i++) {
        uint start = i * base;
        randomWords[i] = sliceUint(returnValue, start);
    }

    return randomWords;
  }

  /**
   * delegatecall 合约
   * @param data 合约input data
   * @param addr 合约地址
   */
    function assemblyCall(bytes memory data, address addr) internal returns (bytes memory) {
        uint256 len = data.length;
        uint retsize;
        bytes memory resval;
        assembly {
            let result := delegatecall(gas(), addr, add(data, 0x20), len, 0, 0)
            retsize := returndatasize()
        }
        resval = new bytes(retsize);
        assembly {
            returndatacopy(add(resval, 0x20), 0, returndatasize())
        }
        return resval;
    }

    function sliceUint(bytes memory bs, uint start) internal pure returns (uint256) {
        require(bs.length >= start + 32, "slicing out of range");
        uint256 x;
        assembly {
            x := mload(add(bs, add(0x20, start)))
        }
        return x;
    }
}
```

注意：

1. 用户请求单个随机数

内置合约通过计算随机源VRF与当前用户交易hash的异或直接返回结果

```
for i := 0; i < txHash.Length; i++ {
	randomWords[i] = currentNonces[i] ^ txhash[i]
}
```

2. 用户请求随机数个数为 n（ n > 1）

内置合约 `VrfInnerContract` 通过计算随机源VRF与当前用户交易hash的异或，再与当前区块的前n个区块的VRF分别异或，顺序为从parent区块到`currentBlockNumber-n+1`块，然后将所有计算结果依次拷贝到返回结果的数组中

```

for i := 0; i < txHash.Length; i++ {
	randomWords[i] = currentNonces[i] ^ txhash[i]
}

vrf := handler.GetVrfHandlerInstance()
nonceInVrf, err := vrf.Load(v.Evm.ParentHash)

for i := 1; i < int(randomWordsNum); i++ {
	// 优先从VrfHandler中获取nonce, 当获取不到则从区块中拿
	if i+1 > len(nonceInVrf) {
		preNonce = vrf2.ProofToHash(v.Evm.GetNonce(currentBlockNum - uint64(i) - 1))
	} else {
		preNonce = nonceInVrf[len(nonceInVrf)-i-1]
	}
	start := i * common.HashLength
	for j := 0; j < common.HashLength; j++ {
		randomNumbers[j+start] = randomNumbers[j] ^ preNonce[j]
	}
}

```

## 注意事项

1. 随机数是“同步”被返回给用户的

这意味着用户在智能合约中请求后可立即使用随机数，无需异步处理。

2. 用户不需要预付费

在一些其他公链相近的方案中，随机数是通过“链下生成+链上验证”的方式提供给用户使用，在PlatON方案中Consumer请求随机数是同步返回的， 因此不需要用户预付费。

## 安全性分析

1. 区块提议人可以提前知道随机数

因随机数必须经由区块提议人私钥签名获得， 这样提议人在提议区块时可以提前获取随机数，因PlatON的VRF随机数被应用于共识节点的随机选取，安全性已经得到充分验证，提议人提前获取随机数并不能做任何恶意行为， 因此安全性是可以得到保障的。

2. 每个区块对应一个随机源

因随机源来源于区块的Nonce，因此在同一个区块内的所有随机数均有同一个随机源产生，因随机源是安全的，故所有衍生的随机数亦是安全的。
