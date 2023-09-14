| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Issues with Large Approvals in ERC20 Tokens | 6 |
| [L-2](#L-2) | Return values of ``approve`` not checked | 6 |
| [L-3](#L-3) | ``approve`` can revert if the current approval is not zero | 6 |
| [L-4](#L-4) | Timestamp may be manipulation | 9 |
| [L-5](#L-5) | Some tokens may revert when large transfers are made | 12 |
| [L-6](#L-6) |  Revert on transfer to the zero address | 2 |



### Report on Issue [L-1]:Issues with Large Approvals in ERC20 Tokens

#### Description:
potential problems related to large approvals in ERC20 tokens, specifically regarding the compliance of `IERC20` implementations. Not all implementations of `IERC20` are fully compliant, and some tokens (e.g., UNI and COMP) may fail if the value passed for approval is larger than a `uint96`. This issue can lead to unexpected behavior or failures when working with such tokens.

#### Source:
Source information for this issue can be found here: [https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers).

Instances (6)*:
```solidity
File: src/PoolManager.sol

249:         EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);

252:         EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

```solidity
File: src/util/SafeTransferLib.sol

37:         (bool success, bytes memory data) = token.call(abi.encodeWithSelector(IERC20.approve.selector, to, value));

```

### [L-2] Return values of ``approve`` not checked
Not all ``IERC20`` implementations ``revert`` when there a failure in ``approve``. The function signature has a boolean return value and they indicate errors that way instead.By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything.

*Instances (6)*:
```solidity
File: src/PoolManager.sol

249:         EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);

252:         EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

```solidity
File: src/util/SafeTransferLib.sol

37:         (bool success, bytes memory data) = token.call(abi.encodeWithSelector(IERC20.approve.selector, to, value));

```

### [L-3] ``approve`` can revert if the current approval is not zero
Some tokens like USDT check for the current approval, and they revert if it not zero. While Tether is known to do this, it applies to other tokens as well, which are trying to protect against this attack vector.Source: https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.m9fhqynw2xvt

*Instances (6)*:
```solidity
File: src/PoolManager.sol

249:         EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);

252:         EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

```solidity
File: src/util/SafeTransferLib.sol

37:         (bool success, bytes memory data) = token.call(abi.encodeWithSelector(IERC20.approve.selector, to, value));

```

### [L-4] Timestamp may be manipulation
The block.timestamp can be manipulated by miners to perform MEV profiting or other time-based attacks.

*Instances (9)*:
```solidity
File: src/LiquidityPool.sol

326:         lastPriceUpdate = block.timestamp;

```

```solidity
File: src/PoolManager.sol

172:         pool.createdAt = block.timestamp;

209:         tranche.createdAt = block.timestamp;

```

```solidity
File: src/Root.sol

66:         schedule[target] = block.timestamp + delay;

77:         require(schedule[target] < block.timestamp, "Root/target-not-ready");

```

```solidity
File: src/token/ERC20.sol

217:         require(block.timestamp <= deadline, "ERC20/permit-expired");

```

```solidity
File: src/token/RestrictionManager.sol

46:         require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");

50:         if (members[user] >= block.timestamp) {

58:         require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");

```


### [L-5] Some tokens may revert when large transfers are made
Tokens such as COMP or UNI will revert when an address balance reaches type(uint96).max. Ensure that the calls below can be broken up into smaller batches if necessary.

*Instances (12)*:
```solidity
File: src/InvestmentManager.sol

167:         lPool.transferFrom(user, address(escrow), _trancheTokenAmount);

272:         userEscrow.transferIn(_currency, address(escrow), recipient, currencyPayout);

313:             LiquidityPoolLike(liquidityPool).transferFrom(address(escrow), user, trancheTokenPayout),

475:             lPool.transferFrom(address(escrow), user, trancheTokenAmount),

545:         userEscrow.transferOut(lPool.asset(), user, receiver, currencyAmount);

```

```solidity
File: src/PoolManager.sol

133:         gateway.transfer(currency, msg.sender, recipient, amount);

146:         gateway.transferTrancheTokensToCentrifuge(poolId, trancheId, msg.sender, destinationAddress, amount);

160:         gateway.transferTrancheTokensToEVM(

```

```solidity
File: src/token/Tranche.sol

60:         return super.transfer(to, value);

69:         return super.transferFrom(from, to, value);

```

```solidity
File: src/util/SafeTransferLib.sol

17:             token.call(abi.encodeWithSelector(IERC20.transferFrom.selector, from, to, value));

27:         (bool success, bytes memory data) = token.call(abi.encodeWithSelector(IERC20.transfer.selector, to, value));

```

### [L-6] Revert on transfer to the zero address
Its good practice to revert a token transfer transaction if the recipients address is the zero address. This can prevent unintentional transfers to the zero address due to accidental operations or programming errors. Many token contracts implement such a safeguard, such as OpenZeppelin - ERC20, OpenZeppelin - ERC721.

*Instances (2)*:
```solidity
File: src/PoolManager.sol

133:         gateway.transfer(currency, msg.sender, recipient, amount);

```

```solidity
File: src/token/Tranche.sol

60:         return super.transfer(to, value);

```