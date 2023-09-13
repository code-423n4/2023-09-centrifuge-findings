# redundant check at PoolManager.sol `deployLiquidityPool`.
If the pool doesn't exist, the function will revert at its beginning. Here's the relevant code:
```
    function deployLiquidityPool(uint64 poolId, bytes16 trancheId, address currency) public returns (address) {
        // @info check if the tranch exist
        Tranche storage tranche = pools[poolId].tranches[trancheId];
        require(tranche.token != address(0), "PoolManager/tranche-does-not-exist"); // Tranche must have been added

        // @info check is currency is added
        require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool

        // @info check if liquidityPool is deployed, so this function can't be called twice with same parameters
        address liquidityPool = tranche.liquidityPools[currency];
        require(liquidityPool == address(0), "PoolManager/liquidityPool-already-deployed");
        // @audit redundant check, this line will never be executed
        require(pools[poolId].createdAt != 0, "PoolManager/pool-does-not-exist");
```