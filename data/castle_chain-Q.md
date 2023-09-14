## unnecessary checks can be removed , to enhance the code 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L310

the `require` keyword can be removed in the function `deployLiquidityPool ()` , because the function `isAllowedAsPoolCurrency` revert on failure in case of the currency is not supported  and always return true on success , so if the check fails the function will revert without executing the `require` key word 
```
    function isAllowedAsPoolCurrency(uint64 poolId, address currencyAddress) public view returns (bool) {
        uint128 currency = currencyAddressToId[currencyAddress];
        require(currency != 0, "PoolManager/unknown-currency"); // Currency index on the Centrifuge side should start at 1
        require(pools[poolId].allowedCurrencies[currencyAddress], "PoolManager/pool-currency-not-allowed");
        return true;
```
```solidity
 function deployLiquidityPool(uint64 poolId, bytes16 trancheId, address currency) public returns (address) {
        Tranche storage tranche = pools[poolId].tranches[trancheId];
        require(tranche.token != address(0), "PoolManager/tranche-does-not-exist"); // Tranche must have been added
@>        require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool

        address liquidityPool = tranche.liquidityPools[currency];
        require(liquidityPool == address(0), "PoolManager/liquidityPool-already-deployed");
        require(pools[poolId].createdAt != 0, "PoolManager/pool-does-not-exist");

```

## the `destination` address should be emitted in the userEscrow contract in the function `transferOut`
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L49

due to that the receiver can be different address from the user address , the `destination` address should be emitted 
```solidity 
    function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
        require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
        require(
            receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max),
            "UserEscrow/receiver-has-no-allowance"
        );
        destinations[token][destination] -= amount;

        SafeTransferLib.safeTransfer(token, receiver, amount);
        emit TransferOut(token, receiver, amount);
    }
``` 
## `getTranche` function should be created in the poolManager contract and use it to get the `tranche` form the storage by specifying the `poolId` and `trancheId` .

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L308

the function `getTranche()` will increase the simplicity and the modularity  of the code instead of get the tranche from the storage each time in different functions 
```solidity
function getTranche (uint64 poolId , bytes16 trancheId) private returns ( Tranche storage tranche ){
          tranche =  pools[poolId].tranches[trancheId];
```
this function can be used instead of the line of code in the poolManager 
```solidity 
        Tranche storage tranche = pools[poolId].tranches[trancheId];

```
## the check of the creation of the liquidity pool in the function `deployLiquidityPool()` in the PoolManager contract can be removed . 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L309

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L314

the `require` statement that checks that the pool is created can be removed since there is a check before it that make sure that the tranche token is deployed and the address of it is stored in the `pool` struct which be be stored only after the pool has been created , so the check of the deployment of the tranche token do the same check of the creation of the pool , so the second check can be removed 

the check in the function `addTranche()` , which make sure that the pool is created 
```solidity 
    function addTranche(
        uint64 poolId,
        bytes16 trancheId,
        string memory tokenName,
        string memory tokenSymbol,
        uint8 decimals
    ) public onlyGateway {
        Pool storage pool = pools[poolId];
@>      require(pool.createdAt != 0, "PoolManager/invalid-pool");
        Tranche storage tranche = pool.tranches[trancheId];
        require(tranche.createdAt == 0, "PoolManager/tranche-already-exists");
```
and the check of the creation of the pool in the function `deployLiquidityPool()` , which make sure that the tranche token is added and deployed 
```solidity
    function deployLiquidityPool(uint64 poolId, bytes16 trancheId, address currency) public returns (address) {
        Tranche storage tranche = pools[poolId].tranches[trancheId];
        require(tranche.token != address(0), "PoolManager/tranche-does-not-exist"); // Tranche must have been added
        require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool


        address liquidityPool = tranche.liquidityPools[currency];
        require(liquidityPool == address(0), "PoolManager/liquidityPool-already-deployed");
        require(pools[poolId].createdAt != 0, "PoolManager/pool-does-not-exist"); 
```
so the check `        require(pools[poolId].createdAt != 0, "PoolManager/pool-does-not-exist"); 
` can be removed 
## the function `addTranche()` should validate the tranche token decimals to make sure that the decimals are not greater than 18 . 
```solidity 
    function addTranche(
        uint64 poolId,
        bytes16 trancheId,
        string memory tokenName,
        string memory tokenSymbol,
        uint8 decimals
    ) public onlyGateway {
        Pool storage pool = pools[poolId];
        require(pool.createdAt != 0, "PoolManager/invalid-pool");
        Tranche storage tranche = pool.tranches[trancheId];
        require(tranche.createdAt == 0, "PoolManager/tranche-already-exists");


        tranche.poolId = poolId;
        tranche.trancheId = trancheId;
   @>   tranche.decimals = decimals;
        tranche.tokenName = tokenName;
        tranche.tokenSymbol = tokenSymbol;
        tranche.createdAt = block.timestamp;


        emit TrancheAdded(poolId, trancheId);
    }
```
## the function `updateMember()` change the state of the contract , but there is no emitted event , so an event should be emitted . 
the function `updateMember` in the restriction manager contract should emit the event `updatedMember()` with the right parameters . 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L57-L60
```solidity
    function updateMember(address user, uint256 validUntil) public auth {
        require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");
        members[user] = validUntil;
    } 
```
## the poolManager should emit events in case of transfer the tokens to the centrifuge chains 
in the functions `transferTrancheTokensToCentrifuge` , `transferTrancheTokensToEVM` and `transfer()` which burn the tokens and transfer the assets , which is a crucial part of the contract that should emit events in case of transfer of the assets and the burn of the tokens 

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L136-L147

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L149-L163

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L128-L134

## the price on the liquidity pool should be updated first before call `previewDeposit` and `previewMint` , to make sure that the amount of the tranche token that the user will get is accurate . 

the functions `previewDeposit` and `previewMint` should make a call to the centrifuge chain to trigger the function updatePrice on the liquidity pool to use the most recent price to calculate the amount of the tokens that the user will get . 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L135-L137
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L370-L380
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L383-L393

## checkTransferRestriction check can be removed because the tranche token transferFrom function do this check before transfer the tokens 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L473
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L474-L477
```solidity
        require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");
        require(
            lPool.transferFrom(address(escrow), user, trancheTokenAmount),
            "InvestmentManager/trancheTokens-transfer-failed"
        );
```
the `checkTransferRestriction` is performed in the `transferFrom` call of the tranche token . 

## consider adding `getLpValues()` to get the lpValuse from the orderBook to enhance the code 

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L247

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L268
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L561

the contract repeat the code of getting the lpValuse from the storage more than 10 times so the function `getLpValues()` should be added .  
```solidity 
function _getLpValues(address user , address liquidityPool) private retruns ( LPValues storage lpValues ) {
      lpValues =  orderbook[user][liquidityPool];
}
```
