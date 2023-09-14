# [01] Zero \_root checks

`_root` can be zero if the minimum fund is not met.

```solidity
File: Factory.sol

29: constructor(address _root) {
30:     root = _root;

74: constructor(address _root) {
75:     root = _root;
```

[Line: 30](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L30)
[Line: 75](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L75)

As such, consider introducing checks on functions calls dependent on it when `_root == 0`. For example, it will be good if a check is implemented in constructor of Root contract to prevent some error.

# [02] Uses timestamp for comparisons

Dangerous usage of `block.timestamp`. `block.timestamp` can be manipulated by miners.

```solidity
File: ERC20.sol

217: require(block.timestamp <= deadline, "ERC20/permit-expired");
```

[Line: 217](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L217)

```solidity
File: Root.sol

77: require(schedule[target] < block.timestamp, "Root/target-not-ready");
```

[Line: 77](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L77)

```solidity
File: RestrictionManager.sol

46: require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");
50: if (members[user] >= block.timestamp) {
58: require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");
```

[Line: 46](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L46)
[Line: 50](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L50)
[Line: 58](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L58)

Avoid relying on `block.timestamp`.

# [03] Cyclomatic complexity

Gateway.handle() has a high cyclomatic complexity

```solidity
File: Gateway.sol

286: if (Messages.isAddCurrency(message)) {
        ...
363: } else {
```

[Lines](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L285-L366)