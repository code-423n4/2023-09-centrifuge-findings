# LOW FINDINGS

##

## [L-1] The owner is a single point of failure and a centralization risk

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

## [L-2] approve()/safeApprove() may revert if the current approval is not zero

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

##

## [L-4] External call ``recipient`` may consume all transaction gas

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
## [L-5] Solidity version 0.8.21 may not work on other chains due to PUSH0

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

## [L-6] Unsafe downcast

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

## [L-7] Using ``block.timestamp`` as the deadline/expiry invites ``MEV``

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

##

## [L-8] ``MAX_DELAY`` should be configurable to avoid problems in future 

If there are any future changes, it can be difficult to modify the contract, and redeployment is currently the only viable option. Therefore, it is advisable to add a setter function for ``MAX_DELAY`` to facilitate future adjustments.

```solidity
FILE: 2023-09-centrifuge/src/Root.sol

17: uint256 private MAX_DELAY = 4 weeks;

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L17

### Recommended Mitigation
Add setter function

```solidity

function setMaxDelay(uint256 _newMaxDelay) external onlyOwner {
        MAX_DELAY = _newMaxDelay;
    }

```

##

## [L-9] Shadowed variable in parameters should be avoided 

Shadowed variables in ``parameters`` should be avoided to maintain code clarity, avoid unintended side effects, and make debugging easier. When you shadow a variable in a function's parameters, it can lead to confusion and unexpected behavior.

In [Factory.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol) and [Auth.sol]() contracts same wards variable

``Factory.sol`` used ``wards`` as function parameter and ``Auth.sol`` used ``wards`` as state variable

```solidity
FILE: 2023-09-centrifuge/src/util/Factory.sol

42: address[] calldata wards

47: for (uint256 i = 0; i < wards.length; i++) {
48: liquidityPool.rely(wards[i]);

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Factory.sol#L42

### Recommended Mitigation

```diff

- 42: address[] calldata wards
+ 42: address[] calldata _wards

```
##

## [L-10] Assigning values to state variables without performing sanity checks on integer values

Integer values are being assigned directly to state variables or immutables without any validation or verification to ensure that the assigned values are within acceptable or "sane" ranges . It's crucial to validate data before updating state variables to prevent unexpected behavior or vulnerabilities.

```solidity
FILE: 2023-09-centrifuge/src/Root.sol

36: delay = _delay;

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L36

If ``decimals `` immutable variable sets wrong value then this will cause unintended consequences. Can't changed once value set  

```solidity
FILE: 2023-09-centrifuge/src/token/ERC20.sol

43:    decimals = decimals_;

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L43

### Recommended Mitigation
Add sanity checks for integer like > 0 

```solidity

require(_delay > 0, " wrong value");

```

##

## [L-11] Using zero as a parameter

Taking zero as a valid argument without checks can lead to severe security issues in some cases.

If using a zero argument is mandatory, consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

```solidity
FILE: 2023-09-centrifuge/src/InvestmentManager.sol

177: require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

190: require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

202: require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

213: require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L177

##

## [L-12] no checks for ``source`` and ``destination`` addresses being the same

The function transferIn() in the UserEscrow contract does not check if the source and destination addresses are the same. This means that a user could potentially deposit tokens into their own escrow address.if an attacker is able to compromise a user's escrow account, they could potentially steal all of the tokens in the account, even if the tokens are deposited into the user's own escrow address.

```solidity
FILE: 2023-09-centrifuge/src/UserEscrow.sol

// --- Token transfers ---
    function transferIn(address token, address source, address destination, uint256 amount) external auth {
        destinations[token][destination] += amount;

        SafeTransferLib.safeTransferFrom(token, source, address(this), amount);
        emit TransferIn(token, source, destination, amount);
    }


```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L29

### Recommended Mitigation

```
require(source != destination, "wrong address");

```
##

## [L-13] ``transferIn`` function lacks the token support checks in ``UserEscrow`` contract

### Impact
The function does not check if the token is supported by the escrow. This means that users could potentially deposit tokens into the escrow that cannot be withdrawn.

```solidity
FILE: 2023-09-centrifuge/src/UserEscrow.sol

// --- Token transfers ---
    function transferIn(address token, address source, address destination, uint256 amount) external auth {
        destinations[token][destination] += amount;

        SafeTransferLib.safeTransferFrom(token, source, address(this), amount);
        emit TransferIn(token, source, destination, amount);
    }


```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L29

### Recommended Mitgation
Add a check to ensure that the token is supported by the escrow

##

## [L-14] ``transferOut()`` function no check for receiver being a contract

The function does not check if the receiver is a contract. This means that an attacker could potentially use a malicious contract to steal the tokens from the destination

```solidity
FILE: 2023-09-centrifuge/src/UserEscrow.sol

function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
        require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
        require(
            /// @dev transferOut can only be initiated by the destination address or an authorized admin.
            ///      The check is just an additional protection to secure destination funds in case of compromized auth.
            ///      Since userEscrow is not able to decrease the allowance for the receiver,
            ///      a transfer is only possible in case receiver has received the full allowance from destination address.
            receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max),
            "UserEscrow/receiver-has-no-allowance"
        );
        destinations[token][destination] -= amount;

        SafeTransferLib.safeTransfer(token, receiver, amount);
        emit TransferOut(token, receiver, amount);
    }


```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L36-L50

### Recommended Mitigation
Add a check to ensure that the receiver is not a contract

##

## [L-15] Add a ``timelock`` to the ``transferOut()`` function to prevent ``attackers`` from withdrawing funds immediately after compromising a user's ``escrow account``

### Impact
To add a timelock to the transferOut() function in the UserEscrow contract, you can add a new state variable called timelock and a new modifier called withTimelock. The timelock variable will store the timestamp at which a transfer was requested, and the withTimelock modifier will ensure that a transfer can only be executed after the timelock period has elapsed.

The timelock period can be set to any desired value, such as 24 hours or 7 days. This will prevent attackers from withdrawing funds immediately after compromising a user's escrow account.

```solidity
FILE: 2023-09-centrifuge/src/UserEscrow.sol

function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
        require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
        require(
            /// @dev transferOut can only be initiated by the destination address or an authorized admin.
            ///      The check is just an additional protection to secure destination funds in case of compromized auth.
            ///      Since userEscrow is not able to decrease the allowance for the receiver,
            ///      a transfer is only possible in case receiver has received the full allowance from destination address.
            receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max),
            "UserEscrow/receiver-has-no-allowance"
        );
        destinations[token][destination] -= amount;

        SafeTransferLib.safeTransfer(token, receiver, amount);
        emit TransferOut(token, receiver, amount);
    }

```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L36-L50

### Recommended Mitigation
Add timelock 

```solidity

modifier withTimelock() {
    require(block.timestamp >= timelock, "UserEscrow/timelock-not-expired");
    _;
}

```



















