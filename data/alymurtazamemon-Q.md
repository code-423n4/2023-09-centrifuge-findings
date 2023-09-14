# Centrifuge - Quality Assurence Report

# Low Risk Findings

## Table Of Content

| Number                                                                                                           | Issue                                                                                                | Instances |
| :--------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------- | --------: |
| [L-01](#l-01---missing-validation-for-address0-in-the-constructors-or-in-the-functions-changing-state-variables) | Missing validation for `address(0)` in the constructors or in the functions changing state variables |        13 |
| [L-02](#l-02---file-functions-should-be-called-on-deployment-inside-the-constructors)                            | `file` functions should be called on deployment inside the `constructors`                            |         3 |
| [L-03](#l-03---unsafe-low-level-call)                                                                            | Unsafe `low-level` call                                                                              |         5 |
| [L-04](#l-04---shadow-variables)                                                                                 | Shadow Variables                                                                                     |         1 |

### [L-01] - Missing validation for `address(0)` in the constructors or in the functions changing state variables.

**Description**

It is recommended to check for `address(0)` when updating state variables through constructors or functions.

This can save the protocol from accidental updates, transfering funds to `address(0)`, etc.

`Example`

```diff
function approve(address token, address spender, uint256 value) external auth {
+   require(token != address(0) && spender != address(0), "ContractName__InvalidAddress");
    SafeTransferLib.safeApprove(token, spender, value);
    emit Approve(token, spender, value);
}
```

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Escrow.sol - Line 24](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Escrow.sol#L24)

[InvestmentManager.sol - Line 89 - 90](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L89-L90)

[LiquidityPool.sol - Lines 88 - 90](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L88-L90)

[LiquidityPool.sol - Line 104](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L104)

[PoolManager.sol - Lines 105 - 107](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L105-L107)

[Root.sol - Line 35 - 36](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L35-L36)

[Root.sol - Line 66](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L66)

[Root.sol - Line 91](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L91)

[Root.sol - Line 99](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L99)

[UserEscrow.sol - Line 32](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L32)

[UserEscrow.sol - Line 48](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L48)

[DelayedAdmin.sol - Line 19](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/DelayedAdmin.sol#L19)

[PauseAdmin.sol - Line 22](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/PauseAdmin.sol#L22)

</details>

**Recommended Mitigation Steps**

Make sure to add the `address(0)` validations for all the above-mentioned addresses.

### [L-02] - `file` functions should be called on deployment inside the `constructors`.

**Description**

The contracts are using the `file` method to update the key parameters and initiate the instances, but these function needs to be called atleast one time after the deployment of the contracts.

This increases the risk factor, there are many contract and if for any contract deployer forgot to call the `file` function then the contracts can behave differently.

Although, these function can be called later on by `wards` to mitigate this issue but before this step it can affect the users.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[InvestmentManager.sol - Line 103](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L103)

[PoolManager.sol - Line 120](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L120)

[Tranche.sol - Line 42](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol#L42)

</details>

**Recommended Mitigation Steps**

The `file` should be called inside the `constructor` on initial deployment.

### [L-03] - Unsafe `low-level` call.

**Description**

The `transferFrom`, `transfer`, `approve`, `mint` and `burn` function of `LiquidityPool` contract uses the low level call and check the `success` status, this status will be passed if the contract does not exist on the provided address.

Solidity docs warn about using low-level calls directly due to security reasons;

-   You should avoid using `.call()` whenever possible when executing another contract function as it bypasses type checking, function existence check, and argument packing.

-   Due to the fact that the EVM considers a call to a non-existing contract to always succeed, Solidity includes an extra check using the `extcodesize` opcode when performing external calls. This ensures that the contract that is about to be called either actually exists (it contains code) or an exception is raised. The low-level calls which operate on addresses rather than contract instances (i.e. `.call()`, `.delegatecall()`, `.staticcall()`, `.send()` and `.transfer()`) **do not** include this check, which makes them cheaper in terms of gas but also less safe.

<details>
  <summary>Findings Locations</summary>
  <p></p>

[LiquidityPool.sol - Line 296](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L296)

[LiquidityPool.sol - Line 302](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L302)

[LiquidityPool.sol - Line 308](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L308)

[LiquidityPool.sol - Line 314](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L314)

[LiquidityPool.sol - Line 319](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L319)

</details>

**Recommended Mitigation Steps**

Use the Openzeppelin [Address](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol) library to perform the external call.

### [L-04] - Shadow Variables.

**Description**

```solidity
function newLiquidityPool(
    uint64 poolId,
    bytes16 trancheId,
    address currency,
    address trancheToken,
    address investmentManager,
    address[] calldata wards
) public auth returns (address) {
    LiquidityPool liquidityPool = new LiquidityPool(poolId, trancheId, currency, trancheToken, investmentManager);

    liquidityPool.rely(root);

    for (uint256 i = 0; i < wards.length; i++) {
        liquidityPool.rely(wards[i]);
    }

    liquidityPool.deny(address(this));
    return address(liquidityPool);
}
```

The `newLiquidityPool` function of `LiquidityPoolFactory` contract has the parameter `wards` which shadows the existing `wards` mappings variable of `Auth` contract.

Due to the shadow naming the interpreter can consider the `wards` of the `Auth` contract which will affect the contract execution.

We should avoid naming shadowing to prevent unexpected results.

<details>
  <summary>Findings Locations</summary>
  <p></p>

```diff
-   address[] calldata wards
+   address[] calldata _wards
-   for (uint256 i = 0; i < wards.length; i++) {
-       liquidityPool.rely(wards[i]);
+   for (uint256 i = 0; i < _wards.length; i++) {
+       liquidityPool.rely(_wards[i]);
```

[Factory.sol - Line 47 - 48](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L47-L48)

</details>

**Recommended Mitigation Steps**

Consider the compiler warnings into consideration and avoid variable shadowing.

# Informational Findings

## Table Of Content

| Number                                                                                                                                        | Issue                                                                                                                              | Instances |
| :-------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------- | --------: |
| [I-01](#i-01---use-natspec-comments-for-smart-contracts-interfaces-and-libraries)                                                             | Use `natspec` comments for smart contracts, interfaces and libraries                                                               |        18 |
| [I-02](#i-02---use-natspec-comments-for-events-state-variables-constructors-functions-and-all-other-things-which-will-be-included-in-the-abi) | Use `natspec` comments for events, state variables, constructors, functions and all other things which will be included in the ABI |        20 |
| [I-03](#i-03---set-the-internal-layout-of-contracts-interfaces-and-libraries)                                                                 | Set the Internal Layout of `Contracts`, `Interfaces`, and `Libraries`                                                              |         3 |
| [I-04](#i-04---investmentmanageramount-exceeds-mint-limits-error-message-is-confusing)                                                        | `InvestmentManager/amount-exceeds-mint-limits` error message is confusing                                                          |         1 |
| [I-05](#i-05---investmentmanageramount-exceeds-withdraw-limits-error-message-is-confusing)                                                    | `InvestmentManager/amount-exceeds-withdraw-limits` error message is confusing                                                      |         1 |
| [I-06](#i-06---function-state-mutability-can-be-restricted-to-view)                                                                           | Function state mutability can be restricted to `view`                                                                              |         1 |
| [I-07](#i-07---unused-functions-parameters-can-be-silent-using--syntax-or-should-be-removed)                                                  | Unused function's parameters can be silent using `/**/` syntax or should be removed                                                |         3 |
| [I-08](#i-08---function-state-mutability-can-be-restricted-to-pure)                                                                           | Function state mutability can be restricted to `pure`                                                                              |         3 |
| [I-09](#i-09---commented-should-be-removed-from-the-production-code)                                                                          | `commented` should be removed from the production code                                                                             |         2 |

### [I-01] - Use `natspec` comments for smart contracts, interfaces and libraries.

**Description**

It is recommended by solidity documentation that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Escrow.sol - Line 07](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Escrow.sol#L7)

[InvestmentManager.sol - Line 08](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L8)

[InvestmentManager.sol - Line 23](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L23)

[InvestmentManager.sol - Line 31](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L31)

[InvestmentManager.sol - Line 41](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L41)

[InvestmentManager.sol - Line 49](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L49)

[InvestmentManager.sol - Line 53](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L53)

[LiquidityPool.sol - Line 09](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L9)

[LiquidityPool.sol - Line 16](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L16)

[LiquidityPool.sol - Line 20](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L20)

[PoolManager.sol - Line 11](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L11)

[PoolManager.sol - Line 30](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L30)

[PoolManager.sol - Line 34](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L34)

[PoolManager.sol - Line 40](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L40)

[PoolManager.sol - Line 44](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L44)

[PoolManager.sol - Line 48](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L48)

[Root.sol - Line 06](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L6)

[UserEscrow.sol - Line 07](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L7)

</details>

**Recommended Mitigation Steps**

Make sure to include the natspec comments for interfaces, libraries and smart contracts mentioned above.

### [I-02] - Use `natspec` comments for events, state variables, constructors, functions and all other things which will be included in the ABI.

**Description**

It is recommended by the solidity documentation that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Escrow.sol - Line 08](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Escrow.sol#L8)

[InvestmentManager.sol - Lines 09 - 20](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L9-L20)

[InvestmentManager.sol - Lines 24 - 28](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L24-L28)

[InvestmentManager.sol - Lines 32 - 38](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L32-L38)

[InvestmentManager.sol - Lines 42 - 46](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L42-L46)

[InvestmentManager.sol - Line 50](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L50)

[InvestmentManager.sol - Lines 54 - 55](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L54-L55)

[InvestmentManager.sol - Lines 70 - 662](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L70-L662)

[LiquidityPool.sol - Line 10 - 13](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L10-L13)

[LiquidityPool.sol - Line 17](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L17)

[LiquidityPool.sol - Lines 21 - 41](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L21-L41)

[PoolManager.sol - Lines 12 - 27](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L12-L27)

[PoolManager.sol - Line 31](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L31)

[PoolManager.sol - Line 35 - 37](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L35-L37)

[PoolManager.sol - Line 41](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L41)

[PoolManager.sol - Line 45](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L45)

[PoolManager.sol - Line 49](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L49)

[Root.sol - Lines 07 - 08](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L7-L8)

[UserEscrow.sol - Line 08](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L8)

[PauseAdmin.sol - Lines 11 - 47](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/PauseAdmin.sol#L11-L47)

</details>

**Recommended Mitigation Steps**

Make sure to include the natspec comments for all public events, state variables, constructors, functions, etc mentioned above.

### [I-03] - Set the Internal Layout of `Contracts`, `Interfaces`, and `Libraries`.

**Description**

**Description**

According to the Solidity [Style Guide](https://docs.soliditylang.org/en/v0.8.21/style-guide.html#style-guide) consistency in the projects layout is very important. If all projects apply the style guides provided by solidity docs then understanding the projects and its code will be much easier for developers, and auditors.

Solidity docs;

> A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is most important.

According to the solidity styles guide;

Inside each contract, library or interface, this order should be followed:

1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

This should be the order of functions:

-   constructor
-   receive function (if exists)
-   fallback function (if exists)
-   external
-   public
-   internal
-   private
-   View and pure functions last

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[InvestmentManager.sol - Line 70 - 703](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L70-L703)

[LiquidityPool.sol - Lines 21 - 41](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L21-L41)

[LiquidityPool.sol - Lines 53 - 347](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L53-L347)

</details>

**Recommended Mitigation Steps**

Apply the recommended layout structure in all mentioned contracts according to the solidity styles guide.

### [I-04] - `InvestmentManager/amount-exceeds-mint-limits` error message is confusing.

**Description**

```solidity
require(
    (_trancheTokenAmount <= orderbook[user][liquidityPool].maxMint && _trancheTokenAmount != 0),
    "InvestmentManager/amount-exceeds-mint-limits"
);
```

The `processMint` function of `InvestmentManager` contract revert with `"InvestmentManager/amount-exceeds-mint-limits"` when the `_trancheTokenAmount` will be greater than `maxMint` value or when it will be `0`.

This is confusing before `0` value cannot exceed the `maxMint` because it's minimum range also is `0`.

[InvestmentManager.sol - Line 456](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L456)

**Recommended Mitigation Steps**

The error message should be `InvestmentManager/amount-exceeds-mint-limits-or-zero`.

```diff
require(
    (_trancheTokenAmount <= orderbook[user][liquidityPool].maxMint && _trancheTokenAmount != 0),
-    "InvestmentManager/amount-exceeds-mint-limits"
+    "InvestmentManager/amount-exceeds-mint-limits-or-zero"
);
```

### [I-05] - `InvestmentManager/amount-exceeds-withdraw-limits` error message is confusing.

**Description**

```solidity
require(
    (_currencyAmount <= orderbook[user][liquidityPool].maxWithdraw && _currencyAmount != 0),
    "InvestmentManager/amount-exceeds-withdraw-limits"
);
```

The `processWithdraw` function of `InvestmentManager` contract revert with `InvestmentManager/amount-exceeds-withdraw-limits` when the `_currencyAmount` will be greater than `maxWithdraw` value or when it will be `0`.

This is confusing before `0` value cannot exceed the `maxWithdraw` because it's minimum range also is `0`.

[InvestmentManager.sol - Line 524](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L524)

**Recommended Mitigation Steps**

The error message should be `InvestmentManager/amount-exceeds-withdraw-limits-or-zero`.

```diff
require(
    (_currencyAmount <= orderbook[user][liquidityPool].maxWithdraw && _currencyAmount != 0),
-    "InvestmentManager/amount-exceeds-withdraw-limits"
+    "InvestmentManager/amount-exceeds-withdraw-limits-or-zero"
);
```

### [I-06] - Function state mutability can be restricted to `view`.

**Description**

```solidity
function _isAllowedToInvest(uint64 poolId, bytes16 trancheId, address currency, address user) internal returns (bool) {
    address liquidityPool = poolManager.getLiquidityPool(poolId, trancheId, currency);

    require(liquidityPool != address(0), "InvestmentManager/unknown-liquidity-pool");
    require(LiquidityPoolLike(liquidityPool).checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");
    return true;
}
```

The `_isAllowedToInvest` of `InvestmentManager` contract only read from the contract, checks some conditions and return the `true` value. It is updating any state of the contract and can be marked as a view function.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

```diff
-   function _isAllowedToInvest(uint64 poolId, bytes16 trancheId, address currency, address user) internal returns (bool) {
+   function _isAllowedToInvest(uint64 poolId, bytes16 trancheId, address currency, address user) internal view returns (bool) {
    address liquidityPool = poolManager.getLiquidityPool(poolId, trancheId, currency);

    require(liquidityPool != address(0), "InvestmentManager/unknown-liquidity-pool");
    require(LiquidityPoolLike(liquidityPool).checkTransferRestriction(address(0), user, 0), "InvestmentManager/not-a-member");
    return true;
}
```

[InvestmentManager.sol - Line 651](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L651)

</details>

**Recommended Mitigation Steps**

Mark the above mentioned functions as view functions.

### [I-07] - Unused function's parameters can be silent using `/**/` syntax or should be removed.

**Description**

```solidity
function _toPriceDecimals(uint128 _value, uint8 decimals, address liquidityPool) internal view returns (uint256 value) {
    if (PRICE_DECIMALS == decimals) return uint256(_value);

    value = uint256(_value) * 10 ** (PRICE_DECIMALS - decimals);
}
```

1. The `_toPriceDecimals` function of `InvestmentManager` contract has the `liquidityPool` address parameter but this function is not using this parameter at all. We should either remove it or should silent the compiler warning using the `/**/` syntax.

2. The `_fromPriceDecimals` function of `InvestmentManager` contract has the same unused parameter `liquidityPool`.

3. The `detectTransferRestriction` function of `RestrictionManager` contract also has unused parameters `from` and `value` (find the link below).

<details>
  <summary>Findings Locations</summary>
  <p></p>

`Example`

```diff
function _toPriceDecimals(uint128 _value, uint8 decimals, address /* liquidityPool */) internal view returns (uint256 value) {
    if (PRICE_DECIMALS == decimals) return uint256(_value);

    value = uint256(_value) * 10 ** (PRICE_DECIMALS - decimals);
}
```

`Locations`

[InvestmentManager.sol - Line 676](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L676)

[InvestmentManager.sol - Line 686](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L686)

[RestrictionManager.sol - Line 28](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L28)

</details>

**Recommended Mitigation Steps**

Make sure to either remove the above mentioned parameters or should remove them.

### [I-08] - Function state mutability can be restricted to `pure`.

**Description**

```solidity
function _toPriceDecimals(uint128 _value, uint8 decimals, address /* liquidityPool */) internal view returns (uint256 value) {
    if (PRICE_DECIMALS == decimals) return uint256(_value);

    value = uint256(_value) * 10 ** (PRICE_DECIMALS - decimals);
}
```

```solidity
function _fromPriceDecimals(uint256 _value, uint8 decimals, address /* liquidityPool */) internal view returns (uint128 value) {
    if (PRICE_DECIMALS == decimals) return _toUint128(_value);
    value = _toUint128(_value / 10 ** (PRICE_DECIMALS - decimals));
}
```

1. The `_toPriceDecimals` and `_fromPriceDecimals` functions of `InvestmentManager` contract only perform the calculation and do not read any state variables. We should mark these type of functions' visibility as `pure`.

```solidity
function messageForTransferRestriction(uint8 restrictionCode) public view returns (string memory) {
    if (restrictionCode == DESTINATION_NOT_A_MEMBER_RESTRICTION_CODE) {
        return DESTINATION_NOT_A_MEMBER_RESTRICTION_MESSAGE;
    }

    return SUCCESS_MESSAGE;
}
```

2. The `messageForTransferRestriction` function of `RestrictionManager` also just check the `restrictionCode` and read only the constant state variables. And reading constant variables is not same as read regular state variables because all the places get replaced by constant values of these variables on deployment.

<details>
  <summary>Findings Locations</summary>
  <p></p>

`Example`

```diff
-   function _toPriceDecimals(uint128 _value, uint8 decimals, address /* liquidityPool */) internal view returns (uint256 value) {
+   function _toPriceDecimals(uint128 _value, uint8 decimals, address /* liquidityPool */) internal pure returns (uint256 value) {
    if (PRICE_DECIMALS == decimals) return uint256(_value);

    value = uint256(_value) * 10 ** (PRICE_DECIMALS - decimals);
}
```

`Locations`

[InvestmentManager.sol - Line 676](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L676)

[InvestmentManager.sol - Line 686](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L686)

[RestrictionManager.sol - Line 36](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L36)

</details>

**Recommended Mitigation Steps**

Mark the above mentioned functions as pure functions.

### [I-09] - `commented` should be removed from the production code.

**Description**

```solidity
/// @notice Collect shares for deposited assets after Centrifuge epoch execution.
///         maxMint is the max amount of shares that can be collected.
function mint(uint256 shares, address receiver) public returns (uint256 assets) {
    // require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");
    assets = investmentManager.processMint(receiver, shares);

    emit Deposit(address(this), receiver, assets, shares);
}
```

1. The `mint` function of `LiquidityPool` contract contains the comment require statement, it should be removed if it is no longer in use.

2. The `formatAddTranche` function of `Message` contract contains a TODO comment, it should be removed if it is completed.

<details>
  <summary>Findings Locations</summary>
  <p></p>

```diff
-   // require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");
```

[LiquidityPool.sol - Line 149](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L149)

[Messages.sol - Line 149](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L149)

</details>

**Recommended Mitigation Steps**

Remove the above-mentioned commented code to clean the production-ready code.