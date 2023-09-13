# Amaze Portfolio Audit Report

###### tags: `Farm`

## 1. INTRODUCTION

### 1.1 Disclaimer
The audit makes no statements or warranties about utility of the code, safety of the code, suitability of the business model,
investment advice, endorsement of the platform or its products, regulatory regime for the business model,
or any other statements about fitness of the contracts to purpose, or their bug free status. The audit documentation is 
for discussion purposes only. The information presented in this report is confidential and privileged. 
If you are reading this report, you agree to keep it confidential, not to copy, disclose or disseminate without the agreement of the Client.
If you are not the intended recipient(s) of this document, please note that any disclosure, copying or dissemination of its content is strictly forbidden.

### 1.2 Security Assessment Methodology

A group of auditors are involved in the work on the audit. The security engineers check the provided source code independently of each other in accordance with the methodology described below:

#### 1. Project architecture review:

* Project documentation review.
* General code review.
* Reverse research and study of the project architecture on the source code alone.


##### Stage goals
* Build an independent view of the project's architecture.
* Identifying logical flaws.


#### 2. Checking the code in accordance with the vulnerabilities checklist:

* Manual code check for vulnerabilities listed on the Contractor's internal checklist. The Contractor's checklist is constantly updated based on the analysis of hacks, research, and audit of the clients' codes.
* Code check with the use of static analyzers (i.e Slither, Mythril, etc).


##### Stage goal 
Eliminate typical vulnerabilities (e.g. reentrancy, gas limit, flash loan attacks etc.).

#### 3. Checking the code for compliance with the desired security model:

* Detailed study of the project documentation.
* Examination of contracts tests.
* Examination of comments in code.
* Comparison of the desired model obtained during the study with the reversed view obtained during the blind audit.
* Exploits PoC development with the use of such programs as Brownie and Hardhat.

##### Stage goal
Detect inconsistencies with the desired model.


#### 4. Consolidation of the auditors' interim reports into one:
* Cross check: each auditor reviews the reports of the others.
* Discussion of the issues found by the auditors.
* Issuance of an interim audit report.

##### Stage goals
* Double-check all the found issues to make sure they are relevant and the determined threat level is correct.
* Provide the Client with an interim report.


#### 5. Bug fixing & re-audit:
* The Client either fixes the issues or provides comments on the issues found by the auditors. Feedback from the Customer must be received on every issue/bug so that the Contractor can assign them a status (either "fixed" or "acknowledged").
* Upon completion of the bug fixing, the auditors double-check each fix and assign it a specific status, providing a proof link to the fix.
* A re-audited report is issued. 


##### Stage goals
* Verify the fixed code version with all the recommendations and its statuses.
* Provide the Client with a re-audited report.

#### 6. Final code verification and issuance of a public audit report: 
* The Customer deploys the re-audited source code on the mainnet.
* The Contractor verifies the deployed code with the re-audited version and checks them for compliance.
* If the versions of the code match, the Contractor issues a public audit report. 

##### Stage goals
* Conduct the final check of the code deployed on the mainnet.
* Provide the Customer with a public audit report.



#### Finding Severity breakdown

All vulnerabilities discovered during the audit are classified based on their potential severity and have the following classification:

Severity | Description
--- | ---
Critical | Bugs leading to assets theft, fund access locking, or any other loss of funds.
High     | Bugs that can trigger a contract failure. Further recovery is possible only by manual modification of the contract state or replacement.
Medium   | Bugs that can break the intended contract logic or expose it to DoS attacks, but do not cause direct loss funds.
Low | Bugs that do not have a significant immediate impact and could be easily fixed.



Based on the feedback received from the Customer regarding the list of findings discovered by the Contractor, they are assigned the following statuses:


Status | Description
--- | ---
Fixed        | Recommended fixes have been made to the project code and no longer affect its security.
Acknowledged | The Customer is aware of the finding. Recommendations for the finding are planned to be resolved in the future.


### 1.3 Project Overview

Amaze Portfolio is a project to manage your Asset Portfolio. Users can create a portfolio and add assets to it. Then he can withdraw his assets from it or remove the portfolio completely. Free 1 account can create 1 portfolio. You can buy premium access for a certain number of tokens, which is set by the administrator. This will allow you to create an unlimited number of portfolios. When you withdraw your assets, a fee is charged on the difference of the price of addition and withdrawal. The fee is set by the administrator.

Portfolio is SBT (soulbond token) - NFT which cannot be transferred.

### 1.4 Project Dashboard

#### Project Summary

Title | Description
--- | ---
Client             | Amaze Portfolio
Project name       | Amaze Portfolio
Timeline           | 30.05.2023 - 03.06.2023

#### Project Log

Date | Commit Hash | Note
--- | --- | ---
30.05.2023 | 2a3fdac5fad2f0c271980ec660157bfbde647097 | initial commit for the audit

#### Project Scope
The audit covered the following files:

File name | Link
--- | ---
libraries/SpecialAddress.sol | https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/libraries/SpecialAddress.sol
portfolio/NFT.sol | https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol
portfolio/Portfolio.sol | https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol
price-feeds/PriceFeedCalculator.sol | https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol
price-feeds/UniswapPriceFeed.sol | https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/UniswapPriceFeed.sol
price-feeds/ChainlinkPriceFeed.sol | https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/ChainlinkPriceFeed.sol

### 1.5 Summary of findings

Severity | # of Findings
--- | ---
CRITICAL| 12
HIGH    | 1
MEDIUM  | 4
LOW | 13

### 1.6 Conclusion
At moment the contract owners have a lot of control over the users and their assets. It is recommended to use the governance pattern. There is also a reentrancy vulnerability when removing portfolio that allows the entire contract balance to be taken out. Insecure use of third-party erc20 contracts, which can cause the entire transaction to fail. Loop optimization is recommended.

---

## 2. FINDINGS REPORT

### 2.1 Critical

#### 1. All user assets can be withdrawn by the owner of the contract

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L421
`SpecialAddresses` can withdraw all user tokens from the contract. The `getTokenBalance` function returns the entire token balance of the contract, not the margin balance.

##### Recommendation
Check `require(_amounts[i] <= getBalanceFee(_tokenAddresses[i]))` and `require(_amountNative <= getBalanceFee(native))` before `transfer`. Should also change the `balanceFee` state before `transfer` to avoid reentrancy.

---

#### 2. The owner has full access to change the oracle address of uniswap

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L162
`SpecialAddresses` can change the uniswap oracle address to any contract that may give wrong prices. Manipulation of asset prices. This can cause failure when adding or removing assets or high margins due to high asset prices. Users will not be able to withdraw their assets.

- withdrawal lock – set the attack contract with the function `estimateAmountOut` which ends with an error.
- margin increase – set the attack contract with the function `estimateAmountOut` which returns too large cost of token before removing liquidity.

##### Recommendation
Remove this function or add government functionality to manage this function.

---

#### 3. The owner has full access to change the oracle address of chainlink

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L150
`SpecialAddresses` can change the chainlink oracle address to any contract that may give wrong prices. Manipulation of asset prices. This can cause failure when adding or removing assets or high margins due to high asset prices. Users will not be able to withdraw their assets.

- withdrawal lock – set the attack contract with the function `getLatestPrice` which ends with an error.
- margin increase – set the attack contract with the function `getLatestPrice` which returns too large cost of token before removing liquidity.

##### Recommendation
Remove this function or add government functionality to manage this function.

---

#### 4. The owner has full access to change the pool address of uniswap oracle

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L134
`SpecialAddresses` can set wrong fee on token of pool. When calculating the price the pool will not be found and the transaction will end with an error. Users will not be able to withdraw their assets and add new ones.

- withdrawal lock – set the attack contract with the function `estimateAmountOut` which ends with an error.

##### Recommendation
Add government functionality to manage this function or set the default fee for uniswap pools.

---

#### 5. The owner has full access to change the proxy address of chainlink oracle

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L115
`SpecialAddresses` can set a wrong proxy address for the token. When calculating the price the function `latestRoundData()` on this address will not be found and the transaction will end with an error. Users will not be able to withdraw their assets and add new ones.
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/ChainlinkPriceFeed.sol#L32

- withdrawal lock – set the attack contract with the function `getLatestPrice` which ends with an error.

##### Recommendation
Add government functionality to manage this function.

---

#### 6. The owner has full access to change the second token of the uniswap pool

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L167
`SpecialAddresses` can change the second token to anything (not necessarily stabelcoin).
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L70
The function `getTokensPrice()` will be return other price. Manipulation of the token price leads to manipulation of the margin. `SpecialAddresses` can increase the margin very much. Users will not be able to withdraw their assets.

##### Recommendation
Add government functionality to manage this function.

---

#### 7. The owner can increase the margin no limits

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L173
`SpecialAddresses` can set a very big value of the margin. Users will not be able to withdraw their assets.

##### Recommendation
Set a limit on the margin value of not more than 15%.

---

#### 8. Switching oracles can block the withdrawal of assets for users

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L155
1. User added liquidity with price calculation through one oracle (e.g. uniswap).
1. `SpecialAddresses` switched orale (e.g. chainlink).
1. User can not withdraw his assets because a price calculation through other oracle don't work (pool don't found).

##### Recommendation
When switching check that all tokens of one oracle are set in the another one. Or use two oracles at the same time (see point 1 on low).

---

#### 9. When removing a portfolio, the user can withdraw the entire balance of the contract

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L380 transfer Ether
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#LL391C52-L391C52 transfer ERC20

Reentrancy attack for get ethers.
1. The attacking contract contributes the ethers (value for step reentrancy)
1. It calls the function `removePortfolio`
1. It gets Ether
1. It re-calls the function `removePortfolio` with same value because the state has not yet been updated

Although the `transfer` function only consumes 2300 gas. It's still not safe.
ERC20 has the same vulnerability. But `transfer` function of ERC20 consumes remaining gas.

Reentrancy attack for get erc20.
1. The attacking contract contributes some erc20 tokens (value for step reentrancy)
1. It contributes the attack erc20 contract with the function `transfer` that it re-calls the function `removePortfolio`
1. The attacking contract calls the function `removePortfolio`
1. It gets its tokens.
1. The attack erc20 contract re-calls the function `removePortfolio` with same value of tokens because the state has not yet been updated

##### Recommendation
Before `transfer` change the balances of the user's portfolio tokens

---

#### 10. When changing the address of the native token, users with native tokens will be not able to withdraw tokens and ethers

##### Status
New

##### Description
- the function changing the address of the native token https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L451
- checking the address of the native token https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L280
- checking the address of the native token https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L379

These checks will return false if the address of the native token is changed. The `erc20 transfer` at this address will be called. The transaction will fail because there is no such function. Also, the user will not be able to withdraw ethers because the old native token no longer provides them.

##### Recommendation
Remove the function `setNativeToken`. The address of native token will be constant (e.g. 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE)

---

#### 11. Blocking when remove a portfolio because of a bad token

##### Status
New

##### Description
- user removes a portfolio - https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L391

External functions may end with an error. If one of the tokens fails, it will cancel the entire transaction. You cannot select tokens when removing a portfolio.

##### Recommendation
Use `contract.call()`, which will not cancel the whole transaction on one failure. You can use openzeppelin's implementation of [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ffceb3cd988874369806139ae9e59d2e2a93daec/contracts/token/ERC20/utils/SafeERC20.sol).

---

#### 12. User assets could be lost if the portfolio link to the nft is changed

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L445
All user assets will be lost when change address of nft because data about assets will be empty in the new contract.

https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L215
`SpecialAddress` can set a new address EOA or a contract. Then manipulate it by calling functions with portfolio address checking.

##### Recommendation
Remove this functions. Setting addresses of portfolio and nft in the deploy process.

### 2.2 High

#### 1. Anyone can change the address of the uniswap factory and break adding and removing liquidity

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/UniswapPriceFeed.sol#L72

* Call `setFactory` with your address contract without the function `getPool()` and `fallback()` or with the function `getPool()` that return 0 address.

##### Recommendation
Remove this function or add government functionality to manage this function.

### 2.3 Medium

#### 1. All owners can be deleted

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/libraries/SpecialAddress.sol#L18
One owner can remove all owners and self too. All administrative functions will become unavailable.

##### Recommendation
Add a count of the number of owners. When deleting, check that the number of owners is greater than 0.

---

#### 2. Loss of assets if duplicate tokens are transferred when adding liquidity

##### Status
New

##### Description
1. Call `createPortfolio` or `Portfolio.addLiquidity` with duplicate tokens.
1. Each token in turn will be passed to the function `NFT.setPriceData`.
1. As a result, the `balances[_idNFT][_token][_idTimeStamp]` will store data about the last duplicate token.

##### Recommendation
- 1 option: Check for duplicates before the loop. Merge tokens. 
- 2 option: Сheck `balances[_idNFT][_token][_idTimeStamp].amount` and fail the transaction. It's better optimized.

---

#### 3. Gas overflow during iteration (DoS)

##### Status
New

##### Description
Each iteration of the cycle requires a gas flow. A moment may come when more gas is required than it is allocated to record one block. In this case, all iterations of the loop will fail. Affected lines:

- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L247
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L354

These functions use loops that loop through dynamic arrays from the storage. The loops are not limited. If the storage gets too large, these functions may stop working.

##### Recommendation
Optimize the operation of loops. Make it possible to partially remove liquidity.

---

#### 4. Blocking when withdraw a liquidity because of a bad token

##### Status
New

##### Description
- user removes a liquidity - https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L287
- owner withdraws tokens - https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L432
- owner withdraws tokens - lines 428, 430 calls the function `getTokenBalance` that calls the function external `balanceOf`

External functions may end with an error. If one of the tokens fails, it will cancel the entire transaction. A bad token does not have to be selected in these functions.

##### Recommendation
Use `contract.call()`, which will not cancel the whole transaction on one failure. You can use openzeppelin's implementation of [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ffceb3cd988874369806139ae9e59d2e2a93daec/contracts/token/ERC20/utils/SafeERC20.sol).

### 2.4 Low

#### 1. Using one oracle

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L67

##### Recommendation
Using two oracles simultaneously is more decentralized and reliable. You can get token prices from multiple sources and aggregate them into one result.

---

#### 2. No check of contract balance before transfer

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L424

##### Recommendation
Make a check that the balance on the contract is enough to transfer.

---

#### 3. Blocking when adding liquidity because of a bad token

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L184
If one of the tokens fails, it will cancel the entire transaction.

##### Recommendation
Use `contract.call()`, which will not cancel the whole transaction on one failure. You can use openzeppelin's implementation of [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ffceb3cd988874369806139ae9e59d2e2a93daec/contracts/token/ERC20/utils/SafeERC20.sol).

---

#### 4. NFT supplyCounter already exists in ERC721Enumerable

##### Status
New

##### Description
double state variable
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L24
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L52
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L209

ERC721Enumerable
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ffceb3cd988874369806139ae9e59d2e2a93daec/contracts/token/ERC721/extensions/ERC721Enumerable.sol#L45

##### Recommendation
Use `ERC721Enumerable.totalSupply()`. 5 point is important for this point.

---

#### 5. Portfolio data is not scrubbed

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L407
NFT have been burn but portfolio data did not.

##### Recommendation
Clear state of this NFT - balances, info, addresses.

---

#### 6. A lot of transfers one token when remove portfolio

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L391
The same tokens can be in different timestamps. But transfer is called in every timestamp.

##### Recommendation
Collect the total number of tokens and call the transfer once (for one erc20 contract is one transfer).

---

#### 7. Compilation error – Wrong argument count for function call: 3 arguments given but expected 4

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L239

##### Recommendation
Add argument `batchSize` – https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ffceb3cd988874369806139ae9e59d2e2a93daec/contracts/token/ERC721/extensions/ERC721Enumerable.sol#L60

---

#### 8. The `data.size` variable is redundant

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L18
`data.size` is `data.prices.length` 

##### Recommendation
Remove `data.size` and use `data.prices.length`

---

#### 9. The modifier `payable` for some functions is redundant

##### Status
New

##### Description
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L251
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L356
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L418

These functions not receive ethers.

##### Recommendation
Remove the modifier `payable` for these functions.

---

#### 10. Poorly readable variable `_memory`

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L255
It is unclear what the indexes of this array mean. `_memory[3]` is always – excessive variable.

##### Recommendation
Make separate variables.

---

#### 11. Excessive getters

##### Status
New

##### Description
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L186
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/PriceFeedCalculator.sol#L193
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/UniswapPriceFeed.sol#L78
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L209
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L203
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L177
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L154
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L131
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L66
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L497
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L481
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L475
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L469
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/Portfolio.sol#L463

For public variables, the getters are the default.

##### Recommendation
Remove these getters. Make a modifier `public` for these variables.

---

#### 12. The `UniswapPriceFeed` interface implementation is missing for `UniswapPriceFeed`

##### Status
New

##### Description
https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/price-feeds/UniswapPriceFeed.sol#L8

##### Recommendation
Add this implementation.

---

#### 13. Overriding functions do not override anything

##### Status
New

##### Description
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L239
- https://github.com/amaze-finance/Amaze-Portfolio/blob/2a3fdac5fad2f0c271980ec660157bfbde647097/contracts/portfolio/NFT.sol#L247

##### Recommendation
Remove these functions.
