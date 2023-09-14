## 1. EXTRA `MLOAD` OPERATION CAN BE SAVED BY MODIFYING THE `InvestmentManager.requestDeposit` FUNCTION

The `InvestmentManager.requestDeposit` function and the `InvestmentManager.requestRedeem` functions initializes the `lpool` variable of type `LiquidityPoolLike` as shown below:

        address liquidityPool = msg.sender;
        LiquidityPoolLike lPool = LiquidityPoolLike(liquidityPool);

But the initial `msg.sender` assignment to the `address liquidityPool` variable can omitted and an extra `MSOTRE` operation can be saved by modifying the above line of codes as shown below:

        LiquidityPoolLike lPool = LiquidityPoolLike(msg.sender);

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L118-L119
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L149-L150


## 2. `delete` CAN BE USED TO RESET THE `uint128` VALUE TO ZERO TO OBTAIN A GAS REFUND

In the `InvestmentManager._decreaseRedemptionLimits` function if the `lpValues.maxWithdraw < _currency` condition is satisfied the `lpValues.maxWithdraw` amount is reset to `0` as shown below:

            lpValues.maxWithdraw = 0;

But it is recommended to use the `delete` keyword to reset the `lpValues.maxWithdraw` value to gain a gas refund.
Hence the above line of code can be modified as shown below:

            delete lpValues.maxWithdraw;

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L640
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L645
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L624
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L629
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L143

## 3. ARITHMETIC OPERATION CAN BE `unchecked` TO SAVE GAS

The `InvestmentManager._decreaseRedemptionLimits` function is used to decrease the `redemption limits` of the given liquidity pool. The `lpValues.maxWithdraw` value is decreased by the given `_currency` amount as shown below:

            lpValues.maxWithdraw = lpValues.maxWithdraw - _currency

Above arithmetic operation will not underflow due to previous conditional check (lpValues.maxWithdraw < _currency). Hence this arithmetic operation can be `unchecked` to save gas as shown below:

    unchecked {
        lpValues.maxWithdraw = lpValues.maxWithdraw - _currency;
    }

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L642
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L647
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L626
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L631

## 4. STORAGE VARIABLE CAN BE REPLACED WITH MEMORY VARIABLE TO SAVE GAS IN THE `InvestmentManager.calculateRedeemPrice` FUNCTION

The `InvestmentManager.calculateRedeemPrice` function saves the liquidity pool details of the provided `user` in a storage variable as shown below:

        LPValues storage lpValues = orderbook[user][liquidityPool];

Then reads the liquidity pool values using this `lpValues` storage variable which consumes lot of gas since we are using `SLOAD` operations.

Since there are no state changes happening in this `calculateRedeemPrice` view function, it is recommended to cache the `orderbook[user][liquidityPool]` value into a memory `LPValues` struct and read from the memory variable. This will replace the `SLOAD` operations with `MLOAD` operations which save lot of gas.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L560-L567
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L551-L558

## 5. ADDITIONAL CONDITIONAL CHECK IN THE `Gateway.addIncomingRouter` FUNCTION CAN SAVE ON A REDUNDANT `SSTORE` OPERATION

The `Gateway.addIncomingRouter` function is used to add a new `router` to the `incomingRouters` mapping as shown below:

    function addIncomingRouter(address router) public auth {
        incomingRouters[router] = true;
        emit AddIncomingRouter(router);
    }

But the `addIncomingRouter` function does not check whether the `address router` is already added to the `incomingRouters` mapping. Hence even if the `router` is already in the `incomingRouters` mapping it will again update the `incomingRouters[router] ` to `true` value which is a redundant `SSTORE` operation. Hence it is recommended to check whether the `incomingRouters[router] == true` condition before assigning the `true` value to the `incomingRouters[router]` mapping which will save gas on a redundant `SSTORE` operation.

The modified code is shown below:

    function addIncomingRouter(address router) public auth {
        if(incomingRouters[router] == true) return;
        incomingRouters[router] = true;
        emit AddIncomingRouter(router);
    }

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L137-L140
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L142-L145