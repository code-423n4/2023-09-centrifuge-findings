
## Summary

### Low Risk Issues

no | Issue |Instances|
|-|:-|:-:|:-:|
| [L-01] | Unchecked return value of low level call()/delegatecall() | 2 | 
| [L-02] | Before transfer of some functions, we should check some variables | 4 | 

## Low Risk Issues

### [L-1] Unchecked return value of low level call()/delegatecall()

When you use call() or delegatecall() to invoke external contract functions, they return a boolean value indicating whether the function call was successful or not. Failing to check the return value can lead to unexpected behavior and vulnerabilities in your smart contract.
It's important to always check the return value of call() or delegatecall() and handle any potential errors. 


```solidity
file: /src/LiquidityPool.sol

296        (bool success, bytes memory data) = address(share).call(bytes.concat(msg.data, bytes20(msg.sender)));

308        (bool success, bytes memory data) = address(share).call(bytes.concat(msg.data, bytes20(msg.sender)));

```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L296


### [L-2] Before transfer of some functions, we should check some variables

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything

```solidity
file: /src/UserEscrow.sol

32        SafeTransferLib.safeTransferFrom(token, source, address(this), amount);

```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L32


```solidity
file: /src/token/Tranche.sol

60        return super.transfer(to, value);

69        return super.transferFrom(from, to, value);

```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol#L60


```solidity
file: /src/gateway/Gateway.sol

197        outgoingRouter.send(Messages.formatTransfer(token, _addressToBytes32(sender), receiver, amount));

```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L197