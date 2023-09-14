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
## Functions line separation within contracts
Functions within contracts should be separated by a single line as denoted in the link below:

https://docs.soliditylang.org/en/latest/style-guide.html#blank-lines

The bot report brought up this issue albeit with wrong instances entailed. Here's one of the supposed instances:

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L9-L42

```solidity
interface ERC20PermitLike {
    function permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
        external;
    function PERMIT_TYPEHASH() external view returns (bytes32);
    function DOMAIN_SEPARATOR() external view returns (bytes32);
}

interface TrancheTokenLike is IERC20, ERC20PermitLike {
    function checkTransferRestriction(address from, address to, uint256 value) external view returns (bool);
}

interface InvestmentManagerLike {
    function processDeposit(address receiver, uint256 assets) external returns (uint256);
    function processMint(address receiver, uint256 shares) external returns (uint256);
    function processWithdraw(uint256 assets, address receiver, address owner) external returns (uint256);
    function processRedeem(uint256 shares, address receiver, address owner) external returns (uint256);
    function maxDeposit(address user, address _tranche) external view returns (uint256);
    function maxMint(address user, address _tranche) external view returns (uint256);
    function maxWithdraw(address user, address _tranche) external view returns (uint256);
    function maxRedeem(address user, address _tranche) external view returns (uint256);
    function totalAssets(uint256 totalSupply, address liquidityPool) external view returns (uint256);
    function convertToShares(uint256 assets, address liquidityPool) external view returns (uint256);
    function convertToAssets(uint256 shares, address liquidityPool) external view returns (uint256);
    function previewDeposit(address user, address liquidityPool, uint256 assets) external view returns (uint256);
    function previewMint(address user, address liquidityPool, uint256 shares) external view returns (uint256);
    function previewWithdraw(address user, address liquidityPool, uint256 assets) external view returns (uint256);
    function previewRedeem(address user, address liquidityPool, uint256 shares) external view returns (uint256);
    function requestRedeem(uint256 shares, address receiver) external;
    function requestDeposit(uint256 assets, address receiver) external;
    function collectDeposit(address receiver) external;
    function collectRedeem(address receiver) external;
    function decreaseDepositRequest(uint256 assets, address receiver) external;
    function decreaseRedeemRequest(uint256 shares, address receiver) external;
}
```
All other instances mostly hover around the in file interfaces:
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L8-L56
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L6-L10
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Escrow.sol#L7-L9
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L11-L50 

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.21",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Kindly visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here is one particular example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.