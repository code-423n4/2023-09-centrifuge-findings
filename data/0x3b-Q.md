### [L-01] You can deposit for another user, however you cannot preview the deposit
[previewDeposit](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L135-L137) works explicitly with `msg.sender` as the user to deposit, meaning you cannot preview deposits for other users. However you can still deposit for other users, since in [deposit](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L141-L144) you can insert an address as the user. Make it possible for users to preview the deposit of another user.
```diff
-    function previewDeposit(uint256 assets) public view returns (uint256 shares) {
-        shares = investmentManager.previewDeposit(msg.sender, address(this), assets);
-    }

+    function previewDeposit(uint256 assets,address user) public view returns (uint256 shares) {
+       shares = investmentManager.previewDeposit(user, address(this), assets);
+    }
```
