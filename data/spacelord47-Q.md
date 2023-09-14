## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Confusing comments | 1 | 
| [L&#x2011;02] | Missing `_delay` parameter validation in `Root.constructor` | 1 | 

Total: 2 instances over 2 issues.

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Confusing comments | 2 | 

Total: 2 instances over 1 issue.


## Low Risk Issues

### [L-01] Confusing comments

There are comments containing incorrect information, which may lead to incorrect code understading by developers. It's suggested to keep comments up-to-date with the code.

```solidity
/// @dev Either msg.sender is the owner or a ward on the contract
modifier withApproval(address owner)
    require(msg.sender == owner, "LiquidityPool/no-approval");
    _;
}
```
[GitHub: src/LiquidityPool.sol#L96](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L96)

"or a ward" should be omitted in this case.

### [L-02] Missing `_delay` parameter validation in `Root.constructor`

The `delay` field in `Root` contract expected to conform the limit specified in `MAX_DELAY` constant:

```solidity
contract Root is Auth {
    /// @dev To prevent filing a delay that would block any updates indefinitely
    uint256 private MAX_DELAY = 4 weeks;
    // ...
```
[GitHub: src/Root.sol#L17](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L17)

It's properly validated when setting in the `file` function:

```solidity
function file(bytes32 what, uint256 data) external auth {
    if (what == "delay") {
        require(data <= MAX_DELAY, "Root/delay-too-long");
	delay = data;
    // ...
```
[GitHub: src/Root.sol#L43](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L43)

But the validation is missing in the constructor:

```solidity
constructor(address _escrow, uint256 _delay) {
    escrow = _escrow;
    delay = _delay;
    // ..
```
[GitHub: src/Root.sol#L36](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L36)
#### Recommended Mitigation Steps

Validate `_delay` parameter in the constructor:

```solidity
constructor(address _escrow, uint256 _delay) {
    require(_delay <= MAX_DELAY, "Root/delay-too-long");

    escrow = _escrow;
    delay = _delay;
    // ..
```


## Non Critical Issues

### [N-01] Confusing comments

There are comments containing incorrect information(not leading to the code misunderstanding). Most occurrences due to the comment not updated after copy-paste.

*There are 2 instance of this issue:*

```solidity
/// @title  Delayed Admin
/// @dev    Any ward can manage accounts who can pause.
///         Any pauser can instantaneously pause the Root.
contract PauseAdmin is Auth {
```
[GitHub: src/admins/PauseAdmin.sol#L7](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L7)

"@title  Delayed Admin" should be replaced with "@title  Pause Admin"

---

```solidity
/// @title  IERC20
/// ..
interface IERC4626 is IERC20 {
```
[GitHub: src/interfaces/IERC4626.sol#L7](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/interfaces/IERC4626.sol#L7)

"@title  IERC20" should be replaced with "@title  IERC4626"