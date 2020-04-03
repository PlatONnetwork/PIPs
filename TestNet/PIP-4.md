---
PIP:  4
Topic: 提议支持WASM合约并修复测试网部分bug
Author: alliswell
Status: DRAFT 
Type: Requirement
Description: 关于支持WASM合约和修复部分bug的提议
Created: 2020-03-30
---

# PIP-4：支持WASM合约并修改测试网部分bug

## 背景

WebAssembly（简称WASM）是一种为栈式虚拟机设计的二进制指令集。WASM具有运行高效、内存安全、无未定义行为和平台独立等特点，经过了编译器和标准化团队多年耕耘，目前已经有了成熟的社区。在区块链领域，越来越多开发者倾向于基于WASM做DAPP开发。为让更多开发者能参与到PlatON建设中来，提议PlatON支持WASM合约。

## 目的

为了让PlatON能吸引更多的开发者，丰富PlatON社区，提高社区活跃度以及降低开发者门槛。

## 内容

### 支持WASM合约

PlatON需要支持以下功能：

- 编译，部署，调用WASM合约
- WASM合约的升级，销毁
- 提供Java、Javascript sdk调用wasm合约
- 发布wasm合约的部署工具

### 修复测试网以下问题

当前测试网有以下问题，提议尽快修复：

-  fast同步中途退出后节点启动失败问题

   **问题**：[一旦fast同步过程中断，节点重新启动将失败](https://github.com/PlatONnetwork/PlatON-Go/pull/1288)
   
   **解决方案**:
   在节点首次进行fast同步时，如果同步途中因某些原因进程退出，造成底层存储的数据不一致，节点再次启动时会失败（数据不一致导致无法继续运行）
   本次对fast同步机制做了修改，如果首次fast同步不成功，后面再次启动节点时需要重新进行fast同步。
   
-  [频繁调用GetTransactionCount接口会导致节点内存溢出问题](https://github.com/PlatONnetwork/PlatON-Go/pull/1256)

   **问题**：在链停止出块的前提下，频繁调用GetTransactionCount接口会导致节点内存不断增长，节点最终会因内存溢出而终止
   
   **解决方案**:
   此问题时RPC接口查询链上数据时父区块和子区块存在交叉引用，导致每次申请的内存没有办法回收，修复方案为对RPC接口做特殊处理，查询执行完直接解除引用
   
-  [不能向内置合约转账的问题](https://github.com/PlatONnetwork/PlatON-Go/issues/1187)

   **问题**：在测试网测试发现，不能直接向内置合约中转账
   
   **解决方案**:
   由于之前的版本对内置合约地址做了特殊处理，判断了data字段不能为空，所以直接转账会失败，本次通过升级提案对这个错误做了修复，可以直接转账到内置合约
   
-  [节点view差距很大时view同步慢的问题](https://github.com/PlatONnetwork/PlatON-Go/pull/1262)

   **问题**：3月9号链停止出块，11号升级恢复时发现节点间view差距较大，view同步太慢导致较长时间后链才恢复出块
   
   **解决方案**:
   判断如果当前节点和邻居节点view差距超过2时不再等超时，而是直接同步下一个view
   
-  [测试网节点同步时出现vrf invalidate问题](https://github.com/PlatONnetwork/PlatON-Go/issues/1134)

   **问题**：节点在同步偶尔会出现vrf invalidate问题，会导致节点同步停止，块高不会继续增长
   
   **解决方案**:
   此问题是由于CBFT引擎在出块时的时间过长导致，当前节点会根据时间超时启动再次出块，但由于上一个块还没出完，导致该节点双出，最终导致vrf校验失败，
   本次修改出块机制，出过一次当前高度的块将不会再次出同一高度的区块。
   
-  [BAD BLOCK问题](https://github.com/PlatONnetwork/PlatON-Go/issues/1198)

   **问题**：在存在EVM合约调用的交易中，个别节点执行区块出现BAD BLOCK
   
   **解决方案**:
   此问题由于PlatON存储优化设计中相同value被多个key引用导致，当前版本的修复方案是对value的引用做计数，当计数为0时才会删除。
