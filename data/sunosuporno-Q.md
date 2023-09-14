## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Return values of `approve()` not checked | 3 |
| [NC-2](#NC-2) | Constants should be defined rather than using magic numbers | 2 |
### [NC-1] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (3)*:
```solidity
File: src/PoolManager.sol

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

Github Links [1](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L261) [2](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L328) [3](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L329)
### [NC-2] Constants should be defined rather than using magic numbers

*Instances (2)*:
```solidity
File: src/token/Tranche.sol

108:                 sender := shr(96, calldataload(sub(calldatasize(), 20)))

```

```solidity
File: src/util/BytesLib.sol

52:                 mstore(0x40, and(add(mc, 31), not(31)))

```
Github Links [1](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L108) [2](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/BytesLib.sol#L52)

## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-2](#L-2) | Do not use deprecated library functions | 2 |
| [L-3](#L-3) | Empty Function Body - Consider commenting why | 1 |
| [L-4](#L-4) | Unsafe ERC20 operation(s) | 11 |
### [L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: src/util/Factory.sol

94:         bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId));

```
Github Links - [1](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Factory.sol#L94)

### [L-2] Do not use deprecated library functions

*Instances (2)*:
```solidity
File: src/Escrow.sol

24:         SafeTransferLib.safeApprove(token, spender, value);

```

```solidity
File: src/util/SafeTransferLib.sol

36:     function safeApprove(address token, address to, uint256 value) internal {

```
Github Links [1](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Escrow.sol#L24) [2](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Escrow.sol#L36)

### [L-3] Empty Function Body - Consider commenting why

*Instances (1)*:
```solidity
File: src/token/Tranche.sol

33:     constructor(uint8 decimals_) ERC20(decimals_) {}

```
Github Links [1](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L33)

### [L-4] Unsafe ERC20 operation(s)

*Instances (11)*:
```solidity
File: src/InvestmentManager.sol

167:         lPool.transferFrom(user, address(escrow), _trancheTokenAmount);

313:             LiquidityPoolLike(liquidityPool).transferFrom(address(escrow), user, trancheTokenPayout),

475:             lPool.transferFrom(address(escrow), user, trancheTokenAmount),

```

```solidity
File: src/PoolManager.sol

133:         gateway.transfer(currency, msg.sender, recipient, amount);

249:         EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);

252:         EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

```solidity
File: src/token/Tranche.sol

60:         return super.transfer(to, value);

69:         return super.transferFrom(from, to, value);

```

Github Links [1](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L167) [2](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L313) [3](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L475) [4](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L133) [5](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L249) [6](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L252) [7](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L261) [8](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L328) [9](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L329) [10](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L60) [11](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L69)