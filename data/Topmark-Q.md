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
### Report 3:
Repetition of code condition validation makes codes less readable and unnecessarily complicated in [L893](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L893) of the Messages.sol contract. As seen in the correction in the code below since _bytes32[i] has been checked for zero before a better way would be to simply use "j" for the second loop in relation to updated value of "i"
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L893
```solidity
  function _bytes32ToString(bytes32 _bytes32) internal pure returns (string memory) {
        uint8 i = 0;
        while (i < 32 && _bytes32[i] != 0) {
            i++;
        }
        bytes memory bytesArray = new bytes(i);
    ---    for (i = 0; i < 32 && _bytes32[i] != 0; i++) {
    ---        bytesArray[i] = _bytes32[i];
    +++    for (uint8 j = 0; j < i ; j++) {
    +++        bytesArray[j] = _bytes32[j];
        }
        return string(bytesArray);
    }
```
### Report 4:
Incomplete comment description in [L233](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L233) of the Messages.sol contract. As seen in the correction in the code below it should be "(16 bytes = uint128)" not "(16 bytes)"
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L233
```solidity
  /**
     * Update a Tranche token's price
     *
     * 0: call type (uint8 = 1 byte)
     * 1-8: poolId (uint64 = 8 bytes)
---   * 9-24: trancheId (16 bytes)
+++   * 9-24: trancheId (16 bytes = uint128)
     * 25-40: currency (uint128 = 16 bytes)
     * 41-56: price (uint128 = 16 bytes)
     */
    function formatUpdateTrancheTokenPrice(uint64 poolId, bytes16 trancheId, uint128 currencyId, uint128 price)
        internal
        pure
        returns (bytes memory)
    {
...
```