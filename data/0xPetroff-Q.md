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