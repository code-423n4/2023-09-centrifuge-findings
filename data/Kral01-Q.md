## [QA-01] The `withApproval` modifier in LiquidityPool.sol is not really needed.

As it stands now the `withApproval` in [LiquidityPool.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol) checks if the `owner` variable passed into functions is equal to `msg.sender` such as this:

```solidity
 function requestDeposit(uint256 assets, address owner) public withApproval(owner) {
        investmentManager.requestDeposit(assets, owner);
        emit DepositRequested(owner, assets);
    }
```
This can be refactored into;

```solidity
 function requestDeposit(uint256 assets) public {
        investmentManager.requestDeposit(assets, msg.sender);
        emit DepositRequested(msg.sender, owner);
    }
```

Which has the same functionality as using the modifier. There are many instances of using this `withApproval` modifier throughout the `LiquidityPool.sol` contract.

## [QA-02] Redundant return statements.

There are some instances of using `returns(type variableName)` while also using another return in the function. This can be avoided.
In [LiquidityPool.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol)

```solidity
function withdraw(uint256 assets, address receiver, address owner)
        public
        withApproval(owner)
        returns (uint256 shares) //@audit return
    {
        uint256 sharesRedeemed = investmentManager.processWithdraw(assets, receiver, owner);
        emit Withdraw(address(this), receiver, owner, assets, sharesRedeemed);
        return sharesRedeemed;//@audit return
    }
```

```solidity
 function redeem(uint256 shares, address receiver, address owner)
        public
        withApproval(owner)
        returns (uint256 assets)//@audit return
    {
        uint256 currencyPayout = investmentManager.processRedeem(shares, receiver, owner);
        emit Withdraw(address(this), receiver, owner, currencyPayout, shares);
        return currencyPayout;//@audit return
    }
```

The return statements can be avoided by using the same variable names as in the function inside the `returns()`.

Like:
```solidity
function withdraw(uint256 assets, address receiver, address owner)
        public
        withApproval(owner)
        returns (uint256 sharesRedeemed )
    {
        uint256 sharesRedeemed = investmentManager.processWithdraw(assets, receiver, owner);
        emit Withdraw(address(this), receiver, owner, assets, sharesRedeemed);
    }
```

This can make the code much cleaner.

