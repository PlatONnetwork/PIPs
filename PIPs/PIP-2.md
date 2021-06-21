---
PIP: 2
Topic: Compatible with Ethereum
Author: clearly
Status: Draft 
Type: Text
Description: The underlying technology shall be fully compatible with Ethereum's ecosystem
Created: 2021-05-28
---

# PIP-2：The PIP of fully Compatible with Ethereum

## Abstract
In order to attract more and more developers to the PlatON network and ecosystem,  and reduce the development cost of moving dApps from Ethereum to PlatON, we should be Ethereum compatible in aspects such as SDK, EVM, RPC interface, solidity language, etc.

## Motivation

As a chunk of the world's second largest market value blockchain network, Ethereum is also the biggest platform for the smart contracts, with many developers and mature community. Many developers evolving to blockchain practitioners are through the Ethereum smart contracts, who are skilled with the smart contracts, the tools for the developement of DApps, SDKs of Ethereum, but not familiar with PlatON. So if we can make it compatible with Ethereum, it will definitely make PlatON more attractive to developers, and it is conducive to the development of ecosystems.

The key points PlatON and Ethereum are currently incompatible:


- The address format is different. PlatON uses the address format of Bech32, while Ethereum uses the address format of Eip55
- The Token units are different. PlatON's Token units are LAT/VON and Ethereum's are Ether/WEI
- Some functions are not implemented in Platon or are implemented differently. Functions such as block.difficlty, miner. SetEtherbase, etc., the Ethereum contracts using these functions may not work on PlatON and may need to be adjusted accordingly, but these functions are rarely used.
- The block header timestamp is different, Ethereum in seconds and PlatON in milliseconds

From these differences, Platon can be made compatible with Ethereum with minimal changes.

## Implementation notes

### Compatible with RPC interface

#### 1.Compatible interface format
PlatON's current address encoding format is Bech32, while Ethereum's current address encoding format is Eip55, so we need to make compatibility in address format.
Parts of the PlatON RPC interface have a namespace of 'PlatON', such as' PlatON_getBalance '. Some of the RPC interfaces on Ethereum have a namespace of 'eth', such as' eth_getBalance '. Both functions have the same functionality, so we need namespace-compatibility.

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
This upgrade will be compatible with historical data and requires an on-chain governance upgrade. See the discussion[Link](https://forum.latticex.foundation/t/topic/4636)

[English translation]
