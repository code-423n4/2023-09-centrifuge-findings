### Report 1:
The variable name used in L598 is wrong and misrepresents the intention of the calculation. It makes the code less readable. From my correction in the code below it can be seen that "currencyAmountInPriceDecimals" was previously used to represent the calculation of TrancheTokenAmount in PriceDecimals, to explain in simple terms for example, if we want to calculate x using y * y, the formula should x = y * y not y = y * y
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L598
```solidity
 function _calculateTrancheTokenAmount(uint128 currencyAmount, address liquidityPool, uint256 price)
        internal
        view
        returns (uint128 trancheTokenAmount)
    {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);

---    uint256 currencyAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool).mulDiv(
+++    uint256 trancheTokenAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool).mulDiv(
            10 ** PRICE_DECIMALS, price, MathLib.Rounding.Down
        );

 ---       trancheTokenAmount = _fromPriceDecimals(currencyAmountInPriceDecimals, trancheTokenDecimals, liquidityPool);
 +++       trancheTokenAmount = _fromPriceDecimals(trancheTokenAmountInPriceDecimals, trancheTokenDecimals, liquidityPool);
    }
```
### Report 2:
Comment is placed wrongly in [L10](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L10) of the Messages.sol contract 
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L10
```solidity
  enum Call
 ---    /// 0 - An invalid message
    {
 +++    /// 0 - An invalid message
        Invalid,
        /// 1 - Add a currency id -> EVM address mapping
        AddCurrency,
   ...
```