## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-loss-of-precision)] | **Ward/Authorized admin** can't able to execute functions in `LiquidityPool.sol` | 6 | 
| [[L&#x2011;02](#l02-event-for-critical-parameters-change)] | Missing Event for critical parameters change | 2 | 
| [[L&#x2011;03](#l03-delay-can-be-set-to-indefinite-value)] | `delay` can unintentionally be set to an indefinite value | 1 | 

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-remove-commented-code)] | Remove commented code | 1 | 
| [[N&#x2011;02](#n02-require-statement-is-not-needed)] | `require` statement is not needed in `src/PoolManager.sol#deployLiquidityPool` | 1 | 
| [[N&#x2011;03](#n03-Adding-events-enhances-transparency)] | Adding events within the incoming and outgoing functions enhances transparency | 1 | 


## Low Risk Issues

### [L&#x2011;01] **Ward/Authorized admin** can't able to execute functions in `LiquidityPool.sol`

Some functions in `src/LiquidityPool.sol` are intended to be executed by **ward/authorized admins**. However, these functions are not being executed by **ward/authorized admins** due to the faulty implementation of the `withApproval()` modifier.

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L96C4-L100C6

```solidity
    /// @dev Either msg.sender is the owner or a ward on the contract
    modifier withApproval(address owner) {
        require(msg.sender == owner, "LiquidityPool/no-approval");
        _;
    }
```

The comment in the preceding code suggests that the code is meant to grant access to a ward, but the logic to do so has not been implemented.

As a result of this issue, authorized admins are unable to access the following functions.

*There is 6 instance of this issue:*

*GitHub*: [174](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L174C1-L184C6), [200](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L200C2-L208C6), [214](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L214C1-L217C6), [231](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L231C1-L234C6), [247](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L247C1-L249C6), [253](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L253C1-L255C6)

### [L&#x2011;02] Missing Event for critical parameters change

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

*There is 2 instance of this issue:*

```solidity
File: src/LiquidityPool.sol

    /// @notice Request decreasing the outstanding deposit orders. Will return the assets once the order
    ///         on Centrifuge is successfully decreased.
    function decreaseDepositRequest(uint256 assets, address owner) public withApproval(owner) {
        investmentManager.decreaseDepositRequest(assets, owner);
    }


    /// @notice Request decreasing the outstanding redemption orders. Will return the shares once the order
    ///         on Centrifuge is successfully decreased.
    function decreaseRedeemRequest(uint256 shares, address owner) public withApproval(owner) {
        investmentManager.decreaseRedeemRequest(shares, owner);
    }

```
*GitHub*: [245](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L245C1-L255C6)

```solidity
File: src/PoolManager.sol

    function updateTrancheTokenMetadata(
        uint64 poolId,
        bytes16 trancheId,
        string memory tokenName,
        string memory tokenSymbol
    ) public onlyGateway {
        TrancheTokenLike trancheToken = TrancheTokenLike(getTrancheToken(poolId, trancheId));
        require(address(trancheToken) != address(0), "PoolManager/unknown-token");


        trancheToken.file("name", tokenName);
        trancheToken.file("symbol", tokenSymbol);
        // @audit no event
    }


    function updateMember(uint64 poolId, bytes16 trancheId, address user, uint64 validUntil) public onlyGateway {
        TrancheTokenLike trancheToken = TrancheTokenLike(getTrancheToken(poolId, trancheId));
        require(address(trancheToken) != address(0), "PoolManager/unknown-token");


        MemberlistLike memberlist = MemberlistLike(address(trancheToken.restrictionManager()));
        memberlist.updateMember(user, validUntil);
        // @audit no event
    }
```
*GitHub*: [214](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L214C1-L233C6)

```solidity
File: src/token/RestrictionManager.sol

    function updateMember(address user, uint256 validUntil) public auth {
        require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");
        members[user] = validUntil;
    }


    function updateMembers(address[] memory users, uint256 validUntil) public auth {
        uint256 userLength = users.length;
        for (uint256 i = 0; i < userLength; i++) {
            updateMember(users[i], validUntil);
```
*GitHub*: [57](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L57C2-L65C6)

### [L&#x2011;03] `delay` can unintentionally be set to an indefinite value.

The lack of a check to ensure that the `delay` parameter is less than or equal to `MAX_DELAY` in the constructor of `src/Root.sol` can result in a delay that may not be reached in the near future.

```solidity
File: src/Root.sol

34:    constructor(address _escrow, uint256 _delay) {
35:        escrow = _escrow; 
36:        delay = _delay; // @audit not checking _delay <= MAX_DELAY

```
*GitHub*: [34](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34C4-L36C24)

#### Recommendation
use `require(data <= MAX_DELAY, "Root/delay-too-long");` in the constructor before setting the delay.

## Non-critical Issues

### [N&#x2011;01] Remove commented code

"Please remove the unnecessary commented-out code from `src/LiquidityPool.sol`."

*There are 1 instances of this issue:*

```solidity
File: src/LiquidityPool.sol

150:   // require(receiver == msg.sender, LiquidityPool/not-authorized-to-mint");

```
*GitHub*: [150](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L149)


### [N&#x2011;02] `require` statement is not needed in `src/PoolManager.sol#deployLiquidityPool`

```solidity
File: src/PoolManager.sol

310: require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool
```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L310

The above `require` statement will never evaluate to `false` because `isAllowedAsPoolCurrency()` always returns `true`, and it reverts otherwise. Therefore, the `require` statement can be replaced with a direct call to `isAllowedAsPoolCurrency()`.

```diff
     function deployLiquidityPool(uint64 poolId, bytes16 trancheId, address currency) public returns (address) {
         Tranche storage tranche = pools[poolId].tranches[trancheId];
         require(tranche.token != address(0), "PoolManager/tranche-does-not-exist"); // Tranche must have been added
-        require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool
+        isAllowedAsPoolCurrency(poolId, currency); // Currency must be supported by pool 

```

### [N&#x2011;03] Adding events within the incoming and outgoing functions in `src/gateway/routers/axelar/Router.sol` enhances transparency

Adding events within the incoming and outgoing functions of `src/gateway/routers/axelar/Router.sol` enhances transparency for external parties and provides more comprehensive logging.

Please add events to the individual functions in the code below.

```solidity
File: src/gateway/routers/axelar/Router.sol

    // --- Incoming ---
    function execute(
        bytes32 commandId,
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) public onlyCentrifugeChainOrigin(sourceChain, sourceAddress) {
        bytes32 payloadHash = keccak256(payload);
        require(
            axelarGateway.validateContractCall(commandId, sourceChain, sourceAddress, payloadHash),
            "Router/not-approved-by-gateway"
        );


        gateway.handle(payload);
    }


    // --- Outgoing ---
    function send(bytes calldata message) public onlyGateway {
        axelarGateway.callContract(axelarCentrifugeChainId, centrifugeGatewayPrecompileAddress, message);
    }
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/routers/axelar/Router.sol#L72C1-L91C6