---
layout: post
title: The Largest Hack Incident in DeFi's History: Polynetwork Exploit
category: programming
---

### The Incident
On August 10th, 2021, Poly Network was attacked by anonymous white hat hacker or hackers, causing over $610 million in digital crypto assets at the price of that date to be transferred to hacker-controlled addresses. Eventually, all assets were returned to Poly Network over the next 15 days. This was the largest security incident in DeFi's history in terms of the value of stolen assets at the price of that date.

Stolen Assets:


| Name | NETWORK | AMOUNT |   
| --- | --- | --- |
| Ethereum | USDC | 96m$ |
| Ethereum | WBTC | 1,032 (50m$) |
| Ethereum | DAI  | 0.67m$ |
| Ethereum | USDT | 33m$   |
| Ethereum | ETH  | 28,966 （111m$）|
| Binance  | USDC | 87m$ |
| Binance  | BUSD | 32m$ |
| Polygon  | USDC | 85m$ |


...
### News: 
![image](https://user-images.githubusercontent.com/4775215/146314501-30ab8068-a0a0-480a-a170-5c7e99fa2cd7.png)
![image](https://user-images.githubusercontent.com/4775215/146314315-3cb83474-eb87-406d-a249-b6a28ffe05b3.png)
![image](https://user-images.githubusercontent.com/4775215/146314388-55b73706-5f0c-4883-a2e5-d59e468a9d64.png)
![image](https://user-images.githubusercontent.com/4775215/146314476-2e5c1c82-4b3e-4cda-b351-608ef48a998e.png)


[Wiki](https://en.wikipedia.org/wiki/Poly_Network_exploit)

### What is Cross Chain and Polynetwork
#### Why to Cross Chain
1. Cross Chain Farm: Transfer your USDT from ETH to BSC to farm; O3Swap;
2. Hidden your money: Address replaced; from BSC, Polygon to ETH to prevent controlled fork;
#### How Polynetwork Works
The Big Graph
![image](https://user-images.githubusercontent.com/4775215/146317676-621a25e6-c8b6-498a-8ef5-994d3889cd2e.png)

The Details
![image](https://user-images.githubusercontent.com/4775215/146317735-3d6cb848-018b-4fc3-9474-c74fd3fb55b9.png)


### How to Exploit
#### Just Miss One Line Code
![image](https://user-images.githubusercontent.com/4775215/146193244-6b1ff1b9-b5f6-4a88-b638-6ceda8ee5405.png)
[Source code](https://github.com/polynetwork/eth-contracts/blob/master/contracts/core/cross_chain_manager/logic/EthCrossChainManager.sol)
And this will then Call:
![image](https://user-images.githubusercontent.com/4775215/146320580-19c0e4dd-4aba-43e3-8fc3-8747becef5a0.png)
[Source code](https://github.com/christianxiao/eth-contracts/blob/master/contracts/core/cross_chain_manager/data/EthCrossChainData.sol)
**If you can put your own public keys, then you can controll everything.**



