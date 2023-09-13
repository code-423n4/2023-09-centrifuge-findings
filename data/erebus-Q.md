# Non-critical
## [NC-01] Unused interface
In [Escrow.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Escrow.sol#L7C1-L9C2), the interface `ApproveLike` is not used in the whole project. Remove that.

## [NC-02] Unclear string error
In [UserEscrow::transferOut](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L37C1-L37C91), the error highlighted shall be more explicit. It is checking that the amount does NOT exceed the mapping value, so consider changing the error string to something like `UserEscrow/amount-exceeds-destination` to make it more explicit and clear.