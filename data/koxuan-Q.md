# Report


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Constants in comparisons should appear on the left side | 22 |
| [NC-2](#NC-2) | delete keyword can be used instead of setting to 0 | 4 |
| [NC-3](#NC-3) | Lines are too long | 7 |
| [NC-4](#NC-4) | Return values of `approve()` not checked | 3 |
| [NC-5](#NC-5) | Constants should be defined rather than using magic numbers | 2 |
| [NC-6](#NC-6) | Variable names don't follow the Solidity style guide | 4 |
| [NC-7](#NC-7) | Variables need not be initialized to zero | 1 |
### <a name="NC-1"></a>[NC-1] Constants in comparisons should appear on the left side
Constants should appear on the left side of comparisons, to avoid accidental assignment

*Instances (22)*:
```solidity
File: InvestmentManager.sol

127:         if (_currencyAmount == 0) {

159:         if (_trancheTokenAmount == 0) {

377:         if (depositPrice == 0) return 0;

390:         if (depositPrice == 0) return 0;

403:         if (redeemPrice == 0) return 0;

416:         if (redeemPrice == 0) return 0;

553:         if (lpValues.maxMint == 0) {

562:         if (lpValues.maxRedeem == 0) {

```

```solidity
File: PoolManager.sol

170:         require(pool.createdAt == 0, "PoolManager/pool-already-added");

202:         require(tranche.createdAt == 0, "PoolManager/tranche-already-exists");

242:         require(currencyAddressToId[currencyAddress] == 0, "PoolManager/currency-address-in-use");

```

```solidity
File: admins/PauseAdmin.sol

29:         require(pausers[msg.sender] == 1, "PauseAdmin/not-authorized-to-pause");

```

```solidity
File: gateway/Messages.sol

860:         require(_bytes128.length == 128, "Input should be 128 bytes");

878:         if (tempEmptyStringTest.length == 0) {

```

```solidity
File: token/ERC20.sol

53:         require(wards[_msgSender()] == 1, "Auth/not-authorized");

197:         if (signature.length == 65) {

213:         return (success && result.length == 32 && abi.decode(result, (bytes4)) == IERC1271.isValidSignature.selector);

```

```solidity
File: util/Auth.sol

27:         require(wards[msg.sender] == 1, "Auth/not-authorized");

```

```solidity
File: util/MathLib.sol

33:             if (prod1 == 0) {

```

```solidity
File: util/SafeTransferLib.sol

18:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-from-failed");

28:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-failed");

38:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");

```

### <a name="NC-2"></a>[NC-2] delete keyword can be used instead of setting to 0
It's clearer and reflects the intention of the programmer

*Instances (4)*:
```solidity
File: InvestmentManager.sol

624:             lpValues.maxDeposit = 0;

629:             lpValues.maxMint = 0;

640:             lpValues.maxWithdraw = 0;

645:             lpValues.maxRedeem = 0;

```

### <a name="NC-3"></a>[NC-3] Lines are too long
Recommended by solidity docs to keep lines to 120 characters or lesser

*Instances (7)*:
```solidity
File: InvestmentManager.sol

251:         LiquidityPoolLike(liquidityPool).mint(address(escrow), trancheTokensPayout); // mint to escrow. Recepeint can claim by calling withdraw / redeem

544:         _decreaseRedemptionLimits(user, liquidityPool, currencyAmount, trancheTokenAmount); // decrease the possible deposit limits

```

```solidity
File: PoolManager.sol

310:         require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

```solidity
File: token/ERC20.sol

165:             balanceOf[to] = balanceOf[to] + value; // note: we don't need an overflow check here b/c balanceOf[to] <= totalSupply and there is an overflow check below

188:             balanceOf[from] = balance - value; // note: we don't need overflow checks b/c require(balance >= value) and balance <= totalSupply

```

### <a name="NC-4"></a>[NC-4] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (3)*:
```solidity
File: PoolManager.sol

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

### <a name="NC-5"></a>[NC-5] Constants should be defined rather than using magic numbers

*Instances (2)*:
```solidity
File: token/Tranche.sol

108:                 sender := shr(96, calldataload(sub(calldatasize(), 20)))

```

```solidity
File: util/BytesLib.sol

52:                 mstore(0x40, and(add(mc, 31), not(31)))

```

### <a name="NC-6"></a>[NC-6] Variable names don't follow the Solidity style guide
Constant variable should be all caps  each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

*Instances (4)*:
```solidity
File: gateway/routers/axelar/Router.sol

25:     string private constant axelarCentrifugeChainId = "centrifuge";

26:     string private constant axelarCentrifugeChainAddress = "0x7369626cef070000000000000000000000000000";

27:     string private constant centrifugeGatewayPrecompileAddress = "0x0000000000000000000000000000000000002048";

```

```solidity
File: token/ERC20.sol

21:     string public constant version = "3";

```

### <a name="NC-7"></a>[NC-7] Variables need not be initialized to zero
Default value is zero which makes the initialization unnecessary.

*Instances (1)*:
```solidity
File: token/RestrictionManager.sol

15:     uint8 public constant SUCCESS_CODE = 0;

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-2](#L-2) | Do not use deprecated library functions | 2 |
| [L-3](#L-3) | Divide before Multiplication | 1 |
| [L-4](#L-4) | Empty Function Body - Consider commenting why | 1 |
| [L-5](#L-5) | ERC20 tokens that do not implement optional decimals method cannot be used | 7 |
| [L-6](#L-6) | Unsafe ERC20 operation(s) | 11 |
### <a name="L-1"></a>[L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: util/Factory.sol

94:         bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId));

```

### <a name="L-2"></a>[L-2] Do not use deprecated library functions

*Instances (2)*:
```solidity
File: Escrow.sol

24:         SafeTransferLib.safeApprove(token, spender, value);

```

```solidity
File: util/SafeTransferLib.sol

36:     function safeApprove(address token, address to, uint256 value) internal {

```

### <a name="L-3"></a>[L-3] Divide before Multiplication
Unnecessary loss of precision caused by divide before multiplication

*Instances (1)*:
```solidity
File: InvestmentManager.sol

692:         value = _toUint128(_value / 10 ** (PRICE_DECIMALS - decimals));

```

### <a name="L-4"></a>[L-4] Empty Function Body - Consider commenting why

*Instances (1)*:
```solidity
File: token/Tranche.sol

33:     constructor(uint8 decimals_) ERC20(decimals_) {}

```

### <a name="L-5"></a>[L-5] ERC20 tokens that do not implement optional decimals method cannot be used
Underlying token that does not implement optional decimals method cannot be used 
 
 > [EIP-20](https://eips.ethereum.org/EIPS/eip-20#decimals) OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present.

*Instances (7)*:
```solidity
File: InvestmentManager.sol

26:     function decimals() external view returns (uint8);

701:         currencyDecimals = ERC20Like(LiquidityPoolLike(liquidityPool).asset()).decimals();

702:         trancheTokenDecimals = LiquidityPoolLike(liquidityPool).decimals();

```

```solidity
File: LiquidityPool.sol

279:     function decimals() public view returns (uint8) {

280:         return share.decimals();

```

```solidity
File: PoolManager.sol

243:         require(IERC20(currencyAddress).decimals() <= MAX_CURRENCY_DECIMALS, "PoolManager/too-many-currency-decimals");

```

```solidity
File: interfaces/IERC20.sol

81:     function decimals() external view returns (uint8);

```

### <a name="L-6"></a>[L-6] Unsafe ERC20 operation(s)

*Instances (11)*:
```solidity
File: InvestmentManager.sol

167:         lPool.transferFrom(user, address(escrow), _trancheTokenAmount);

313:             LiquidityPoolLike(liquidityPool).transferFrom(address(escrow), user, trancheTokenPayout),

475:             lPool.transferFrom(address(escrow), user, trancheTokenAmount),

```

```solidity
File: PoolManager.sol

133:         gateway.transfer(currency, msg.sender, recipient, amount);

249:         EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);

252:         EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);

261:         EscrowLike(escrow).approve(currencyAddress, address(this), amount);

328:         EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers

329:         EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn

```

```solidity
File: token/Tranche.sol

60:         return super.transfer(to, value);

69:         return super.transferFrom(from, to, value);

```


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 15 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (15)*:
```solidity
File: Escrow.sol

14: contract Escrow is Auth {

```

```solidity
File: InvestmentManager.sol

69: contract InvestmentManager is Auth {

```

```solidity
File: LiquidityPool.sol

52: contract LiquidityPool is Auth, IERC4626 {

```

```solidity
File: PoolManager.sol

78: contract PoolManager is Auth {

```

```solidity
File: Root.sol

15: contract Root is Auth {

```

```solidity
File: UserEscrow.sol

15: contract UserEscrow is Auth {

```

```solidity
File: admins/DelayedAdmin.sol

12: contract DelayedAdmin is Auth {

```

```solidity
File: admins/PauseAdmin.sol

10: contract PauseAdmin is Auth {

```

```solidity
File: gateway/Gateway.sol

84: contract Gateway is Auth {

```

```solidity
File: gateway/routers/axelar/Router.sol

24: contract AxelarRouter is Auth {

```

```solidity
File: gateway/routers/xcm/Router.sol

45: contract XCMRouter is Auth {

```

```solidity
File: token/RestrictionManager.sol

14: contract RestrictionManager is Auth {

```

```solidity
File: util/Auth.sol

7: contract Auth {

```

```solidity
File: util/Factory.sol

26: contract LiquidityPoolFactory is Auth {

71: contract TrancheTokenFactory is Auth {

```

