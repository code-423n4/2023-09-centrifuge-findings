### InvestmentManager.sol
1. No check case for if _currencyAmount and  _trancheTokenAmount is 0 in decreaseDepositRequest() and decreaseRedeemRequest() respectively
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L174-L185
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L187-L198

2. Silent overflow in _fromPriceDecimals if (PRICE_DECIMALS < decimals)
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L682
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L692

3. If `currencyPayout` is set to be 0 in handleExecutedCollectRedeem() then user will not be able to withdraw their funds becoz it set lpValues.maxWithdraw = 0. 
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L269
Use something like, require(currencyPayout != 0)

4. If `trancheTokensPayout` is set to be 0 in handleExecutedCollectInvest() then user will not be able to mint becoz it set lpValues.maxMint=0.
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L249C56-L249C56
Use something like, require(trancheTokensPayout != 0)

5. trigger approve() before transferFrom()
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L167
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L313
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L474

6. trigger approve() before safeTransferFrom()
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L136
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L291

### PoolManager.sol
1. Zero address check missing for `recipient` in handleTransfer()

2. trigger approve() before safeTransferFrom()
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L132

### RestrictionManager.sol
1. Duplicate address check is missing in updateMembers()
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L62

