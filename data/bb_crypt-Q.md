
1) Several functions return a value that is never used and can be confusing for future use:
 - PoolManager:
 ```
function isAllowedAsPoolCurrency(uint64 poolId, address currencyAddress) public view returns (bool){
        uint128 currency = currencyAddressToId[currencyAddress];
        require(currency != 0, "PoolManager/unknown-currency"); // Currency index on the Centrifuge side should start at 1
        require(pools[poolId].allowedCurrencies[currencyAddress], "PoolManager/pool-currency-not-allowed");
        return true;
 }
```
 This function is used in `InvestmentManager` lines 124, 154 without checking the return type.

 If the PoolManager will be changed in the future, and the `isAllowedAsPoolCurrency` can return false, those calls in the `InvestmentManager` will be useless. Also we can observe that in the `PoolManager` the result from `isAllowedAsPoolCurrency` is checked (line: 310 -> `require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool`

2) Some functions can be declared as VIEW or PURE: InvestmentManager.sol:
Functions `_toPriceDecimals` and `_fromPriceDecimals` can be declared as PURE
Function `_isAllowedToInvest` can be declared as VIEW

3) Input verification 
LiquidityPool constructor don't check for address(0) on `asset_` param. This can represent a issue if a prior check is not made in future use