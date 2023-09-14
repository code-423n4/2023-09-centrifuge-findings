QA
[Q-1]
Severity: non-critical
Source file requires different compiler version. Recommendation: Use the same version in compiler and .sol files.

[Q-2]
Severity: non-critical
Move all interface declarations into separate `I<name>.sol` files, import them, and reuse them across the codebase.

[Q-3]
Severity: non-critical
Add a blacklist for non-allowed currency coins (those with rebase logic and fee-on-transfer-logic). Implement additional inspection in the `addCurrency` method to ensure that the currency address is not in the blacklist.

[Q-4]
Severity: non-critical
The collectDeposit and collectRedeem functions in InvestmentManager.sol are almost identical. Consider using the file pattern to create a common function for both functionalities.

[Q-5]
Severity: non-critical
Duplication of lines related to `_currency` and `liquidityPool` across the code. It's better to create reusable functions. The lpValues selector `orderbook[user][liquidityPool]` is also duplicated. Consider creating a getter for user LP values.

[Q-6]
Severity: non-critical
The functions in `PoolManager.sol` that deploy tranches and pools are public. It might be more appropriate for only the admin to have the ability to deploy tranches and pools.

[Q-7]
Severity: low
The `public onlyGateway` modifier seems a bit unconventional since `onlyGateway` refers to a contract. Consider changing this modifier to `external onlyGateway`.

[Q-8]
Severity: low
Certain functions in PoolManager.sol do not emit events. Consider adding appropriate events for these functions.

[Q-9]
Severity: low
In `PoolManager.sol`, it would be beneficial to add an `UpdateMember` event.

[Q-10]
Severity: low
In `PoolManager.sol`, consider adding a getter for `pools[poolId].tranches[trancheId]`.

[Q-11]
Severity: non-critical
In `LiquidityPool.sol`, the description of the redeem function includes "or an authorized admin." This should be removed. The same words should also be removed from another function in the same file.
[Q-12]
Severity: low
https://github.com/code-423n4/2023-09-centrifuge/blob/e725dde1901f00aa0574026de24f0de6764618d4/src/InvestmentManager.sol#L583
In a mathematical way, calculation of shares prices should be RoundUp, instead of rounding down