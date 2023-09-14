## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Prevent `newTrancheToken()` from being Dos-ed | 1 |
| [L&#x2011;02] | Misleading comment in `Context.sol` as it should be `ERC2771` context | 1 |
| [L&#x2011;03] | In `Tranche.sol`, Misleading modifier name `restricted()` should be `notRestricted()` | 1 |
| [L&#x2011;04] | While setting `delay`, the contract does not check `delay` should be less than `maxDelay` | 1 |
| [L&#x2011;05] | The Root.delay will initially be set to 48 hours per docs | 1 |
| [L&#x2011;06] | `updatePrice()` can set the price to 0 | 1 |

### [L&#x2011;01]  Prevent `newTrancheToken()` from being Dos-ed
In `Factory.sol` has `TrancheTokenFactory` which has `newTrancheToken()` which can be Dos-ed or front-run attacker as the salt used does not contain `msg.sender`. It is also recommended to add incremented nonce to avoid the incoming front running issue.

There is [1 instance](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Factory.sol#L94-L96) of this issue:

```Solidity
File: src/util/Factory.sol

    function newTrancheToken(
        uint64 poolId,
        bytes16 trancheId,
        string memory name,
        string memory symbol,
        uint8 decimals,
        address[] calldata trancheTokenWards,
        address[] calldata restrictionManagerWards
    ) public auth returns (address) {
        address restrictionManager = _newRestrictionManager(restrictionManagerWards);

        // Salt is hash(poolId + trancheId)
        // same tranche token address on every evm chain
>>      bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId));

>>      TrancheToken token = new TrancheToken{salt: salt}(decimals);

    // some code


    }
```

### Recommended Mitigation steps

```Solidity
File: src/util/Factory.sol


+    /// @notice Mapping to store deployer nonces for CREATE2
+    mapping(address deployer => uint256 nonce) public deployerNonces;



    function newTrancheToken(
        uint64 poolId,
        bytes16 trancheId,
        string memory name,
        string memory symbol,
        uint8 decimals,
        address[] calldata trancheTokenWards,
        address[] calldata restrictionManagerWards
    ) public auth returns (address) {
        address restrictionManager = _newRestrictionManager(restrictionManagerWards);

        // Salt is hash(poolId + trancheId)
        // same tranche token address on every evm chain
-       bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId));
+       bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId, msg.sender, deployerNonces[msg.sender]++));

        TrancheToken token = new TrancheToken{salt: salt}(decimals);

    // some code


    }
```

### [L&#x2011;02]  Misleading comment in `Context.sol` as it should be `ERC2771` context
In `Context.sol`, There is misleading Natspec as title of contract.

```Solidity
/// @title  ERC1271 Context
```

This is not correct for the context base contract as `ERC-1271` is for Standard Signature Validation method whereas `ERC2771` talks about secure Protocol for Native Meta Transactions and this also confirms the other Natspecin context.sol

### Recommended Mitigation steps

```diff
- /// @title  ERC1271 Context
+ /// @title  ERC2771 Context
```

### [L&#x2011;03]  In `Tranche.sol`, Misleading modifier name `restricted()` should be `notRestricted()`
The modifier checks whether the transfer, mint, transferFrom are restricted per ERC1404. However `restricted()` modifier should be `notRestricted()` which makes more sense as the modifier first condition would be to check the `success code` and in rare case it would revert with error code.

Also refer [this](https://github.com/simple-restricted-token/reference-implementation/blob/master/contracts/token/ERC1404/ERC1404ReferenceImpl.sol) ERC1404 implementation to see `notRestricted` has been used.

There is [1 instance](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L35) of this issue:

### Recommended Mitigation steps

```diff

-    modifier restricted(address from, address to, uint256 value) {
+    modifier NotRestricted(address from, address to, uint256 value) {
```

### [L&#x2011;04]  While setting `delay`, the contract does not check `delay` should be less than `maxDelay`
While setting `delay` in `Root.sol` constructor. It does not check the `delay` must be less than `maxDelay`. With current implementation `delay` can be set to any value which means it can be delayed indefinately.

```Solidity
    /// @dev To prevent filing a delay that would block any updates indefinitely
    uint256 private MAX_DELAY = 4 weeks;
```

There is [1 instance](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34-L40) of this issue:

### Recommended Mitigation steps

```diff

    constructor(address _escrow, uint256 _delay) {
+       require(_delay <= MAX_DELAY, "out of range delay");
        escrow = _escrow;
        delay = _delay;

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }
```
### [L&#x2011;05]  The Root.delay will initially be set to 48 hours per docs
Per contest readme.md states,
> The Root.delay will initially be set to 48 hours.

However this is not implemented in contract.

There is [1 instance](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L22) of this issue:

```Solidity
    uint256 public delay;
```

### Recommended Mitigation steps
The Root.delay() should be set to to 48 hours per contest readme.md

### [L&#x2011;06]  `updatePrice()` can set the price to 0
updatePrice can be used to update the price, however if it is set to zero. Everything multiplied with price will be zero too. 

```Solidity

    function updatePrice(uint128 price) public auth {
        latestPrice = price;
        lastPriceUpdate = block.timestamp;
        emit UpdatePrice(price);
    }
```

### Recommended Mitigation steps
Add price is not equal to 0 validation check.

```diff

    function updatePrice(uint128 price) public auth {
+       require(price != 0, "invalid price");
        latestPrice = price;
        lastPriceUpdate = block.timestamp;
        emit UpdatePrice(price);
    }
```