## [Low] precision loss in `InvestmentManager`

### [_calculatePrice](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L569)

If `currencyAmountInPriceDecimals` * 10 ** `PRICE_DECIMALS` is less than `trancheTokenAmountInPriceDecimals`, then this function will return 0 as the price.
For example, `currencyAmountInPriceDecimals` is 10, `trancheTokenAmountInPriceDecimals` is 100e18, the result is zero.

```
    function _calculatePrice(uint128 currencyAmount, uint128 trancheTokenAmount, address liquidityPool)
        public
        view
        returns (uint256 depositPrice)
    {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);
        uint256 currencyAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool);
        uint256 trancheTokenAmountInPriceDecimals =
            _toPriceDecimals(trancheTokenAmount, trancheTokenDecimals, liquidityPool);

        depositPrice = currencyAmountInPriceDecimals.mulDiv(
            10 ** PRICE_DECIMALS, trancheTokenAmountInPriceDecimals, MathLib.Rounding.Down
        );
    }
```


### [convertToShares](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L325)

If `assets` * 10 ** (PRICE_DECIMALS + trancheTokenDecimals - currencyDecimals) is less than `latestPrice`, this function will return zero.

For example, `asset` is 10, `(PRICE_DECIMALS + trancheTokenDecimals - currencyDecimals)` is 18, `lastestPrice` is 100e18, then the calculated shares is zero due to precision loss. 

```
    function convertToShares(uint256 _assets, address liquidityPool) public view auth returns (uint256 shares) {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);
        uint128 assets = _toUint128(_assets);

        shares = assets.mulDiv(
            10 ** (PRICE_DECIMALS + trancheTokenDecimals - currencyDecimals),
            LiquidityPoolLike(liquidityPool).latestPrice(),
            MathLib.Rounding.Down
        );
    }
```


### [convertToAssets](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L338)

This function is similar to `convertToShares`.

```
    function convertToAssets(uint256 _shares, address liquidityPool) public view auth returns (uint256 assets) {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);
        uint128 shares = _toUint128(_shares);

        assets = shares.mulDiv(
            LiquidityPoolLike(liquidityPool).latestPrice(),
            10 ** (PRICE_DECIMALS + trancheTokenDecimals - currencyDecimals),
            MathLib.Rounding.Down
        );
    }
```

### [_calculateTrancheTokenAmount](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L591)

If `_toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool)` * 1e18 is less than `price`, it will return zero due to precision loss.

```
    function _calculateTrancheTokenAmount(uint128 currencyAmount, address liquidityPool, uint256 price)
        internal
        view
        returns (uint128 trancheTokenAmount)
    {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);

        uint256 currencyAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool).mulDiv(
            10 ** PRICE_DECIMALS, price, MathLib.Rounding.Down
        );

        trancheTokenAmount = _fromPriceDecimals(currencyAmountInPriceDecimals, trancheTokenDecimals, liquidityPool);
    }
```


### [_calculateCurrencyAmount](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L605)

This is similar to `_calculateTrancheTokenAmount`

```
    function _calculateCurrencyAmount(uint128 trancheTokenAmount, address liquidityPool, uint256 price)
        internal
        view
        returns (uint128 currencyAmount)
    {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);

        uint256 currencyAmountInPriceDecimals = _toPriceDecimals(
            trancheTokenAmount, trancheTokenDecimals, liquidityPool
        ).mulDiv(price, 10 ** PRICE_DECIMALS, MathLib.Rounding.Down);

        currencyAmount = _fromPriceDecimals(currencyAmountInPriceDecimals, currencyDecimals, liquidityPool);
    }
```


## [Low] `handleTransfer` should check the approval after transfer
`handleTransfer` approves this and then transfers `currency` to the recipient. After transferring, it should check that the approval is zero or set it to zero.

```
    function handleTransfer(uint128 currency, address recipient, uint128 amount) public onlyGateway {
        address currencyAddress = currencyIdToAddress[currency];
        require(currencyAddress != address(0), "PoolManager/unknown-currency");

        EscrowLike(escrow).approve(currencyAddress, address(this), amount);
        SafeTransferLib.safeTransferFrom(currencyAddress, address(escrow), recipient, amount);
    }
```


## [Low] `PoolManager` can only add pools but not remove
In `PoolManager`, `gateway` could add pools, tranche and currency, but it seems that there is no way to remove any one of them, this might not be convenient to manage in certain situations.

