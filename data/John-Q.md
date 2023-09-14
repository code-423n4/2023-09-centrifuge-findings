If `auth` perform wrong action, the permission will be going to be lost.
In `Auth` contract of `Auth.sol`, person whose `ward` is 1 can `rely` or `deny` the permission. If he performs wrong action, for example he `deny` the permission of other whose `ward` is 1 even himself.
In summary, the person whose `ward` is 1 have all permission that can control others' permission.

src/util/Auth.sol
```
// SPDX-License-Identifier: AGPL-3.0-only
// Copyright (C) Centrifuge 2020, based on MakerDAO dss https://github.com/makerdao/dss
pragma solidity 0.8.21;

/// @title  Auth
/// @notice Simple authentication pattern
contract Auth {
    mapping(address => uint256) public wards;

    event Rely(address indexed user);
    event Deny(address indexed user);

    /// @dev Give permissions to the user
    function rely(address user) external auth {
        wards[user] = 1;
        emit Rely(user);
    }

    /// @dev Remove permissions from the user
    function deny(address user) external auth {
        wards[user] = 0;
        emit Deny(user);
    }

    /// @dev Check if the msg.sender has permissions
    modifier auth() {
        require(wards[msg.sender] == 1, "Auth/not-authorized");
        _;
    }
}
```