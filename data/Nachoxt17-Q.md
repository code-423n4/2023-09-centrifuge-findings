https://github.com/code-423n4/2023-09-centrifuge/blob/0af232255c7d045efde4ac40801dfeeed8a8d889/src/util/Auth.sol#L14

### No check for Zero Address:
`./src/util/Auth.sol`:
```solidity
//- Lines #14-16:
/// @dev Give permissions to the user
    function rely(address user) external auth {
        wards[user] = 1;
        emit Rely(user);
    }
```

#### 📄Description:
The smart contract currently lacks a safeguard to prevent the assignment of permissions to the null address (0x0). This oversight could potentially result in unintentional relinquishment of control.

#### 🛠️Solution:
In the `rely(address user)` function, it is recommended to incorporate the following requirement: `require(user != address(0), "Address cannot be zero");`. This will ensure that the user's address is not null, thereby enhancing the security of the smart contract.