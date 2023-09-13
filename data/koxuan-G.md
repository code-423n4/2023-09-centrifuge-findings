# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Cache array length outside of loop | 3 |
| [GAS-2](#GAS-2) | Don't initialize variables with default value | 9 |
| [GAS-3](#GAS-3) | Long revert strings | 33 |
| [GAS-4](#GAS-4) | Use mappings over arrays | 14 |
| [GAS-5](#GAS-5) | Use != 0 instead of > 0 for unsigned integer comparison | 1 |
### <a name="GAS-1"></a>[GAS-1] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (3)*:
```solidity
File: util/Factory.sol

47:         for (uint256 i = 0; i < wards.length; i++) {

103:         for (uint256 i = 0; i < trancheTokenWards.length; i++) {

117:         for (uint256 i = 0; i < restrictionManagerWards.length; i++) {

```

### <a name="GAS-2"></a>[GAS-2] Don't initialize variables with default value

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

### <a name="GAS-3"></a>[GAS-3] Long revert strings

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

### <a name="GAS-4"></a>[GAS-4] Use mappings over arrays
Arrays uses more gas than mappings. Consider using mappings whenever possible.

*Instances (14)*:
```solidity
File: PoolManager.sol

285:         address[] memory trancheTokenWards = new address[](2);

285:         address[] memory trancheTokenWards = new address[](2);

289:         address[] memory memberlistWards = new address[](1);

289:         address[] memory memberlistWards = new address[](1);

316:         address[] memory liquidityPoolWards = new address[](1);

316:         address[] memory liquidityPoolWards = new address[](1);

```

```solidity
File: token/RestrictionManager.sol

62:     function updateMembers(address[] memory users, uint256 validUntil) public auth {

```

```solidity
File: util/Factory.sol

20:         address[] calldata wards

42:         address[] calldata wards

62:         address[] calldata trancheTokenWards,

63:         address[] calldata restrictionManagerWards

87:         address[] calldata trancheTokenWards,

88:         address[] calldata restrictionManagerWards

111:     function _newRestrictionManager(address[] calldata restrictionManagerWards) internal returns (address memberList) {

```

### <a name="GAS-5"></a>[GAS-5] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (1)*:
```solidity
File: util/MathLib.sol

102:         if (rounding == Rounding.Up && mulmod(x, y, denominator) > 0) {

```

