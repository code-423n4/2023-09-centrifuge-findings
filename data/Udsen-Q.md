## 1. RETURN VALUE OF THE `transferFrom` FUNCTION IS NOT CHECKED IN THE `InvestmentManager.requestRedeem` FUNCTION

The `InvestmentManager.requestRedeem` function is used by liquidity pools to request redemptions of tranche tokens from Centrifuge. The function calls the `transferFrom` function of the `lPool` contract to transfer `_trancheTokenAmount` amount from the `user` to the `escrow` as shown below:

        lPool.transferFrom(user, address(escrow), _trancheTokenAmount);

The `transferFrom` function of the `LiquidityPool` contract has a return boolean value. As shown below:

    function transferFrom(address, address, uint256) public returns (bool) {
        (bool success, bytes memory data) = address(share).call(bytes.concat(msg.data, bytes20(msg.sender)));
        _successCheck(success);
        return abi.decode(data, (bool));
    }

But the above return value is not checked inside the `InvestmentManager.requestRedeem` function after the `lPool.transferFrom` function call is made. Hence the `InvestmentManager.requestRedeem` transaction will continue even if the `transfer of _trancheTokenAmount` failed. 

Hence it is recommended to check the return value of the `lPool.transferFrom` function call in the `InvestmentManager.requestRedeem` function as shown below:

    bool success = lPool.transferFrom(user, address(escrow), _trancheTokenAmount);
    require(success, "Transfer Failed");

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L167
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L295-L299

## 2. `InvestmentManager._updateLiquidityPoolPrice` FUNCTION DOES NOT CHECK FOR DIVISION BY ZERO WHEN CALCULATING THE LIQUIDITY POOL PRICE

The `InvestmentManager._updateLiquidityPoolPrice` function is called by both `InvestmentManager.handleExecutedCollectInvest` function and `InvestmentManager.handleExecutedCollectRedeem` function. `_updateLiquidityPoolPrice` function calls the `InvestmentManager._calculatePrice` function to calculate the liquidity pool price as shown below:

        uint128 price = _toUint128(_calculatePrice(currencyPayout, trancheTokensPayout, liquidityPool));

The issue here is there is no check whether `trancheTokensPayout == 0`. Hence if the value is equal to zero the transaction will revert without providing a proper error message.

Hence it is recommended to check whether `trancheTokensPayout == 0` and if so assign `price = 0` before `InvestmentManager._calculatePrice` is called inside the `_updateLiquidityPoolPrice` function. The suggested modification to the `_updateLiquidityPoolPrice` function is shown below:

        uint128 price;
        if (trancheTokensPayout == 0) {
            price = 0;
        } else {
            price = _toUint128(_calculatePrice(currencyPayout, trancheTokensPayout, liquidityPool));
        }

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L584-L589

## 3. LACK OF `NATSPEC` COMMENTS MAKE IT DIFFICULT TO UNDERSTAND THE CODE BASE

The `InvestmentManager.sol` contract contains the core business logic for liqudity pool interactions. But there is lack of `Natspect` comments in the critical functions of the `InvestmentManager.sol` contract. This makes it difficult to understand the codebase for both developers as well as for the auditors.

Hence it is recommended to add clear `Natspec` comments to the functions of the `InvestmentManager.sol` contract for better understanding of the procotol.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol

## 4. RECOMMENDED TO EMIT EVENTS WITH BOTH OLD AND NEW VALUES FOR CRITICAL STATE CHANGES

The `LiqudityPool.updatePrice` function is used to change the `Tranche token price`. Once the price is updated it emits an `UpdatePrice` event as shown below:

        emit UpdatePrice(price);

But the event only emits the newly update price and does not emit the old `Tranche token price`. Old price will be useful for future off-chain analysis of the transactions.

Hence it is recommneded to update the `UpdatePrice` event of the `LiqudityPool.updatePrice` function to include both `old and new` price values.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L324-L328

## 5. `abi.encodePacked` USES PACKED ENCODING HENCE RECOMMENDED TO USE `abi.encode` IN IT'S PLACE, FOR CONVENIENT DECODING OF ENCODED DATA

The `Messages.formatTransferTrancheTokens` function uses `abi.encodePacked` to construct the `bytes` data varaiable to be sent to the `axelarGateway` as shown below:

        return abi.encodePacked(
            uint8(Call.TransferTrancheTokens), poolId, trancheId, sender, destinationDomain, destinationAddress, amount
        );

The `abi.encodePacked` uses packed data encoding with out any padding to make the lenght bytes32 for each data type. This could be problematic (issues with potential loss of type information) if the destination contract which decodes the `bytes data` variable does not know the correct order of the variables and thier respective data types.

Hence it is recommended to use `abi.encode` inplace of `abi.encodePacked` in the `Messages.formatTransferTrancheTokens` to construct the `bytes data` to be sent to the `axelarGateway`.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L323-L325

## 6. LACK OF INPUT VALIDATION CHECK FOR `address(0)` IN THE `Gateway.file` FUNCTION

The `Gateway.file` function is used to update the critical state variable addresses of the contract. Namely the `poolManager` and `investmentManager` contract addresses as shown below:

        if (what == "poolManager") poolManager = PoolManagerLike(data);
        else if (what == "investmentManager") investmentManager = InvestmentManagerLike(data);

But there is no input validation check for the `address(0)` for the passed in `address data` variable. Hence it is recommended to include the input validation check on the `Gateway.file` function for the `address data` input variable.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L130-L135
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.soler.sol#L103-L108
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L120-L125

## 7. TYPO'S IN THE NATSPEC COMMENTS SHOULD BE CORRECTED

```solidity
///         and it `encoded` outgoing messages and sends these to Routers.
```

Above should be corrected as follows:

```solidity
///         and it `encodes` outgoing messages and sends these to Routers.
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L80