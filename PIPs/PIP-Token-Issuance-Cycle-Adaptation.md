---
PIP:  100
Topic: Token issuance cycle adaptation|[简体中文](./PIP-Token-Issuance-Cycle-Adaptation-CN.md)
Author: jianghaitao
Status: Draft
Type: Requirement
Description: 
Created: 2019-11-28
---

# PIP-100：Token issuance cycle adaptation

## Abstract

The block interval is not accurate enough to evaluate the additional period, 
and the average block interval can be regarded as a key parameter of the additional period.

## Motivation

Calibration cycle of token issuance

## Detail

### Requirement

The timpstamp in blockheaders is the host time in which the block was mined, 
There are 2 parameters in the 'CbftConfig' configuration, 'Amount' and 'Period', 
'Amount' is the maximum number of blocks that a node can produce in a cycle, 
and 'Period' is the duration of the cycle, in milliseconds.

With these 2 parameters, does it mean that the average block generation interval 
is 'Period' / 'Amount'?
It's wrong to understand this way. 

Because each mining node needs to maximize the mining revenue, 
that is, they will try to fill up 'Amount' blocks as much as possible 
(so as to get all the block rewards), when the transaction pool is empty For the 
miner node to maximize the revenue within a limited period of time, the miner node 
will usually fill out 'Amount' blocks in advance, because once the subsequent 
blocks are produced when the time is about to expire, The probability of confirming 
this block can be confirmed will be lower.

If the token issuance is calculated based on the block interval 'Period' / 'Amount', 
the actual token issuance time may become unpredictable due to the large deviation 
between the actual interval and the calculation result. It may be issued at 6th months
first time, and 2 years later, so it becomes necessary to adjust the block height of 
the token issuance according to the average block interval on the chain.

This upgrade is mainly to dynamically adjust the token issuance cycle based on the 
average block generation interval, so that the actual token issuance cycle is in line 
with our design expectations. Correspondingly, because the number of settlement cycles 
in each additional issue cycle is uncertain, but the rewards for block production and 
staking are fixed, the block production and staking rewards in each settlement cycle 
need to be dynamic calculation base on the total remaining rewards in the current 
additional cycle.

### Implement

#### Theoretical value of additional time interval

Because there is no absolutely accurate time in the blockchain world, and the token issuance 
in the PlatON economic model is fixed in accordance with the natural year, a theoretical 
value of the time interval for the additional issuance is required. There is a leap year 
every 4 years in the natural year. so the average annual fixed time is about: 365 + 1/4 days, 
converted to seconds is 31557600 seconds.

#### Calculate the average block interval

We use the time stamp of the first block as the starting time of the first year, 
and calculate the average of the previous year (based on the number of settlement cycles 
experienced in the first year) in the last block of each settlement cycle The block interval 
is as follows:

N: number of settlement cycles in the first year

Hn: height of the last block in the current settlement cycle

Hl: Hn> N? Hn-N: 1 (the height of the last block of N settlement cycles from Hn forward, 
    if Hn is greater than N, Hl is Hn-N, otherwise Hl = 1)
	
Tn: timestamp of the last block of the current settlement cycle (ms)

Tl: timestamp corresponding to Hl

I: block interval

Calculate the average block interval:

I = Floor ((Tn-Tl) / (Hn-Hl))

#### Calculate the number of remaining settlement cycles for the current year

Set:

n: the number of additional issuances in the current settlement cycle (the first additional 
   issuance was in the genesis block)
   
Y: 31557600000 milliseconds (365 + 1/4 days)

T0: timestamp of the first block (milliseconds)

Tn: timestamp of the last block of the current settlement cycle (ms)

C: 10750 (number of blocks per settlement cycle)

L: number of settlement cycles remaining in the current issuance cycle

then:
L = Ceiling (((T0 + n * Y)-Tn) / I * C)

#### Calculate the block reward and pledge reward for the current settlement cycle

Set:

Ml: Total number of annual remaining tokens at the beginning of the current settlement cycle

V: Total number of validators corresponding to the block height at the end of the settlement cycle

L: The current issue cycle includes the number of settlement cycles remaining in the current settlement cycle

Rb: Fixed reward for block production

Rs: Pledged rewards received by each node

R: Reward ratio (percentage) of the block giver in the reward

then:

Rb = Ml * R / C * L

Rs = Ml * (1-R) / (L * V)


#### Foundation lockup release plan adjustment

The plan of the PlatON Foundation to subsidize miners will be released to the rewarding pool 
contract along with the token issuance based on the actual number of token issuances.

#### Foundation token distribution plan adjustment

PlatON allocates a specified percentage of additional tokens to the foundation after the 
additional n issuances. The actual initial allocation time is allocated according to the 
number of additional issuances and the token issuance.

### Deployment

Deploy nodes as normal

## References

[PlatON Blue Paper](https://www.platon.network/static-new/pdf/en/PlatON_Blue_Paper_on_Economics_EN.pdf)
