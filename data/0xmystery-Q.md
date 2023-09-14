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