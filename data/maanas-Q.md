# Findings Summary

- Informational : 7


|ID|Title|Severity|
|-|:-|:-:|
| NC-01| Remove commented code | Informational |
| NC-02| Messed up comment | Informational |
| NC-03| Misguiding comment | Informational |
| NC-04| Sequence dependent function calls can revert due to nature of cross-chain messages | Informational |
| NC-05| Typo | Informational |
| NC-06| Unused Event | Informational |
| NC-07| Misleading variable name | Informational |


# Detailed Findings

# NC-01 Remove commented code

## Instances

Insignificant code is leftover as comment
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L148-L152

```solidity
    function mint(uint256 shares, address receiver) public returns (uint256 assets) {
        // require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");
        assets = investmentManager.processMint(receiver, shares);
        emit Deposit(address(this), receiver, assets, shares);
    }
```

##

# NC-02 Messed up comment

## Instances

The comment is broken by messing up the lines.
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L196-L199

```solidity
    /// @notice Redeem shares after successful epoch execution. Receiver will receive assets for
    /// @notice Redeem shares can only be called by the Owner or an authorized admin.
    ///         the exact amount of redeemed shares from Owner after epoch execution.
    /// @return assets payout for the exact amount of redeemed shares
```

Change this to:

```solidity
    /// @notice Redeem shares after successful epoch execution. Receiver will receive assets for the exact amount of redeemed shares from Owner after epoch execution.
    /// @notice Redeem shares can only be called by the Owner or an authorized admin.
    /// @return assets payout for the exact amount of redeemed shares
```

##

# NC-03 Misguiding comment

## Instances

The comment states that the msg.sender could be a ward whereas it should actually only be the sender.  
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L96-L100

```solidity
    /// @dev Either msg.sender is the owner or a ward on the contract
    modifier withApproval(address owner) {
        require(msg.sender == owner, "LiquidityPool/no-approval");
        _;
    }
```

The comment states that maxDeposit is the max amount of shares that can be collected while maxDeposit actually is the maximum amount of assets that can be deposited
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L139-L140

```solidity
       /// @notice Collect shares for deposited assets after Centrifuge epoch execution.
    ///         maxDeposit is the max amount of shares that can be collected.
```

The title should be Paused Admin while it is given Delayed Admin

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L7

```solidity
/// @title  Delayed Admin
```

##

# NC-04 Sequence dependent function calls can revert due to nature of cross-chain messages

## Instances

Cross-chain messages are not guarenteed to arrive in the same sequence they have been invoked. Hence if two message are sent cross-chain at very close intervals, some function calls might revert.

isAllowPoolCurrency,isAddTranche can revert if the pool creation message has not been reached yet.
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L285-L366

```solidity
    function handle(bytes calldata message) external onlyIncomingRouter pauseable {
        if (Messages.isAddCurrency(message)) {
            (uint128 currency, address currencyAddress) = Messages.parseAddCurrency(message);
            poolManager.addCurrency(currency, currencyAddress);
        } else if (Messages.isAddPool(message)) {
            (uint64 poolId) = Messages.parseAddPool(message);
            poolManager.addPool(poolId);
        } else if (Messages.isAllowPoolCurrency(message)) {
            (uint64 poolId, uint128 currency) = Messages.parseAllowPoolCurrency(message);
            poolManager.allowPoolCurrency(poolId, currency);
        } else if (Messages.isAddTranche(message)) {
            (
                uint64 poolId,
                bytes16 trancheId,
                string memory tokenName,
                string memory tokenSymbol,
                uint8 decimals,
                uint128 _price
            ) = Messages.parseAddTranche(message);
            poolManager.addTranche(poolId, trancheId, tokenName, tokenSymbol, decimals);
        } else if (Messages.isUpdateMember(message)) {
            (uint64 poolId, bytes16 trancheId, address user, uint64 validUntil) = Messages.parseUpdateMember(message);
            poolManager.updateMember(poolId, trancheId, user, validUntil);
        } else if (Messages.isUpdateTrancheTokenPrice(message)) {
            (uint64 poolId, bytes16 trancheId, uint128 currencyId, uint128 price) =
                Messages.parseUpdateTrancheTokenPrice(message);
            investmentManager.updateTrancheTokenPrice(poolId, trancheId, currencyId, price);
        } else if (Messages.isTransfer(message)) {
            (uint128 currency, address recipient, uint128 amount) = Messages.parseIncomingTransfer(message);
            poolManager.handleTransfer(currency, recipient, amount);
        } else if (Messages.isTransferTrancheTokens(message)) {
            (uint64 poolId, bytes16 trancheId, address destinationAddress, uint128 amount) =
                Messages.parseTransferTrancheTokens20(message);
            poolManager.handleTransferTrancheTokens(poolId, trancheId, destinationAddress, amount);
        } else if (Messages.isExecutedDecreaseInvestOrder(message)) {
            (uint64 poolId, bytes16 trancheId, address investor, uint128 currency, uint128 currencyPayout) =
                Messages.parseExecutedDecreaseInvestOrder(message);
            investmentManager.handleExecutedDecreaseInvestOrder(poolId, trancheId, investor, currency, currencyPayout);
        } else if (Messages.isExecutedDecreaseRedeemOrder(message)) {
            (uint64 poolId, bytes16 trancheId, address investor, uint128 currency, uint128 trancheTokensPayout) =
                Messages.parseExecutedDecreaseRedeemOrder(message);
            investmentManager.handleExecutedDecreaseRedeemOrder(
                poolId, trancheId, investor, currency, trancheTokensPayout
            );
        } else if (Messages.isExecutedCollectInvest(message)) {
            (
                uint64 poolId,
                bytes16 trancheId,
                address investor,
                uint128 currency,
                uint128 currencyPayout,
                uint128 trancheTokensPayout
            ) = Messages.parseExecutedCollectInvest(message);
            investmentManager.handleExecutedCollectInvest(
                poolId, trancheId, investor, currency, currencyPayout, trancheTokensPayout
            );
        } else if (Messages.isExecutedCollectRedeem(message)) {
            (
                uint64 poolId,
                bytes16 trancheId,
                address investor,
                uint128 currency,
                uint128 currencyPayout,
                uint128 trancheTokensPayout
            ) = Messages.parseExecutedCollectRedeem(message);
            investmentManager.handleExecutedCollectRedeem(
                poolId, trancheId, investor, currency, currencyPayout, trancheTokensPayout
            );
        } else if (Messages.isScheduleUpgrade(message)) {
            address target = Messages.parseScheduleUpgrade(message);
            root.scheduleRely(target);
        } else if (Messages.isCancelUpgrade(message)) {
            address target = Messages.parseCancelUpgrade(message);
            root.cancelRely(target);
        } else if (Messages.isUpdateTrancheTokenMetadata(message)) {
            (uint64 poolId, bytes16 trancheId, string memory tokenName, string memory tokenSymbol) =
                Messages.parseUpdateTrancheTokenMetadata(message);
            poolManager.updateTrancheTokenMetadata(poolId, trancheId, tokenName, tokenSymbol);
        } else {
            revert("Gateway/invalid-message");
        }
    }
```

##

# NC-05 Typo

## Instances

Liquidity Pools must be changed to Liquidity Pool  
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L110

```solidity
        /// @return Total value of the shares, denominated in the asset of this Liquidity Pools
```

Must be changed to `Enable LP to take the tranche tokens out of escrow in case of investments`
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L325C1-L325C85

```solidity
        // Enable LP to take the tranche tokens out of escrow in case if investments
```

Change `forwarded` to `forwarders`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L18-L22

```solidity
/// @title  Tranche Token
/// @notice Extension of ERC20 + ERC1404 for tranche tokens,
///         which manages the liquidity pools that are considered
///         trusted forwarded for the ERC20 token, and ensures
///         the transfer restrictions as defined in the RestrictionManager.
```

Change `Recepeint` to `Recepient`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L251C105-L251C114

```solidity
        LiquidityPoolLike(liquidityPool).mint(address(escrow), trancheTokensPayout); // mint to escrow. Recepeint can claim by calling withdraw / redeem
```

##

# NC-06 Unused Event

## Instances

The file event is not used and not required in DelayedAdmin.  
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/DelayedAdmin.sol#L16

```solidity
        event File(bytes32 indexed what, address indexed data);
```

The file event is not used and not required in PauseAdmin.

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L19

```solidity
    event File(bytes32 indexed what, address indexed data);
```

##

# NC-07 Misleading variable name

## Instances

The variable is named as `currencyAmountInPriceDecimals` although it actually stores the `trancheTokenAmountInPriceDecimals`  
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L591-L603

```solidity
    function _calculateTrancheTokenAmount(uint128 currencyAmount, address liquidityPool, uint256 price)
        internal
        view
        returns (uint128 trancheTokenAmount)
    {
        (uint8 currencyDecimals, uint8 trancheTokenDecimals) = _getPoolDecimals(liquidityPool);


        uint256 currencyAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool).mulDiv(
            10 ** PRICE_DECIMALS, price, MathLib.Rounding.Down
        );


        trancheTokenAmount = _fromPriceDecimals(currencyAmountInPriceDecimals, trancheTokenDecimals, liquidityPool);
    }
```

##