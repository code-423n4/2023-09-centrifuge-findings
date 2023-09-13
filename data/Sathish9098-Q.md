approving uint256.max reverts 

hash collision abi.encodepacked

zero address minting and transfers should be avoided 


need to use salt to avoid replying attacks 

All hardcoded values correct also compatible for all chains ?


##

## [L-] The owner is a single point of failure and a centralization risk

### Impact
Having a ``single EOA`` as the ``owner`` of contracts is a large ``centralization risk`` and a ``single point of failure``. A single ``private key`` may be taken in a ``hack``, or the ``sole holder`` of the key may become unable to retrieve the key when necessary. 

## POC
```solidity
FILE: 2023-09-centrifuge/src/LiquidityPool.sol

214: function requestDeposit(uint256 assets, address owner) public withApproval(owner) {

231: function requestRedeem(uint256 shares, address owner) public withApproval(owner) {

247: function decreaseDepositRequest(uint256 assets, address owner) public withApproval(owner) {

253: function decreaseRedeemRequest(uint256 shares, address owner) public withApproval(owner) {

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L214

### Recommended Mitigation
Consider changing to a ``multi-signature setup``, or having a ``role-based authorization model``.

##

## [L-] approve()/safeApprove() may revert if the current approval is not zero

### Impact
Calling approve() without first calling approve(0) if the current approval is non-zero will revert with some tokens, such as Tether (USDT). While Tether is known to do this, it applies to other tokens as well, which are trying to protect against this attack vector. safeApprove() itself also implements this protection. Always reset the approval to zero before changing it to a new value, or use safeIncreaseAllowance()/safeDecreaseAllowance()

### POC

```solidity
FILE: 2023-09-centrifuge/src/token/ERC20.sol

131: function approve(address spender, uint256 value) external returns (bool) {

```

```solidity
FILE: 2023-09-centrifuge/src/PoolManager.sol

249: EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);

252: EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);

261: EscrowLike(escrow).approve(currencyAddress, address(this), amount);

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L249

### Recommended Mitigation
First approve(0) then approve the amount 

##

## [L-3] ``approve()`` return value not checked

The ``approve()`` function returns a ``boolean`` value indicating whether or not the approval was successful. If the approval was successful, the return value will be ``true``. If the approval was not successful, the return value will be ``false``.

Not checking the ``return value`` of the ``approve()`` function can lead to ``security vulnerabilities``.

```solidity
FILE: 2023-09-centrifuge/src/PoolManager.sol

249: EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);

252: EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);

261: EscrowLike(escrow).approve(currencyAddress, address(this), amount);

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L249

### Recommended Mitigation
Check the status of the ``approve()`` function

## [L-] External call ``recipient`` may consume all transaction gas

### Impact
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use addr.call{gas: <amount>}("") or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

### POC
```solidity
FILE: Breadcrumbs2023-09-centrifuge/src/LiquidityPool.sol

296: (bool success, bytes memory data) = address(share).call(bytes.concat(msg.data, bytes20(msg.sender)));

302: (bool success, bytes memory data) = address(share).call(bytes.concat(msg.data, bytes20(msg.sender)));

308: (bool success, bytes memory data) = address(share).call(bytes.concat(msg.data, bytes20(msg.sender)));

314: (bool success,) = address(share).call(bytes.concat(msg.data, bytes20(address(this))));

319: (bool success,) = address(share).call(bytes.concat(msg.data, bytes20(address(this))));

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L296

##
## [L-] Solidity version 0.8.21 may not work on other chains due to PUSH0

### Impact
The compiler for Solidity 0.8.21 switches the default target EVM version to Shanghai, which includes the new PUSH0 op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier EVM version. While the project itself may or may not compile with 0.8.21, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

All contracts and librarires using the solidity version 0.8.21. Still ``PUSH0`` opcode is not deprecated 

### POC
```solidity
FILE: Breadcrumbs2023-09-centrifuge/src/LiquidityPool.sol

2: pragma solidity 0.8.21;

```

### Recommended Mitgation
Use lower version solidity like ``0.8.19 ``

##

## [L-] Unsafe downcast

### Impact
When a type is downcast to a smaller type, the higher order bits are truncated, effectively applying a modulo to the original value. Without any other checks, this wrapping will lead to unexpected behavior and bugs

``_value`` ``uint256`` is unsafely down casted to ``uint128``

### POC
```solidity
FILE: 2023-09-centrifuge/src/InvestmentManager.sol

670: value = uint128(_value);

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L670

### Recommended Mitgation
Use ``OpenZeppelin SafeCast Libraries ``


##

## [L-] Using ``block.timestamp`` as the deadline/expiry invites ``MEV``

### Impact
Passing ``block.timestamp`` as the expiry/deadline of an operation does not mean ``require immediate execution`` - it means ``whatever block this transaction appears in, I'm comfortable with that block's timestamp``. Providing this value means that a malicious miner can hold the transaction for as long as they like (think the ``flashbots mempool`` for bundling transactions), which may be until they are able to cause the transaction to incur the maximum amount of slippage allowed by the slippage parameter, or until conditions become unfavorable enough that other orders, e.g. liquidations, are triggered. Timestamps should be chosen off-chain, and should be specified by the caller to avoid ``unnecessary MEV``.

### POC

```solidity
FILE: 2023-09-centrifuge/src/token/RestrictionManager.sol

46: require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");
50: if (members[user] >= block.timestamp) {
58: require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L50

```solidity
FILE: 2023-09-centrifuge/src/Root.sol

77: require(schedule[target] < block.timestamp, "Root/target-not-ready");

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L77

```solidity
FILE: 2023-09-centrifuge/src/token/ERC20.sol

217: require(block.timestamp <= deadline, "ERC20/permit-expired");

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L217

### Recommended Mitgation
Timestamps should be chosen ``off-chain``, and should be specified by the ``caller``
























