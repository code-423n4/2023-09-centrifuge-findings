# State variable doesn't update at `updateTrancheTokenMetadata`

## Links to affected code

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L214-L225](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L214-L225)

## Impact

`pools[poolId].tranches[trancheId].tokenName` and `pools[poolId].tranches[trancheId].tokenSymbol` are not match with real token name/symbol if updated later.

## Proof of Concept

At `updateTrancheTokenMetadata`, you can change tranche token name and symbol. But it doesn’t change storage variable at PoolManager contract.

```solidity
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
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Change state variable.

```diff
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
+   pools[poolId].tranches[trancheId].tokenName = tokenName;
+   pools[poolId].tranches[trancheId].tokenSymbol = tokenSymbol;
}
```

# Duplicate member check at `handleTransferTrancheTokens`

## Links to affected code

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L273](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L273)

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L49-L53](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L49-L53)

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L72-L74](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L72-L74)

## Impact

Check same condition twice at `handleTransferTrancheTokens`

## Proof of Concept

At `handleTransferTrancheTokens` , it calls `MemberlistLike(address(trancheToken.restrictionManager())).hasMember(destinationAddress)` to check if `destinationAddress` is allowed member or not.

```solidity
function handleTransferTrancheTokens(uint64 poolId, bytes16 trancheId, address destinationAddress, uint128 amount)
    public
    onlyGateway
{
    TrancheTokenLike trancheToken = TrancheTokenLike(getTrancheToken(poolId, trancheId));
    require(address(trancheToken) != address(0), "PoolManager/unknown-token");

    require(
        MemberlistLike(address(trancheToken.restrictionManager())).hasMember(destinationAddress),
        "PoolManager/not-a-member"
    );
    trancheToken.mint(destinationAddress, amount);
}
```

At `trancheToken.mint` , it check same thing at `restricted` modifier.

```solidity
function mint(address to, uint256 value) public override restricted(_msgSender(), to, value) {
    return super.mint(to, value);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function handleTransferTrancheTokens(uint64 poolId, bytes16 trancheId, address destinationAddress, uint128 amount)
    public
    onlyGateway
{
    TrancheTokenLike trancheToken = TrancheTokenLike(getTrancheToken(poolId, trancheId));
    require(address(trancheToken) != address(0), "PoolManager/unknown-token");

-   require(
-       MemberlistLike(address(trancheToken.restrictionManager())).hasMember(destinationAddress),
-       "PoolManager/not-a-member"
-   );
    trancheToken.mint(destinationAddress, amount);
}
```

# Not checking `MAX_DELAY` at Root constructor

## Links to affected code

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L36](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L36)

## Impact

Can set delay more than `MAX_DELAY` at root constructor

## Proof of Concept

```solidity
constructor(address _escrow, uint256 _delay) {
    escrow = _escrow;
    delay = _delay;

    wards[msg.sender] = 1;
    emit Rely(msg.sender);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
constructor(address _escrow, uint256 _delay) {
    escrow = _escrow;
+   require(data <= MAX_DELAY, "Root/delay-too-long");
    delay = _delay;

    wards[msg.sender] = 1;
    emit Rely(msg.sender);
}
```

# Some contract using Auth cannot be managed by Root contract

## Links to affected code

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L90-L101](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L90-L101)

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/script/Deployer.sol#L48-L49](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/script/Deployer.sol#L48-L49)

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/script/Deployer.sol#L55-L56](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/script/Deployer.sol#L55-L56)

## Impact

Some contract using Auth cannot be managed by Root contract

## Proof of Concept

Root contract have functions to manage wards of all the other contracts. To use these functions, all the other contracts should set Root contract as ward.

```solidity
function relyContract(address target, address user) public auth {
    AuthLike(target).rely(user);
    emit RelyContract(target, user);
}

function denyContract(address target, address user) public auth {
    AuthLike(target).deny(user);
    emit DenyContract(target, user);
}
```

At Deploy script, the LiquidityPoolFactory and TrancheTokenFactory, PauseAdmin, DelayedAdmin contracts don’t call `rely` to set root contract ward. And also, these contracts don’t set Root contract as ward at constructor.

```solidity
function deployInvestmentManager() public {
    ...

    LiquidityPoolFactory(liquidityPoolFactory).rely(address(poolManager));
    TrancheTokenFactory(trancheTokenFactory).rely(address(poolManager));
}
function wire(address router) public {
    // Deploy gateway and admins
    pauseAdmin = new PauseAdmin(address(root));
    delayedAdmin = new DelayedAdmin(address(root));
    pauseAdmin.rely(address(delayedAdmin));
    ...
}
```

You can't manage them with management functions unless you add Root as a ward.

## Tools Used

Manual Review

## Recommended Mitigation Steps

To avoid these situation, make Root contract the default ward at contructor. This is an example.(DelayedAdmin constructor)

```diff
constructor(address root_) {
    root = Root(root_);
+   wards[root_] = 1;
+   emit Rely(root_);
    wards[msg.sender] = 1;
    emit Rely(msg.sender);
}
```

# `detectTransferRestriction` do not check `from` parameter so that `restricted` modifier `from` parameter doesn’t need to set perfectly

## Links to affected code

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L28-L34](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L28-L34)

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L59-L61](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L59-L61)

[https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L72-L74](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L72-L74)

## Impact

Useless functino call for useless parameter

## Proof of Concept

At `detectTransferRestriction` , it does not use `from` parameter at all. But it cannot be removed to comply with ERC1404.

```solidity
function detectTransferRestriction(address from, address to, uint256 value) public view returns (uint8) {
    if (!hasMember(to)) {
        return DESTINATION_NOT_A_MEMBER_RESTRICTION_CODE;
    }

    return SUCCESS_CODE;
}
```

But at Tranche contract, you don’t need to use `_msgSender()` to set `restricted` modifier `from` parameter, because it will not be used. 

```solidity
modifier restricted(address from, address to, uint256 value) {
    uint8 restrictionCode = detectTransferRestriction(from, to, value);
    require(restrictionCode == restrictionManager.SUCCESS_CODE(), messageForTransferRestriction(restrictionCode));
    _;
}

function transfer(address to, uint256 value) public override restricted(_msgSender(), to, value) returns (bool) {
    return super.transfer(to, value);
}

function mint(address to, uint256 value) public override restricted(_msgSender(), to, value) {
    return super.mint(to, value);
}
```