## Summary
[I-1] Use better naming for parameter.
[I-2] Useless casting.
[I-3] Typo.
[I-4] Function can be modifier.

### [I-1] Use better naming for parameter.
The function `file` can be seen in many files. However the first parameter is called `what`. This is not good practice as the name does not explain what this parameter is used for. Change the name of the parameter to improve better code readability.

### [I-2] Useless casting.
In `PoolManager.sol` the variable `escrow` is set to `EscrowLike(escrow_)` in the constructor. Later when the variable is used in some of the function, it is casted to `EscrowLike` again, which is useless. The tests still pass when casting is removed. Remove the `EscrowLike` casting of `escrow` variable.

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L249

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L252

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L261

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L328

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L329

### [I-3] Typo.

```solidity
    // --- EIP712 niceties --- @audit necessities
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L29

### [I-4] Function can be modifier.
The function `member` only consists of `require` statement. If this is the expected behaviour, then it better be changed to `modifier`. Potentially this could not be the expected behaviour, bacause the function is marked as `view`, but returns nothing.

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L45-L47