## Title

there is no validation of delay time in root.sol constructor, this allows delay to be indefinite when there is a clear maximum.

## Vulnerability Details

in root.sol there is an invariant
```solidity
 uint256 private MAX_DELAY = 4 weeks;
```
this invariant is validated when updating delay within the `file` function
```solidity
function file(bytes32 what, uint256 data) external auth {
        if (what == "delay") {
            require(data <= MAX_DELAY, "Root/delay-too-long");
            delay = data;
        } else {
            revert("Root/file-unrecognized-param");
        }
        emit File(what, data);
    }
```
this ensure that delay is never above `MAX_DELAY`
However there is no check in the constructor that ensure delay is below `MAX_DELAY`
```solidity
    constructor(address _escrow, uint256 _delay) {
        escrow = _escrow;
        delay = _delay;

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }
```
this allows the deployer to bypass the MAX_DELAY invariant.

## Impact
inavriant can be passed, allowing deployer of contract to set delay to a very high number, essentially DOSing the contract until it is changed. 
