
# delay must be checked in Root constructor
`_delay` value is not checked by the constructor, and so can exceed `MAX_DELAY` value.

Add the following statement to constructor:

```solidity
require(_delay <= MAX_DELAY, "Root/delay-too-long");
```

# Dead code in DelayedAdmin
Scope: https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/DelayedAdmin.sol#L16


The event File is declared in DelayedAdmin.sol, but none of its functions emit this event.

This line of code can so be erased.

```solidity

    // --- Events ---
    event File(bytes32 indexed what, address indexed data);
```

# Missing checks for address(0x0) in Auth.sol
A check must be implemented in `rely()` function to avoid giving wards rights to 0x0.

Add a require statement such as `require(user != address(0x0), "user is zero address")`.


