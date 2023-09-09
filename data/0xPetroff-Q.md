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