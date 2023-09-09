## Impact
Missing constructor sanity checks

The implementation of constructor() functions are missing some sanity checks.


## Proof of Concept

Missing sanity check for != address(0)

There're no sanity checks for the constructor argument escrow_ and userEscrow_.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol?plain=1#L88-L94
```solidity
    constructor(address escrow_, address userEscrow_) {
        escrow = EscrowLike(escrow_);
        userEscrow = UserEscrowLike(userEscrow_);

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol?plain=1#L29-L34
```solidity
    constructor(address _root) {
        root = _root;

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }
```

## Tools Used
Manual code review

## Recommended Mitigation Steps
To reduce possible errors and make the code more robust, consider adding sanity checks where needed.

For example:
```
require(
      escrow_ != address(0) || userEscrow_ != address(0),
      "ZERO_ADDRESS"
    );
```
