## Inconsistency of `checkTransferRestriction()` implementation
Unless it's being intended for, `checkTransferRestriction` in InvestmentManager.sol has not been fairly imposed on all pairs of deposit/redeem logic.

Here are the two instances entailed:

1. `LiquidityPoolLike(liquidityPool).checkTransferRestriction` is enforced in [`handleExecutedDecreaseRedeemOrder()`](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L307-L310) but not in [`handleExecutedDecreaseInvestOrder()`](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L277-L292).
2. `lPool.checkTransferRestriction` is enforced in [`_deposit()`](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L473) but not in [`_redeem()`](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L535-L548).  

## Gas poisoning on optional logic
In InvestmentManager.sol, if an amount of 0 is passed, this triggers cancelling outstanding deposit/redemption orders. Since no amount of currency or tranche tokens have been involved, it does not make good sense trying to send `cancelInvestOrder()` or `cancelRedeemOrder()` through the gateway. Apparently, a malicious actor could send lots of requests entailing 0 amounts, thereby congesting the gateway.  

Consider refactoring the affected logic:

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L127-L133

```diff
        if (_currencyAmount == 0) {
-            // Case: outstanding investment orders only needed to be cancelled
-            gateway.cancelInvestOrder(
-                lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset())
-            );
            return;
        }
```  
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L159-L165

```diff
        if (_trancheTokenAmount == 0) {
-            // Case: outstanding redemption orders will be cancelled
-            gateway.cancelRedeemOrder(
-                lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset())
-            );
            return;
        }
```
## Excessive calls on `_updateLiquidityPoolPrice()`
In InvestmentManager.sol, each time `handleExecutedCollectInvest()` or `handleExecutedCollectRedeem()` is called,  `_updateLiquidityPoolPrice()` will be invoked to update the liquidity pool with the latest tranche token price. This could be add up excessively considering the amount of requests executed usually ran in huge volumes. 

Since `updateTrancheTokenPrice()` has already been implemented to accomplish the same purpose at the end of each epoch, consider removing the optional code logic:

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L252
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L274

```diff
-        _updateLiquidityPoolPrice(liquidityPool, currencyPayout, trancheTokensPayout);
```
## Unneeded modifier
`LiquidityPool.withApproval` is practically not needed.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L97-L100

```solidity
    modifier withApproval(address owner) {
        require(msg.sender == owner, "LiquidityPool/no-approval");
        _;
    }
```
For instance, `withdraw()` below may simply be refactored by removing the `withApproval()` visibility:

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L176-L184

```diff
-    function withdraw(uint256 assets, address receiver, address owner)
+    function withdraw(uint256 assets, address receiver)
        public
-        withApproval(owner)
        returns (uint256 shares)
    {
-        uint256 sharesRedeemed = investmentManager.processWithdraw(assets, receiver, owner);
+        uint256 sharesRedeemed = investmentManager.processWithdraw(assets, receiver, msg.sender);
        emit Withdraw(address(this), receiver, owner, assets, sharesRedeemed);
        return sharesRedeemed;
    }
```
All other instances entailed:
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L200-L208
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L214-L217
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L231-L234
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L247-L249
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L253-L255

## Uninitialized variables
Consider initializing essential state variables at the constructor to ensure smooth early function calls.

For instance, `gateway` and `poolManager` in InvestmentManager.sol should be initialized upon contract deployment just as this has been implemented LiquidityPool.sol, Gateway.sol and Root.sol rather than strictly through file methods. 

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L88-L94

```diff
    constructor(address escrow_, address userEscrow_) {
        escrow = EscrowLike(escrow_);
        userEscrow = UserEscrowLike(userEscrow_);
+        gateway = GatewayLike(data);
+        poolManager = PoolManagerLike(data);

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }
```
All other instances entailed:
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L120-L125
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/routers/axelar/Router.sol#L62-L70
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L83-L88
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol#L42-L46

## Incorrect comments
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L140

```diff
-    ///         maxDeposit is the max amount of shares that can be collected.
+    ///         maxDeposit is the max amount of assets that can be deposited.
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L65

```diff
-    // important: the decimals of the leading pool currency.
+    // important: the decimals of the lending pool currency.
```
## Cached variable not fully utilized
In `InvestmentManager.requestDeposit`, `lPool.asset()` has been cached as `currency` but it is still being used in subsequent code lines:

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L117-L141

```diff
    function requestDeposit(uint256 currencyAmount, address user) public auth {
        address liquidityPool = msg.sender;
        LiquidityPoolLike lPool = LiquidityPoolLike(liquidityPool);
        address currency = lPool.asset();
        uint128 _currencyAmount = _toUint128(currencyAmount);

        // Check if liquidity pool currency is supported by the Centrifuge pool
        poolManager.isAllowedAsPoolCurrency(lPool.poolId(), currency);
        // Check if user is allowed to hold the restricted tranche tokens
        _isAllowedToInvest(lPool.poolId(), lPool.trancheId(), currency, user);
        if (_currencyAmount == 0) {
            // Case: outstanding investment orders only needed to be cancelled
            gateway.cancelInvestOrder(
-                lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset())
+                lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(currency)
            );
            return;
        }

        // Transfer the currency amount from user to escrow. (lock currency in escrow).
        SafeTransferLib.safeTransferFrom(currency, user, address(escrow), _currencyAmount);

        gateway.increaseInvestOrder(
-            lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset()), _currencyAmount
+            lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(currency), _currencyAmount
        );
    }
```
## Membership expiration
All investors have been KYC'd and AML'd before they could deposit/mint and withdraw/redeem in LiquidityPool.sol. Under this context, consider making their membership indefinite. Gas saving on reduced logic aside, investors would not have to run into issues where their requests and executions (processes) got denied just moments after the membership had expired. Additionally, wards would not have to take the trouble to periodically extend investors' expired membership. And if an investor needed to be removed, the option would always be there for the ward to disfellowship it.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L57-L67

```diff
-    function updateMember(address user, uint256 validUntil) public auth {
+    function updateMember(address user) public auth {
-        require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");
-        members[user] = validUntil;
+        members[user] = type(uint256).max;
    }

-    function updateMembers(address[] memory users, uint256 validUntil) public auth {
+    function updateMembers(address[] memory users) public auth {
        uint256 userLength = users.length;
        for (uint256 i = 0; i < userLength; i++) {
-            updateMember(users[i], validUntil);
+            updateMember(users[i]);
        }
    }
```
## Inexpedient previews
In InvestmentManager.sol, `previewDeposit()`, `previewMint()`, `previewWithdraw()` and `previewRedeem()` will all be returning 0 if `depositPrice/redeemPrice == 0`. Additionally, the prices will be pre-determined from lpValues.maxDeposit/lpValues.maxMint or lpValues.maxWithdraw/lpValues.maxRedeem via [`calculateDepositPrice()`](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L557) or [`calculateRedeemPrice()`](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L566). Like it or not, these static prices aren't going to change regardless of the latest prices till the processes are called. 

## Misleading variable name
Consider renaming the memory variable below to enhance readability and avoid confusion. 

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L591-L603

```diff
    function _calculateTrancheTokenAmount(uint128 currencyAmount, address liquidityPool, uint256 price)
        internal
        view
        returns (uint128 trancheTokenAmount)
    {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);

-        uint256 currencyAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool).mulDiv(
+        uint256 trancheTokenAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool).mulDiv(
            10 ** PRICE_DECIMALS, price, MathLib.Rounding.Down
        );

-        trancheTokenAmount = _fromPriceDecimals(currencyAmountInPriceDecimals, trancheTokenDecimals, liquidityPool);
+        trancheTokenAmount = _fromPriceDecimals(trancheTokenAmountInPriceDecimals, trancheTokenDecimals, liquidityPool);
    }
```