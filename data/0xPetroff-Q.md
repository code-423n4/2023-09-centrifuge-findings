In functions such as ``requestDeposit()`` in LiquidityPool.sol on line 214 there are required two arguments: a uint256 and an address. That function uses a modifier which checks if the passed in address is our own. The second argument can just be removed and substituted with msg.sender and the modifier removed.

```
function requestDeposit(uint256 assets, address owner) public withApproval(owner) {
     investmentManager.requestDeposit(assets, owner);
     emit DepositRequested(owner, assets);
}
```

```
    modifier withApproval(address owner) {
        require(msg.sender == owner, "LiquidityPool/no-approval");
        _;
    }
```






In ``InvestmentManager.sol`` on L662 the ``checkTransferRestriction`` method requires 3 arguments: ``from, to and value``. Following the function throughout the whole protocol we can see that its core functionality is met in ``RestrictionManager.sol`` on L28 where it still accepts the 3 arguments from before, but uses only ``to`` to check whether or not the ``to`` address is a valid investor. 

The 2 arguments (``from`` and ``value``) passed down are not needed nor used throughout the protocol's functionality to check if the user is allowed to invest.






In ``Factory.sol`` we can create a Liquidity Pool, initially setting a poolId and trancheId. In ``PoolManager.sol`` on L197 the function ``addTranche()`` is used to create tranches. On L210 and L211 we are re-setting these poolId and trancheId, which have been left from an old design and can be simply removed to improve the simplicity of the code.



Renaming the function [allowPoolCurrency](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L179C1-L179C5) to ``allowInvestmentCurrency`` would be more proper and suit it better, as well as clear up some potential confusion from the auditors POV about the function's functionality.