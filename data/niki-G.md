## In `_decreaseDepositLimits` the _currency cannot be greater than maxDeposit because there is a check in processDeposit or processMint

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L623
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L628
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L431
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L455
## Impact
There is no way to go in this check, this means unwanted gas for deploying and it will be less gas if the check in the recommended is used

## Recommended Mitigation Steps
if (lpValues.maxDeposit <= _currency) {
            lpValues.maxDeposit = 0;
}
