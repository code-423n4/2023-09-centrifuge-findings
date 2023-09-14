# QA-01: Missing check in the constructor of `Root.sol` if the initially set delay is smaller than the `MAX_DELAY`

Link to code:
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34-L40

## Description:
There is no check in the constructor of `Root.sol` that makes sure that the initially set delay is smaller than the `MAX_DELAY`. This can lead to delays in the launch of the project in case the dalay is set to more than the `MAX_DELAY` since only wards of root can change this value and it would take more than `MAX_DELAY` to set a new ward for the root contract. Consider adding a check that ensures that the initially set delay is smaller than `MAX_DELAY` 
# QA-02:  redundant check at PoolManager.sol `deployLiquidityPool`.
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

# QA-03:  The functions _stringToBytes32 and _stringToBytes128 are missing length checks. 
Unexpected results may occur if the string length is greater than or equal to 32 or 128, respectively.

## POC
```Solidity
    function testMsgForwarding() public {
        string memory test = "thisisatet";
        bytes32 result1 = _stringToBytes32(test);
        emit log_bytes32(result1);
        string memory string1 = _bytes32ToString(result1);
        console.log("test: %s string1 : %s ", string1);

        string memory stringWithLeng40 = "1234567890123456789012345678901234567890";

        bytes32 result2 = _stringToBytes32(stringWithLeng40);
        string memory string2 = _bytes32ToString(result2);
        console.log("stringWithLeng40 : %s result : %s", stringWithLeng40, string2);

        emit log_bytes32(result2);
    }
```