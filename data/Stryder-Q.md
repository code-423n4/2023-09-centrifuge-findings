## [QA-01] Functions are reverting and still returning a bool 

## Summary 
Functions should either revert for the invalid transactions or they shoudld return the bool associated with the result indicating the transaction to be a success or fail. But some functions are reverting if there is a failure 

## Impact
Some of the sanity checks on the Chainlink Oracle are not present while getting the price of the tokens . 
This makes that the data that the feed returns as invalid , stale or erreneous in the Oracle 

## Total instances 
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L154
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L157

## Tools Used 

- Remix IDE 
- VS Code 
- Solidity Metrics 

## Recommend Mitigation

Recemmonded way is to make the function return bool and the revert can then happen in the calling function . Even if not , you can just revert in the called function instread of return a true everytime there is no revert. 