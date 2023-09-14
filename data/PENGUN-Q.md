## Root.sol: delay could be set higher than MAX_DELAY in constructor

```solidity
constructor(address _escrow, uint256 _delay) {
        escrow = _escrow;
        delay = _delay;

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }
```

There is no check for delay at constructor.

## Root.sol: delay could be set zero to bypass timelock.

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
Admin can set delay value in file(). There is MAX_DELAY check but no MIN_DELAY.

```solidity
    /// --- Timelocked ward management ---
    function scheduleRely(address target) external auth {
        schedule[target] = block.timestamp + delay;
        emit RelyScheduled(target, schedule[target]);
    }
```
When delay is set to zero, admin can bypass timelock.