## Non Critical Issues

|      | Issue                                                       |
| ---- | :---------------------------------------------------------- |
| NC-1 | Avoid passing as reference the `chainid`                    |
| NC-2 | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)        |
| NC-3 | GENERATE PERFECT CODE HEADERS EVERY TIME                    |
| NC-4 | SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`               |
| NC-5 | ALLOWS MALLEABLE SECP256K1 SIGNATURES                       |
| NC-6 | Omissions in events                                         |
| NC-7 | Return values of `approve()` not checked                    |
| NC-8 | Constants should be defined rather than using magic numbers |

### [NC-1] Avoid passing as reference the `chainid`

#### Description:

Instead of passing chainid as reference you could get current `chainid` using `block.chainid`

Please see:

https://docs.soliditylang.org/en/v0.8.0/units-and-global-variables.html#block-and-transaction-properties

#### **Proof Of Concept**

```solidity
File: src/token/ERC20.sol

47:         deploymentChainId = block.chainid;

48:         _DOMAIN_SEPARATOR = _calculateDomainSeparator(block.chainid);

80:         return block.chainid == deploymentChainId ? _DOMAIN_SEPARATOR : _calculateDomainSeparator(block.chainid);

80:         return block.chainid == deploymentChainId ? _DOMAIN_SEPARATOR : _calculateDomainSeparator(block.chainid);

228:                 block.chainid == deploymentChainId ? _DOMAIN_SEPARATOR : _calculateDomainSeparator(block.chainid),

228:                 block.chainid == deploymentChainId ? _DOMAIN_SEPARATOR : _calculateDomainSeparator(block.chainid),

```

### [NC-2] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

#### **Proof Of Concept**

```solidity
File: src/gateway/Messages.sol

80:         return abi.encodePacked(uint8(Call.AddCurrency), currency, currencyAddress);

99:         return abi.encodePacked(uint8(Call.AddPool), poolId);

118:         return abi.encodePacked(uint8(Call.AllowPoolCurrency), poolId, currency);

210:         return abi.encodePacked(uint8(Call.UpdateMember), poolId, trancheId, member, validUntil);

242:         return abi.encodePacked(uint8(Call.UpdateTrancheTokenPrice), poolId, trancheId, currencyId, price);

274:         return abi.encodePacked(uint8(Call.Transfer), currency, sender, receiver, amount);

381:         return abi.encodePacked(uint8(Call.IncreaseInvestOrder), poolId, trancheId, investor, currency, amount);

417:         return abi.encodePacked(uint8(Call.DecreaseInvestOrder), poolId, trancheId, investor, currency, amount);

449:         return abi.encodePacked(uint8(Call.IncreaseRedeemOrder), poolId, trancheId, investor, currency, amount);

481:         return abi.encodePacked(uint8(Call.DecreaseRedeemOrder), poolId, trancheId, investor, currency, amount);

509:         return abi.encodePacked(uint8(Call.CollectInvest), poolId, trancheId, investor, currency);

540:         return abi.encodePacked(uint8(Call.CollectRedeem), poolId, trancheId, investor, currency);

701:         return abi.encodePacked(uint8(Call.ScheduleUpgrade), _contract);

713:         return abi.encodePacked(uint8(Call.CancelUpgrade), _contract);

771:         return abi.encodePacked(uint8(Call.CancelInvestOrder), poolId, trancheId, investor, currency);

790:         return abi.encodePacked(uint8(Call.CancelRedeemOrder), poolId, trancheId, investor, currency);

817:         return abi.encodePacked(uint8(Call.UpdateTrancheInvestmentLimit), poolId, trancheId, investmentLimit);

841:         return bytes9(BytesLib.slice(abi.encodePacked(uint8(domain), chainId), 0, 9));

```

```solidity
File: src/token/ERC20.sol

242:         permit(owner, spender, value, deadline, abi.encodePacked(r, s, v));

```

```solidity
File: src/util/Factory.sol

94:         bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId));

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`

#### Description:

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Use the `ecrecover` function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

```solidity
File: src/token/ERC20.sol

206:             if (signer == ecrecover(digest, v, r, s)) {

```

### [NC-5] ALLOWS MALLEABLE SECP256K1 SIGNATURES

#### Description:

Here, the `ecrecover()` method doesn’t check the s range.

Homestead (EIP-2) added this limitation, however the precompile remained unaltered. The majority of libraries, including OpenZeppelin, do this check.

Since an order can only be confirmed once and its hash is saved, there doesn’t seem to be a serious danger in existing use cases.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7201e6707f6631d9499a569f492870ebdd4133cf/contracts/utils/cryptography/ECDSA.sol#L138-L149

#### **Proof Of Concept**

```solidity
File: src/token/ERC20.sol

206:             if (signer == ecrecover(digest, v, r, s)) {

```

### [NC-6] Omissions in events

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible.

#### **Proof Of Concept**

```solidity
File: src/Escrow.sol

19:         emit Rely(msg.sender);

```

```solidity
File: src/InvestmentManager.sol

93:         emit Rely(msg.sender);

```

```solidity
File: src/LiquidityPool.sol

93:         emit Rely(msg.sender);

261:         emit DepositCollected(receiver);

267:         emit RedeemCollected(receiver);

327:         emit UpdatePrice(price);

```

```solidity
File: src/PoolManager.sol

110:         emit Rely(msg.sender);

173:         emit PoolAdded(poolId);

```

```solidity
File: src/Root.sol

39:         emit Rely(msg.sender);

56:         emit Pause();

61:         emit Unpause();

72:         emit RelyCancelled(target);

80:         emit Rely(target);

```

```solidity
File: src/UserEscrow.sol

25:         emit Rely(msg.sender);

```

```solidity
File: src/admins/DelayedAdmin.sol

22:         emit Rely(msg.sender);

```

```solidity
File: src/admins/PauseAdmin.sol

25:         emit Rely(msg.sender);

36:         emit AddPauser(user);

41:         emit RemovePauser(user);

```

```solidity
File: src/gateway/Gateway.sol

106:         emit Rely(msg.sender);

139:         emit AddIncomingRouter(router);

144:         emit RemoveIncomingRouter(router);

149:         emit UpdateOutgoingRouter(router);

```

```solidity
File: src/gateway/routers/axelar/Router.sol

40:         emit Rely(msg.sender);

```

```solidity
File: src/token/ERC20.sol

45:         emit Rely(_msgSender());

59:         emit Rely(user);

64:         emit Deny(user);

```

```solidity
File: src/token/RestrictionManager.sol

24:         emit Rely(msg.sender);

```

```solidity
File: src/token/Tranche.sol

50:         emit AddLiquidityPool(liquidityPool);

55:         emit RemoveLiquidityPool(liquidityPool);

```

```solidity
File: src/util/Auth.sol

16:         emit Rely(user);

22:         emit Deny(user);

```

```solidity
File: src/util/Factory.sol

33:         emit Rely(msg.sender);

78:         emit Rely(msg.sender);

```

### [NC-7] Return values of `approve()` not checked

#### Description:

Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

#### **Proof Of Concept**

```solidity
File: src/PoolManager.sol

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

### [NC-8] Constants should be defined rather than using magic numbers

#### **Proof Of Concept**

```solidity
File: src/token/Tranche.sol

108:                 sender := shr(96, calldataload(sub(calldatasize(), 20)))

```

## Low Issues

|     | Issue                                                 |
| --- | :---------------------------------------------------- |
| L-1 | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()   |
| L-2 | Avoid `transfer()`/`send()` as reentrancy mitigations |
| L-3 | Avoid using low call function ecrecover               |
| L-4 | DON’T USE OWNER AND TIMELOCK                          |

### [L-1] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()

#### **Proof Of Concept**

```solidity
File: src/util/Factory.sol

94:         bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId));

```

### [L-2] Avoid `transfer()`/`send()` as reentrancy mitigations

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

```solidity
File: src/LiquidityPool.sol

301:     function transfer(address, uint256) public returns (bool) {

```

```solidity
File: src/PoolManager.sol

27:     function transfer(uint128 currency, address sender, bytes32 recipient, uint128 amount) external;

128:     function transfer(address currencyAddress, bytes32 recipient, uint128 amount) public {

133:         gateway.transfer(currency, msg.sender, recipient, amount);

```

```solidity
File: src/gateway/Gateway.sol

65:     function send(bytes memory message) external;

160:         outgoingRouter.send(

180:         outgoingRouter.send(

192:     function transfer(uint128 token, address sender, bytes32 receiver, uint128 amount)

197:         outgoingRouter.send(Messages.formatTransfer(token, _addressToBytes32(sender), receiver, amount));

207:         outgoingRouter.send(

219:         outgoingRouter.send(

231:         outgoingRouter.send(

245:         outgoingRouter.send(

257:         outgoingRouter.send(Messages.formatCollectInvest(poolId, trancheId, _addressToBytes32(investor), currency));

265:         outgoingRouter.send(Messages.formatCollectRedeem(poolId, trancheId, _addressToBytes32(investor), currency));

273:         outgoingRouter.send(Messages.formatCancelInvestOrder(poolId, trancheId, _addressToBytes32(investor), currency));

281:         outgoingRouter.send(Messages.formatCancelRedeemOrder(poolId, trancheId, _addressToBytes32(investor), currency));

```

```solidity
File: src/gateway/routers/axelar/Router.sol

89:     function send(bytes calldata message) public onlyGateway {

```

```solidity
File: src/token/ERC20.sol

91:     function transfer(address to, uint256 value) public virtual returns (bool) {

```

```solidity
File: src/token/Tranche.sol

59:     function transfer(address to, uint256 value) public override restricted(_msgSender(), to, value) returns (bool) {

60:         return super.transfer(to, value);

```

### [L-3] Avoid using low call function ecrecover

#### Description:

Use OZ library ECDSA that its battle tested to avoid classic errors.

contracts/utils/cryptography/ECDSA.sol

https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA

#### **Proof Of Concept**

```solidity
File: src/token/ERC20.sol

206:             if (signer == ecrecover(digest, v, r, s)) {

```

### [L-4] DON’T USE OWNER AND TIMELOCK

#### Description:

Using a timelock contract gives confidence to the user, but when check onlyByOwnGov allow the owner and the timelock The owner manipulates the contract without a lock time period.

#### **Proof Of Concept**

```solidity
File: src/LiquidityPool.sol

98:         require(msg.sender == owner, "LiquidityPool/no-approval");

```
### [L-5] `ECRECOVER` MAY RETURN EMPTY ADDRESS

#### Description:

There is a common issue that ecrecover returns empty (0x0) address when the signature is invalid. function `_verifySigner` should check that before returning the result of ecrecover.

#### **Proof Of Concept**

```solidity
File: src/token/ERC20.sol

206:             if (signer == ecrecover(digest, v, r, s)) {

```

#### Recommended Mitigation Steps:

See the solution here: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.4.0/contracts/cryptography/ECDSA.sol#L68