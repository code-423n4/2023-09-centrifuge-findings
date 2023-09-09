## [G-01] Don't Initialize Variables with Default Value
Description
Uninitialized variables are assigned with the types default value.
Explicitly initializing a variable with it's default value costs unnecessary gas.
```txt
2023-09-centrifuge/src/gateway/Messages.sol::848 => for (uint256 i = 0; i < 128; i++) {
2023-09-centrifuge/src/gateway/Messages.sol::862 => uint8 i = 0;
2023-09-centrifuge/src/gateway/Messages.sol::869 => for (uint8 j = 0; j < i; j++) {
2023-09-centrifuge/src/gateway/Messages.sol::888 => uint8 i = 0;
2023-09-centrifuge/src/token/RestrictionManager.sol::64 => for (uint256 i = 0; i < userLength; i++) {
2023-09-centrifuge/src/util/Factory.sol::47 => for (uint256 i = 0; i < wards.length; i++) {
2023-09-centrifuge/src/util/Factory.sol::103 => for (uint256 i = 0; i < trancheTokenWards.length; i++) {
2023-09-centrifuge/src/util/Factory.sol::117 => for (uint256 i = 0; i < restrictionManagerWards.length; i++) {
```
## [G-02] Cache Array Length Outside of Loop
Description
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.
```txt
2023-09-centrifuge/src/gateway/Messages.sol::849 => if (i < temp.length) {
2023-09-centrifuge/src/gateway/Messages.sol::860 => require(_bytes128.length == 128, "Input should be 128 bytes");
2023-09-centrifuge/src/gateway/Messages.sol::878 => if (tempEmptyStringTest.length == 0) {
2023-09-centrifuge/src/gateway/routers/xcm/Router.sol::147 => // We need to specify the length of the message in the scale-encoding format
2023-09-centrifuge/src/gateway/routers/xcm/Router.sol::154 => // Obtain the Scale-encoded length of a given message. Each Liquidity Pools Message is fixed-sized and
2023-09-centrifuge/src/gateway/routers/xcm/Router.sol::155 => // have thus a fixed scale-encoded length associated to which message variant (aka Call).
2023-09-centrifuge/src/token/ERC20.sol::197 => if (signature.length == 65) {
2023-09-centrifuge/src/token/ERC20.sol::213 => return (success && result.length == 32 && abi.decode(result, (bytes4)) == IERC1271.isValidSignature.selector);
2023-09-centrifuge/src/token/RestrictionManager.sol::63 => uint256 userLength = users.length;
2023-09-centrifuge/src/token/Tranche.sol::101 => ///         a call is not performed by the trusted forwarder or the calldata length is less than
2023-09-centrifuge/src/token/Tranche.sol::102 => ///         20 bytes (an address length).
2023-09-centrifuge/src/token/Tranche.sol::104 => if (isTrustedForwarder(msg.sender) && msg.data.length >= 20) {
2023-09-centrifuge/src/util/Factory.sol::47 => for (uint256 i = 0; i < wards.length; i++) {
2023-09-centrifuge/src/util/Factory.sol::103 => for (uint256 i = 0; i < trancheTokenWards.length; i++) {
2023-09-centrifuge/src/util/Factory.sol::117 => for (uint256 i = 0; i < restrictionManagerWards.length; i++) {
2023-09-centrifuge/src/util/SafeTransferLib.sol::18 => require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-from-failed");
2023-09-centrifuge/src/util/SafeTransferLib.sol::28 => require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-failed");
2023-09-centrifuge/src/util/SafeTransferLib.sol::38 => require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");
```
## [G-03] Use immutable for OpenZeppelin AccessControl's Roles Declarations
Description
⚡️ Only valid for solidity versions <0.6.12 ⚡️
Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.
Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.
```txt
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::46 => keccak256(bytes(axelarCentrifugeChainId)) == keccak256(bytes(sourceChain)),
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::50 => keccak256(bytes(axelarCentrifugeChainAddress)) == keccak256(bytes(sourceAddress)),
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::79 => bytes32 payloadHash = keccak256(payload);
2023-09-centrifuge/src/token/ERC20.sol::33 => keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
2023-09-centrifuge/src/token/ERC20.sol::68 => return keccak256(
2023-09-centrifuge/src/token/ERC20.sol::70 => keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
2023-09-centrifuge/src/token/ERC20.sol::71 => keccak256(bytes(name)),
2023-09-centrifuge/src/token/ERC20.sol::72 => keccak256(bytes(version)),
2023-09-centrifuge/src/token/ERC20.sol::225 => bytes32 digest = keccak256(
2023-09-centrifuge/src/token/ERC20.sol::229 => keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonce, deadline))
2023-09-centrifuge/src/util/Factory.sol::94 => bytes32 salt = keccak256(abi.encodePacked(poolId, trancheId));
```
## [G-04] Long Revert Strings
Description
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.
If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.
```txt
2023-09-centrifuge/src/InvestmentManager.sol::98 => require(msg.sender == address(gateway), "InvestmentManager/not-the-gateway");
2023-09-centrifuge/src/InvestmentManager.sol::106 => else revert("InvestmentManager/file-unrecognized-param");
2023-09-centrifuge/src/InvestmentManager.sol::229 => require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");
2023-09-centrifuge/src/InvestmentManager.sol::245 => require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");
2023-09-centrifuge/src/InvestmentManager.sol::266 => require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");
2023-09-centrifuge/src/InvestmentManager.sol::288 => require(liquidityPool != address(0), "InvestmentManager/tranche-does-not-exist");
2023-09-centrifuge/src/InvestmentManager.sol::289 => require(_currency == LiquidityPoolLike(liquidityPool).asset(), "InvestmentManager/not-tranche-currency");
2023-09-centrifuge/src/InvestmentManager.sol::305 => require(address(liquidityPool) != address(0), "InvestmentManager/tranche-does-not-exist");
2023-09-centrifuge/src/InvestmentManager.sol::314 => "InvestmentManager/trancheTokens-transfer-failed"
2023-09-centrifuge/src/InvestmentManager.sol::432 => "InvestmentManager/amount-exceeds-deposit-limits"
2023-09-centrifuge/src/InvestmentManager.sol::436 => require(depositPrice != 0, "LiquidityPool/deposit-token-price-0");
2023-09-centrifuge/src/InvestmentManager.sol::456 => "InvestmentManager/amount-exceeds-mint-limits"
2023-09-centrifuge/src/InvestmentManager.sol::460 => require(depositPrice != 0, "LiquidityPool/deposit-token-price-0");
2023-09-centrifuge/src/InvestmentManager.sol::473 => require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");
2023-09-centrifuge/src/InvestmentManager.sol::476 => "InvestmentManager/trancheTokens-transfer-failed"
2023-09-centrifuge/src/InvestmentManager.sol::498 => "InvestmentManager/amount-exceeds-redeem-limits"
2023-09-centrifuge/src/InvestmentManager.sol::502 => require(redeemPrice != 0, "LiquidityPool/redeem-token-price-0");
2023-09-centrifuge/src/InvestmentManager.sol::524 => "InvestmentManager/amount-exceeds-withdraw-limits"
2023-09-centrifuge/src/InvestmentManager.sol::528 => require(redeemPrice != 0, "LiquidityPool/redeem-token-price-0");
2023-09-centrifuge/src/InvestmentManager.sol::656 => require(liquidityPool != address(0), "InvestmentManager/unknown-liquidity-pool");
2023-09-centrifuge/src/InvestmentManager.sol::668 => revert("InvestmentManager/uint128-overflow");
2023-09-centrifuge/src/LiquidityPool.sol::105 => else revert("LiquidityPool/file-unrecognized-param");
2023-09-centrifuge/src/LiquidityPool.sol::149 => // require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");
2023-09-centrifuge/src/PoolManager.sol::123 => else revert("PoolManager/file-unrecognized-param");
2023-09-centrifuge/src/PoolManager.sol::202 => require(tranche.createdAt == 0, "PoolManager/tranche-already-exists");
2023-09-centrifuge/src/PoolManager.sol::240 => require(currency != 0, "PoolManager/currency-id-has-to-be-greater-than-0");
2023-09-centrifuge/src/PoolManager.sol::242 => require(currencyAddressToId[currencyAddress] == 0, "PoolManager/currency-address-in-use");
2023-09-centrifuge/src/PoolManager.sol::243 => require(IERC20(currencyAddress).decimals() <= MAX_CURRENCY_DECIMALS, "PoolManager/too-many-currency-decimals");
2023-09-centrifuge/src/PoolManager.sol::282 => require(tranche.token == address(0), "PoolManager/tranche-already-deployed");
2023-09-centrifuge/src/PoolManager.sol::309 => require(tranche.token != address(0), "PoolManager/tranche-does-not-exist"); // Tranche must have been added
2023-09-centrifuge/src/PoolManager.sol::310 => require(isAllowedAsPoolCurrency(poolId, currency), "PoolManager/currency-not-supported"); // Currency must be supported by pool
2023-09-centrifuge/src/PoolManager.sol::313 => require(liquidityPool == address(0), "PoolManager/liquidityPool-already-deployed");
2023-09-centrifuge/src/PoolManager.sol::348 => require(pools[poolId].allowedCurrencies[currencyAddress], "PoolManager/pool-currency-not-allowed");
2023-09-centrifuge/src/UserEscrow.sol::44 => "UserEscrow/receiver-has-no-allowance"
2023-09-centrifuge/src/admins/PauseAdmin.sol::29 => require(pausers[msg.sender] == 1, "PauseAdmin/not-authorized-to-pause");
2023-09-centrifuge/src/gateway/Gateway.sol::110 => require(msg.sender == address(investmentManager), "Gateway/only-investment-manager-allowed-to-call");
2023-09-centrifuge/src/gateway/Gateway.sol::115 => require(msg.sender == address(poolManager), "Gateway/only-pool-manager-allowed-to-call");
2023-09-centrifuge/src/gateway/Gateway.sol::120 => require(incomingRouters[msg.sender], "Gateway/only-router-allowed-to-call");
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::26 => string private constant axelarCentrifugeChainAddress = "0x7369626cef070000000000000000000000000000";
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::27 => string private constant centrifugeGatewayPrecompileAddress = "0x0000000000000000000000000000000000002048";
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::47 => "AxelarRouter/invalid-source-chain"
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::51 => "AxelarRouter/invalid-source-address"
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::57 => require(msg.sender == address(gateway), "AxelarRouter/only-gateway-allowed-to-call");
2023-09-centrifuge/src/gateway/routers/axelar/Router.sol::66 => revert("AxelarRouter/file-unrecognized-param");
2023-09-centrifuge/src/gateway/routers/xcm/Router.sol::84 => require(msg.sender == address(gateway), "XCMRouter/only-gateway-allowed-to-call");
2023-09-centrifuge/src/gateway/routers/xcm/Router.sol::93 => revert("XCMRouter/file-unrecognized-param");
2023-09-centrifuge/src/gateway/routers/xcm/Router.sol::106 => revert("CentrifugeXCMRouter/file-unrecognized-param");
2023-09-centrifuge/src/gateway/routers/xcm/Router.sol::175 => revert("XCMRouter/unsupported-outgoing-message");
2023-09-centrifuge/src/token/ERC20.sol::33 => keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
2023-09-centrifuge/src/token/ERC20.sol::70 => keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
2023-09-centrifuge/src/token/RestrictionManager.sol::17 => string public constant SUCCESS_MESSAGE = "RestrictionManager/transfer-allowed";
2023-09-centrifuge/src/token/RestrictionManager.sol::18 => string public constant DESTINATION_NOT_A_MEMBER_RESTRICTION_MESSAGE = "RestrictionManager/destination-not-a-member";
2023-09-centrifuge/src/token/RestrictionManager.sol::46 => require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");
2023-09-centrifuge/src/token/RestrictionManager.sol::58 => require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");
2023-09-centrifuge/src/token/Tranche.sol::44 => else revert("TrancheToken/file-unrecognized-param");
2023-09-centrifuge/src/util/SafeTransferLib.sol::18 => require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-from-failed");
2023-09-centrifuge/src/util/SafeTransferLib.sol::28 => require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-failed");
2023-09-centrifuge/src/util/SafeTransferLib.sol::38 => require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");
```
## [G-05] Use Shift Right/Left instead of Division/Multiplication if possible
Description
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.
While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.
```txt
2023-09-centrifuge/src/gateway/Messages.sol::15 => /// 2 - Add Pool
2023-09-centrifuge/src/gateway/Messages.sol::19 => /// 4 - Add a Pool's Tranche Token
2023-09-centrifuge/src/gateway/Messages.sol::27 => /// 8 - A transfer of Tranche tokens
2023-09-centrifuge/src/gateway/Messages.sol::51 => /// 20 - Cancel a redeem order
2023-09-centrifuge/src/gateway/Messages.sol::53 => /// 21 - Schedule an upgrade contract to be granted admin rights
2023-09-centrifuge/src/gateway/Messages.sol::55 => /// 22 - Cancel a previously scheduled upgrade
2023-09-centrifuge/src/gateway/Messages.sol::57 => /// 23 - Update tranche token metadata
2023-09-centrifuge/src/gateway/Messages.sol::59 => /// 24 - Update tranche investment limit
2023-09-centrifuge/src/gateway/Messages.sol::136 => * 25-152: tokenName (string = 128 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::193 => * 25-45: user (Ethereum address, 20 bytes - Skip 12 bytes from 32-byte addresses)
2023-09-centrifuge/src/gateway/Messages.sol::234 => * 25-40: currency (uint128 = 16 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::235 => * 41-56: price (uint128 = 16 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::266 => * 49-80: receiver address (32 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::267 => * 81-96: amount (uint128 = 16 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::310 => * 25-56: sender (bytes32)
2023-09-centrifuge/src/gateway/Messages.sol::370 => * 25-56: investor address (32 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::406 => * 25-56: investor address (32 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::438 => * 25-56: investor address (32 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::470 => * 25-56: investor address (32 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::502 => * 25-56: investor address (32 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::533 => * 25-56: investor address (32 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::730 => * 25-152: tokenName (string = 128 bytes)
2023-09-centrifuge/src/gateway/Messages.sol::810 => * 25-40: investmentLimit (uint128 = 16 bytes)
```