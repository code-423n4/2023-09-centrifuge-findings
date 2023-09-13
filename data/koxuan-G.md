# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Using bools for storage incurs overhead | 4 |
| [GAS-2](#GAS-2) | Cache array length outside of loop | 3 |
| [GAS-3](#GAS-3) | Use Custom Errors | 101 |
| [GAS-4](#GAS-4) | Don't initialize variables with default value | 9 |
| [GAS-5](#GAS-5) | Long revert strings | 33 |
| [GAS-6](#GAS-6) | Functions guaranteed to revert when called by normal users can be marked `payable` | 9 |
| [GAS-7](#GAS-7) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 10 |
| [GAS-8](#GAS-8) | Using `private` rather than `public` for constants, saves gas | 7 |
| [GAS-9](#GAS-9) | Splitting require() statements that use && saves gas | 6 |
| [GAS-10](#GAS-10) | Use != 0 instead of > 0 for unsigned integer comparison | 1 |
### <a name="GAS-1"></a>[GAS-1] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (4)*:
```solidity
File: PoolManager.sol

57:     mapping(address => bool) allowedCurrencies;

```

```solidity
File: Root.sol

23:     bool public paused = false;

```

```solidity
File: gateway/Gateway.sol

89:     mapping(address => bool) public incomingRouters;

```

```solidity
File: token/Tranche.sol

26:     mapping(address => bool) public liquidityPools;

```

### <a name="GAS-2"></a>[GAS-2] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (3)*:
```solidity
File: util/Factory.sol

47:         for (uint256 i = 0; i < wards.length; i++) {

103:         for (uint256 i = 0; i < trancheTokenWards.length; i++) {

117:         for (uint256 i = 0; i < restrictionManagerWards.length; i++) {

```

### <a name="GAS-3"></a>[GAS-3] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (101)*:
```solidity
File: InvestmentManager.sol

98:         require(msg.sender == address(gateway), "InvestmentManager/not-the-gateway");

106:         else revert("InvestmentManager/file-unrecognized-param");

177:         require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

190:         require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

202:         require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

213:         require(liquidityPool.checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");

229:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

242:         require(currencyPayout != 0, "InvestmentManager/zero-invest");

245:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

263:         require(trancheTokensPayout != 0, "InvestmentManager/zero-redeem");

266:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

284:         require(currencyPayout != 0, "InvestmentManager/zero-payout");

288:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

289:         require(_currency == LiquidityPoolLike(liquidityPool).asset(), "InvestmentManager/not-tranche-currency");

301:         require(trancheTokenPayout != 0, "InvestmentManager/zero-payout");

305:         require(address(liquidityPool) != address(0), "InvestmentManager/tranche-does-not-exist");

436:         require(depositPrice != 0, "LiquidityPool/deposit-token-price-0");

460:         require(depositPrice != 0, "LiquidityPool/deposit-token-price-0");

473:         require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");

502:         require(redeemPrice != 0, "LiquidityPool/redeem-token-price-0");

528:         require(redeemPrice != 0, "LiquidityPool/redeem-token-price-0");

656:         require(liquidityPool != address(0), "InvestmentManager/unknown-liquidity-pool");

668:             revert("InvestmentManager/uint128-overflow");

```

```solidity
File: LiquidityPool.sol

98:         require(msg.sender == owner, "LiquidityPool/no-approval");

105:         else revert("LiquidityPool/file-unrecognized-param");

```

```solidity
File: PoolManager.sol

115:         require(msg.sender == address(gateway), "PoolManager/not-the-gateway");

123:         else revert("PoolManager/file-unrecognized-param");

130:         require(currency != 0, "PoolManager/unknown-currency");

143:         require(address(trancheToken) != address(0), "PoolManager/unknown-token");

157:         require(address(trancheToken) != address(0), "PoolManager/unknown-token");

170:         require(pool.createdAt == 0, "PoolManager/pool-already-added");

181:         require(pool.createdAt != 0, "PoolManager/invalid-pool");

184:         require(currencyAddress != address(0), "PoolManager/unknown-currency");

200:         require(pool.createdAt != 0, "PoolManager/invalid-pool");

202:         require(tranche.createdAt == 0, "PoolManager/tranche-already-exists");

221:         require(address(trancheToken) != address(0), "PoolManager/unknown-token");

229:         require(address(trancheToken) != address(0), "PoolManager/unknown-token");

240:         require(currency != 0, "PoolManager/currency-id-has-to-be-greater-than-0");

241:         require(currencyIdToAddress[currency] == address(0), "PoolManager/currency-id-in-use");

242:         require(currencyAddressToId[currencyAddress] == 0, "PoolManager/currency-address-in-use");

243:         require(IERC20(currencyAddress).decimals() <= MAX_CURRENCY_DECIMALS, "PoolManager/too-many-currency-decimals");

259:         require(currencyAddress != address(0), "PoolManager/unknown-currency");

270:         require(address(trancheToken) != address(0), "PoolManager/unknown-token");

282:         require(tranche.token == address(0), "PoolManager/tranche-already-deployed");

283:         require(tranche.createdAt != 0, "PoolManager/tranche-not-added");

309:         require(tranche.token != address(0), "PoolManager/tranche-does-not-exist"); // Tranche must have been added

310:         require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool

313:         require(liquidityPool == address(0), "PoolManager/liquidityPool-already-deployed");

314:         require(pools[poolId].createdAt != 0, "PoolManager/pool-does-not-exist");

347:         require(currency != 0, "PoolManager/unknown-currency"); // Currency index on the Centrifuge side should start at 1

348:         require(pools[poolId].allowedCurrencies[currencyAddress], "PoolManager/pool-currency-not-allowed");

```

```solidity
File: Root.sol

45:             require(data <= MAX_DELAY, "Root/delay-too-long");

48:             revert("Root/file-unrecognized-param");

76:         require(schedule[target] != 0, "Root/target-not-scheduled");

77:         require(schedule[target] < block.timestamp, "Root/target-not-ready");

```

```solidity
File: UserEscrow.sol

37:         require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");

```

```solidity
File: admins/PauseAdmin.sol

29:         require(pausers[msg.sender] == 1, "PauseAdmin/not-authorized-to-pause");

```

```solidity
File: gateway/Gateway.sol

110:         require(msg.sender == address(investmentManager), "Gateway/only-investment-manager-allowed-to-call");

115:         require(msg.sender == address(poolManager), "Gateway/only-pool-manager-allowed-to-call");

120:         require(incomingRouters[msg.sender], "Gateway/only-router-allowed-to-call");

125:         require(!root.paused(), "Gateway/paused");

133:         else revert("Gateway/file-unrecognized-param");

364:             revert("Gateway/invalid-message");

```

```solidity
File: gateway/Messages.sol

860:         require(_bytes128.length == 128, "Input should be 128 bytes");

```

```solidity
File: gateway/routers/axelar/Router.sol

44:         require(msg.sender == address(axelarGateway), "AxelarRouter/invalid-origin");

57:         require(msg.sender == address(gateway), "AxelarRouter/only-gateway-allowed-to-call");

66:             revert("AxelarRouter/file-unrecognized-param");

```

```solidity
File: gateway/routers/xcm/Router.sol

79:         require(msg.sender == address(centrifugeChainOrigin), "XCMRouter/invalid-origin");

84:         require(msg.sender == address(gateway), "XCMRouter/only-gateway-allowed-to-call");

93:             revert("XCMRouter/file-unrecognized-param");

106:             revert("CentrifugeXCMRouter/file-unrecognized-param");

175:             revert("XCMRouter/unsupported-outgoing-message");

```

```solidity
File: token/ERC20.sol

53:         require(wards[_msgSender()] == 1, "Auth/not-authorized");

86:         else revert("ERC20/file-unrecognized-param");

92:         require(to != address(0) && to != address(this), "ERC20/invalid-address");

94:         require(balance >= value, "ERC20/insufficient-balance");

107:         require(to != address(0) && to != address(this), "ERC20/invalid-address");

109:         require(balance >= value, "ERC20/insufficient-balance");

114:                 require(allowed >= value, "ERC20/insufficient-allowance");

150:         require(allowed >= subtractedValue, "ERC20/insufficient-allowance");

163:         require(to != address(0) && to != address(this), "ERC20/invalid-address");

174:         require(balance >= value, "ERC20/insufficient-balance");

179:                 require(allowed >= value, "ERC20/insufficient-allowance");

217:         require(block.timestamp <= deadline, "ERC20/permit-expired");

218:         require(owner != address(0), "ERC20/invalid-owner");

233:         require(_isValidSignature(owner, digest, signature), "ERC20/invalid-permit");

```

```solidity
File: token/RestrictionManager.sol

46:         require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");

58:         require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");

```

```solidity
File: token/Tranche.sol

44:         else revert("TrancheToken/file-unrecognized-param");

```

```solidity
File: util/Auth.sol

27:         require(wards[msg.sender] == 1, "Auth/not-authorized");

```

```solidity
File: util/BytesLib.sol

10:         require(_length + 31 >= _length, "slice_overflow");

11:         require(_bytes.length >= _start + _length, "slice_outOfBounds");

69:         require(_bytes.length >= _start + 20, "toAddress_outOfBounds");

80:         require(_bytes.length >= _start + 1, "toUint8_outOfBounds");

91:         require(_bytes.length >= _start + 8, "toUint64_outOfBounds");

102:         require(_bytes.length >= _start + 16, "toUint128_outOfBounds");

113:         require(_bytes.length >= _start + 32, "toBytes32_outOfBounds");

124:         require(_bytes.length >= _start + 16, "toBytes16_outOfBounds");

```

```solidity
File: util/SafeTransferLib.sol

18:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-from-failed");

28:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-failed");

38:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");

```

### <a name="GAS-4"></a>[GAS-4] Don't initialize variables with default value

*Instances (9)*:
```solidity
File: gateway/Messages.sol

848:         for (uint256 i = 0; i < 128; i++) {

862:         uint8 i = 0;

869:         for (uint8 j = 0; j < i; j++) {

888:         uint8 i = 0;

```

```solidity
File: token/RestrictionManager.sol

15:     uint8 public constant SUCCESS_CODE = 0;

64:         for (uint256 i = 0; i < userLength; i++) {

```

```solidity
File: util/Factory.sol

47:         for (uint256 i = 0; i < wards.length; i++) {

103:         for (uint256 i = 0; i < trancheTokenWards.length; i++) {

117:         for (uint256 i = 0; i < restrictionManagerWards.length; i++) {

```

### <a name="GAS-5"></a>[GAS-5] Long revert strings

*Instances (33)*:
```solidity
File: InvestmentManager.sol

98:         require(msg.sender == address(gateway), "InvestmentManager/not-the-gateway");

229:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

245:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

266:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

288:         require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");

289:         require(_currency == LiquidityPoolLike(liquidityPool).asset(), "InvestmentManager/not-tranche-currency");

305:         require(address(liquidityPool) != address(0), "InvestmentManager/tranche-does-not-exist");

436:         require(depositPrice != 0, "LiquidityPool/deposit-token-price-0");

460:         require(depositPrice != 0, "LiquidityPool/deposit-token-price-0");

473:         require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");

502:         require(redeemPrice != 0, "LiquidityPool/redeem-token-price-0");

528:         require(redeemPrice != 0, "LiquidityPool/redeem-token-price-0");

656:         require(liquidityPool != address(0), "InvestmentManager/unknown-liquidity-pool");

```

```solidity
File: PoolManager.sol

202:         require(tranche.createdAt == 0, "PoolManager/tranche-already-exists");

240:         require(currency != 0, "PoolManager/currency-id-has-to-be-greater-than-0");

242:         require(currencyAddressToId[currencyAddress] == 0, "PoolManager/currency-address-in-use");

243:         require(IERC20(currencyAddress).decimals() <= MAX_CURRENCY_DECIMALS, "PoolManager/too-many-currency-decimals");

282:         require(tranche.token == address(0), "PoolManager/tranche-already-deployed");

309:         require(tranche.token != address(0), "PoolManager/tranche-does-not-exist"); // Tranche must have been added

310:         require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool

313:         require(liquidityPool == address(0), "PoolManager/liquidityPool-already-deployed");

348:         require(pools[poolId].allowedCurrencies[currencyAddress], "PoolManager/pool-currency-not-allowed");

```

```solidity
File: admins/PauseAdmin.sol

29:         require(pausers[msg.sender] == 1, "PauseAdmin/not-authorized-to-pause");

```

```solidity
File: gateway/Gateway.sol

110:         require(msg.sender == address(investmentManager), "Gateway/only-investment-manager-allowed-to-call");

115:         require(msg.sender == address(poolManager), "Gateway/only-pool-manager-allowed-to-call");

120:         require(incomingRouters[msg.sender], "Gateway/only-router-allowed-to-call");

```

```solidity
File: gateway/routers/axelar/Router.sol

57:         require(msg.sender == address(gateway), "AxelarRouter/only-gateway-allowed-to-call");

```

```solidity
File: gateway/routers/xcm/Router.sol

84:         require(msg.sender == address(gateway), "XCMRouter/only-gateway-allowed-to-call");

```

```solidity
File: token/RestrictionManager.sol

46:         require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");

58:         require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");

```

```solidity
File: util/SafeTransferLib.sol

18:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-from-failed");

28:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-failed");

38:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");

```

### <a name="GAS-6"></a>[GAS-6] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (9)*:
```solidity
File: PoolManager.sol

168:     function addPool(uint64 poolId) public onlyGateway {

179:     function allowPoolCurrency(uint64 poolId, uint128 currency) public onlyGateway {

227:     function updateMember(uint64 poolId, bytes16 trancheId, address user, uint64 validUntil) public onlyGateway {

238:     function addCurrency(uint128 currency, address currencyAddress) public onlyGateway {

257:     function handleTransfer(uint128 currency, address recipient, uint128 amount) public onlyGateway {

```

```solidity
File: gateway/Gateway.sol

285:     function handle(bytes calldata message) external onlyIncomingRouter pauseable {

```

```solidity
File: gateway/routers/axelar/Router.sol

89:     function send(bytes calldata message) public onlyGateway {

```

```solidity
File: gateway/routers/xcm/Router.sol

113:     function handle(bytes memory _message) external onlyCentrifugeChainOrigin {

118:     function send(bytes memory message) public onlyGateway {

```

### <a name="GAS-7"></a>[GAS-7] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (10)*:
```solidity
File: gateway/Messages.sol

848:         for (uint256 i = 0; i < 128; i++) {

864:             i++;

869:         for (uint8 j = 0; j < i; j++) {

890:             i++;

893:         for (i = 0; i < 32 && _bytes32[i] != 0; i++) {

```

```solidity
File: token/ERC20.sol

222:             nonce = nonces[owner]++;

```

```solidity
File: token/RestrictionManager.sol

64:         for (uint256 i = 0; i < userLength; i++) {

```

```solidity
File: util/Factory.sol

47:         for (uint256 i = 0; i < wards.length; i++) {

103:         for (uint256 i = 0; i < trancheTokenWards.length; i++) {

117:         for (uint256 i = 0; i < restrictionManagerWards.length; i++) {

```

### <a name="GAS-8"></a>[GAS-8] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (7)*:
```solidity
File: InvestmentManager.sol

74:     uint8 public constant PRICE_DECIMALS = 18;

```

```solidity
File: token/ERC20.sol

21:     string public constant version = "3";

32:     bytes32 public constant PERMIT_TYPEHASH =

```

```solidity
File: token/RestrictionManager.sol

15:     uint8 public constant SUCCESS_CODE = 0;

16:     uint8 public constant DESTINATION_NOT_A_MEMBER_RESTRICTION_CODE = 1;

17:     string public constant SUCCESS_MESSAGE = "RestrictionManager/transfer-allowed";

18:     string public constant DESTINATION_NOT_A_MEMBER_RESTRICTION_MESSAGE = "RestrictionManager/destination-not-a-member";

```

### <a name="GAS-9"></a>[GAS-9] Splitting require() statements that use && saves gas

*Instances (6)*:
```solidity
File: token/ERC20.sol

92:         require(to != address(0) && to != address(this), "ERC20/invalid-address");

107:         require(to != address(0) && to != address(this), "ERC20/invalid-address");

163:         require(to != address(0) && to != address(this), "ERC20/invalid-address");

```

```solidity
File: util/SafeTransferLib.sol

18:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-from-failed");

28:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-failed");

38:         require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");

```

### <a name="GAS-10"></a>[GAS-10] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (1)*:
```solidity
File: util/MathLib.sol

102:         if (rounding == Rounding.Up && mulmod(x, y, denominator) > 0) {

```

