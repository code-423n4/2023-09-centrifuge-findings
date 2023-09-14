## 1. Use memory instead of storage
`calculateDepositPrice` and `calculateRedeemPrice` access `orderbook`'s info by storage, which will waste gas because the `lpValues` is not overridden.

[calculateDepositPrice calculateRedeemPrice](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L551)
```
    // --- Helpers ---
    function calculateDepositPrice(address user, address liquidityPool) public view returns (uint256 depositPrice) {
        LPValues storage lpValues = orderbook[user][liquidityPool];
        if (lpValues.maxMint == 0) {
            return 0;
        }

        depositPrice = _calculatePrice(lpValues.maxDeposit, lpValues.maxMint, liquidityPool);
    }

    function calculateRedeemPrice(address user, address liquidityPool) public view returns (uint256 redeemPrice) {
        LPValues storage lpValues = orderbook[user][liquidityPool];
        if (lpValues.maxRedeem == 0) {
            return 0;
        }

        redeemPrice = _calculatePrice(lpValues.maxWithdraw, lpValues.maxRedeem, liquidityPool);
    }
```