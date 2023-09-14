-(1)  Import gateway functions should return a bool to indicate successful function execution or functions that involves talking to bridges or the router, to add an extra layer of security
- like so:- e.g on [cancelredeemOrder](https://github.com/code-423n4/2023-09-centrifuge/blob/705e3500f8a43579cb15751d09100c93b249120f/src/gateway/Gateway.sol#L276)
```diff
diff --git a/src/gateway/Gateway.sol b/src/gateway/Gateway.sol
index a9ff8bd..b75f74d 100644
--- a/src/gateway/Gateway.sol
+++ b/src/gateway/Gateway.sol
@@ -277,8 +277,10 @@ contract Gateway is Auth {
         public
         onlyInvestmentManager
         pauseable
+        returns (bool)
     {
         outgoingRouter.send(Messages.formatCancelRedeemOrder(poolId, trancheId, _addressToBytes32(investor), currency));
+        return true;
     }
```
this bool can then be checked against for successful funntion execution like so
```dif
diff --git a/src/InvestmentManager.sol b/src/InvestmentManager.sol
index fe94965..96cd5a5 100644
--- a/src/InvestmentManager.sol
+++ b/src/InvestmentManager.sol
@@ -17,7 +17,9 @@ interface GatewayLike {
     function collectInvest(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
     function collectRedeem(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
     function cancelInvestOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
-    function cancelRedeemOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
+    function cancelRedeemOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
+        external
+        returns (bool);
 }
 
@@ -158,9 +161,10 @@ contract InvestmentManager is Auth {
         if (_trancheTokenAmount == 0) {
             // Case: outstanding redemption orders will be cancelled
-            gateway.cancelRedeemOrder(
+            (bool success) = gateway.cancelRedeemOrder(
                 lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset())
             );
+            require(success, "InvestmentManager/run-not-successful");
             return;
         }
```
