# Low Severity Findings
## L01 Permissionless functions incentivize MEV

### Lines of code
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L141-L144
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L148-L152

### Impact
The fact that anyone can trigger the epoch execution, combined with a lack of access control on deposit and mint functions, could give investors with conflicting interests reasons to engage in negative MEV against each other, in order to secure a particular pool's state.

### Description 
Consider the following scenario:

- Alice wishes to invest in the junior tranche of a pool, and calls requestDeposit.
- After the minimal epoch duration has elapsed, but before execution is triggered, Alice changes her mind and cancels her order.
- Bob wishes to invest in the senior tranche in the next epoch, but notices the subordination ratio is too low at present for such an order to go through: 
    SubordinationRatio = value(Junior) / ( value(Junior) + value(Senior))
As the ratio is below its target, the pool will not accept further senior investments nor junior redeems. Thus Bob needs to secure Alice's deposit for his own plans.
- Bob front runs Alice's cancellation with an epoch execution.
- Bob back runs the epoch execution with a call to deposit on behalf of Alice
- As the pool's subordination ratio has improved, Bob can now call requestDeposit on the senior tranche.
Alice now has an investment that she expected to be cancelled; she could be stuck with it if the subordination ratio keeps evolving adversely and a partial freeze prevents her from redeeming, through other investments or the pool's NAV, .

### Proof-of-Concept
The following addition to LiquidityPool.t.sol demonstrates how anyone can call deposit on behalf of an investor:

```diff
diff --git a/./LiquidityPool.t.sol.orig b/./LiquidityPool.t.sol
index 4e60ec1..a392c78 100644
--- a/./LiquidityPool.t.sol.orig
+++ b/./LiquidityPool.t.sol
@@ -1346,4 +1346,38 @@ contract LiquidityPoolTest is TestSetup {
         );
         investor.deposit(_lPool, amount, _investor); // deposit the amount
     }
+
+        function testMEVDeposit( ) public {
+        uint64 poolId = 9;
+        bytes16 trancheId = bytes16("9");
+        uint128 currencyId = 9;
+        uint256 amount = 1e18;
+        uint64 validUntil = uint64(block.timestamp + 1);
+
+        // setup
+        address _lPool_ = deployLiquidityPool(poolId, erc20.decimals(), "TEST1", "TT1", trancheId, currencyId);
+        LiquidityPool lPool = LiquidityPool(_lPool_);
+        erc20.mint(self, amount);
+        homePools.updateMember(poolId, trancheId, self, validUntil); 
+        erc20.approve(address(investmentManager), amount); 
+
+        // request deposit
+        lPool.requestDeposit(amount, self);
+
+        // front run with epoch execution
+        vm.startPrank(address(999));
+        homePools.isExecutedCollectInvest(
+            poolId, trancheId, bytes32(bytes20(self)), currencyId, uint128(amount), uint128(amount)
+        );
+        vm.stopPrank();
+
+        // attempt to cancel order
+        lPool.requestDeposit(0, self);
+
+        // back run execution with actual deposit
+        vm.startPrank(address(999));
+        lPool.deposit(amount,self);
+        vm.stopPrank();
+        
+    }
 }
```

### Recommended mitigation steps
Consider adding access control to deposit/mint.
Warn users of the associated risks.
