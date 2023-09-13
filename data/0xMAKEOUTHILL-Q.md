In `Root.sol` there is a **MAX_DELAY** = 4 weeks used to prevent filing a delay that would block any updates indefinitely, but it is not enforced in the constructor which allows for the owner to set the delay to an amount bigger than 4 weeks and block updates indefinitely.

```
contract Root is Auth {
    /// @dev To prevent filing a delay that would block any updates indefinitely
    uint256 private MAX_DELAY = 4 weeks;

    //some code...

    mapping(address => uint256) public schedule;
    uint256 public delay;

    //some code...

    constructor(address _escrow, uint256 _delay) {
        escrow = _escrow;
        //@audit delay not enforced in constructor
        delay = _delay;

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }
```