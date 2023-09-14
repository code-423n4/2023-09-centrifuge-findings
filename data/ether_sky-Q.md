1. 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L569-L582

I guess you can change the 'public' visibility to 'internal'.
Otherwise, consider the case ```trancheTokenAmount = 0``` in ```_calculatePrice()``` function.

2. 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L238-L243

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L192-L212

You checked that the decimals is less than or equal to 18 in ```addCurrency()``` function, but not in ```addTranche()``` function.

3.
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L598

The name ```currencyAmountInPriceDecimals``` is confusing.
Can change to ```trancheTokenAmountInPriceDecimals```.