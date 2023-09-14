# Non-critical
## [NC-01] Unused interface
In [Escrow.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Escrow.sol#L7C1-L9C2), the interface `ApproveLike` is not used in the whole project. Remove that.

## [NC-02] Unclear string error
In [UserEscrow::transferOut](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L37C1-L37C91), the error highlighted shall be more explicit. It is checking that the amount does NOT exceed the mapping value, so consider changing the error string to something like `UserEscrow/amount-exceeds-destination` to make it more explicit and clear.

## [NC-03] Misplaced comment
I will just [point where it is](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L10). It shall be above `Invalid` INSIDE the enum block

## [NC-04] Redundant code
When dealing with calls from pools within `InvestmentManager`, there is a pattern with `modifier auth - address liquidityPool = msg.sender` in many functions. As the modifier "ensures" the caller is from a ward/pool (as other wards do not call such functions), consider removing the `address liquidityPool = msg.sender;` and use message sender when initializing the `LiquidityPoolLike lPool` variable. Check [here](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L117C74-L119C68) but the occurrences is within the whole file.