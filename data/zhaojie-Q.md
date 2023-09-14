
## The Router has excessive permissions

Once added to the incomingRouters, the Router can perform any operations such as addCurrency, transfer tokens in pools, etc.

Many operations that require permissions can be performed through Gateway.handle, such as token transfer.

The onlyIncomingRouter modifier verifies whether the msg.sender exists in the incomingRouters map.

msg.sender is the Router. The Router calls gateway.handle after receiving the cross-chain message. There are two routers in the current code.


`If any one of the Routers added to incomingRouters is at risk, the entire protocol is at risk.`

Router has the possibility of risk, such as cross-chain bridge attack, cross-chain bridge is often attacked by hackers or other problems, such as MultiChain some time ago.

The Router can construct arbitrary messages, such as a Transfer message. The gateway.handle finally invokes the poolManager and sends the token to the specified recipient.

The Router can be removed by the administrator, but the router has too high permission, it can transfer all the tokens in the pool. 
If something goes wrong, it is too late to remove the Router after the token is transferred.

```solidity
    modifier onlyIncomingRouter() {
        require(incomingRouters[msg.sender], "Gateway/only-router-allowed-to-call");
        _;
    }
```

```solidity
function handle(bytes calldata message) external onlyIncomingRouter pauseable {
        ......
        } else if (Messages.isTransfer(message)) {
            (uint128 currency, address recipient, uint128 amount) = Messages.parseIncomingTransfer(message);
            poolManager.handleTransfer(currency, recipient, amount);
```

PoolManager.handleTransfer only allow Gateway to call

```solidity
    function handleTransfer(uint128 currency, address recipient, uint128 amount) public onlyGateway {
        address currencyAddress = currencyIdToAddress[currency];
        require(currencyAddress != address(0), "PoolManager/unknown-currency");

        EscrowLike(escrow).approve(currencyAddress, address(this), amount);
        SafeTransferLib.safeTransferFrom(currencyAddress, address(escrow), recipient, amount);
    }
```

