# [G-01] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost


```
File: src/LiquidityPool.sol

214: function requestDeposit(uint256 assets, address owner) public withApproval(owner) {

231: function requestRedeem(uint256 shares, address owner) public withApproval(owner) {

247: function decreaseDepositRequest(uint256 assets, address owner) public withApproval(owner) {

253: function decreaseRedeemRequest(uint256 shares, address owner) public withApproval(owner) {
```
GitHub: [214](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L214C1-L214C1), [231](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L231), [247](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L247), [253](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L253)

```
File: src/admins/PauseAdmin.sol

45: function pause() public canPause {
```
GitHub: [45](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L45)

# [G-02] Usage of smaller uint/int types causes overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

```
File: src/LiquidityPool.sol

72:  uint128 public latestPrice;

```
GitHub: [72](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L72)

```
File: src/InvestmentManager.sol

74:  uint8 public constant PRICE_DECIMALS = 18;

```

GitHub: [74](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L74C4-L74C47)

```
File: src/PoolManager.sol

79: uint8 internal constant MAX_CURRENCY_DECIMALS = 18;

```
GitHub: [79](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L79C1-L79C1)

```
File: src/token/ERC20.sol

22: uint8 public immutable decimals;

```
GitHub: [22](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L22)

```
File: src/token/RestrictionManager.sol

15: uint8 public constant SUCCESS_CODE = 0;
16: uint8 public constant DESTINATION_NOT_A_MEMBER_RESTRICTION_CODE = 1;

```
GitHub: [15](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L15), [16](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L16C5-L16C73)

# [G-03] Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

```
File:src/PoolManager.sol

79: uint8 internal constant MAX_CURRENCY_DECIMALS = 18;

```
GitHub: [79](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L79)

# [G-04] Use custom errors rather than revert()/require() strings to save gas

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they're hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

```
File: src/LiquidityPool.sol

105:  else revert("LiquidityPool/file-unrecognized-param");

```
GitHub: [105](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L105)

```
File: src/InvestmentManager.sol

106: else revert("InvestmentManager/file-unrecognized-param");

668:  revert("InvestmentManager/uint128-overflow");

```
GitHub: [106](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L106), [668](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L668C25-L668C57)

```
File: src/PoolManager.sol

123: else revert("PoolManager/file-unrecognized-param");

```
GitHub: [123](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L123)

```
File: src/Root.sol

48:   revert("Root/file-unrecognized-param");

```
GitHub: [48](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L48)

```
File: src/token/Tranche.sol

44:  else revert("TrancheToken/file-unrecognized-param");

```
GitHub: [44](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L44)

```
File: src/token/ERC20.sol

86:  else revert("ERC20/file-unrecognized-param");

```
GitHub: [86](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L86)

```
File: src/gateway/Gateway.sol

133:  else revert("Gateway/file-unrecognized-param");

364:  revert("Gateway/invalid-message");

```
GitHub: [133](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L133), [364](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L364)




```
File: src/gateway/routers/axelar/Router.sol

66: revert("AxelarRouter/file-unrecognized-param");

```
GitHub: [66](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/routers/axelar/Router.sol#L66)



# [G-05] Using bools for storage incurs overhead

> // Booleans are more expensive than uint256 or any type that takes up a full
> // word because each write operation emits an extra SLOAD to first read the
> // slot's contents, replace the bits taken up by the boolean, and then write
> // back. This is the compiler's defense against contract upgrades and
> // pointer aliasing, and it cannot be disabled.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27 Use uint256(0) and uint256(1) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD

```
File: src/PoolManager.sol

57:  mapping(address => bool) allowedCurrencies;

```
GitHub: [57](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L57)

# [G-06] Don’t initialize variables with default value

The default value for variables is zero, so initializing them to zero is superfluous.

```
File: src/token/RestrictionManager.sol

15: uint8 public constant SUCCESS_CODE = 0;

```
GitHub: [15](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L15)
