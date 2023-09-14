# [L-1] Users may not redeem their tranche tokens due to out of membership

## Impact

Membership consists of a whitelist of users who have signed relevant agreements and successfully completed the Know Your Customer (KYC) process. This allows them to invest in specific tranches. Membership is generally valid for one year, after which it expires.

However, investors can only redeem their tranche tokens if their membership is active, creating a potential barrier for those who no longer wish to invest.


        // Check if user is allowed to hold the restricted tranche tokens

        _isAllowedToInvest(lPool.poolId(), lPool.trancheId(), lPool.asset(), user);

https://github.com/code-423n4/2023-09-centrifuge/blob/6e735d42c1e0cfe52bbbd74f0e8b0c3541073060/src/InvestmentManager.sol#L157

## Recommended Mitigation Steps

Only check membership if a user want to deposit, not when he wants to redeem.

# [L-2] View function is not accurate because it used the price from latest update that can be staled

View function in Investment Manager and Liquidity pool such as totalAssets, convertToShares, convertToAssets, maxDeposit, maxMint, maxWithdraw, previewDeposit, ... are not accurate because they used the price from latest update that can be staled.

This is because the process in Centrifuge Chain is asynchronous. The price is updated when there is a message from Centrifuge Chain to the current chain and function `_updateLiquidityPoolPrice` is called.

https://github.com/code-423n4/2023-09-centrifuge/blob/6e735d42c1e0cfe52bbbd74f0e8b0c3541073060/src/InvestmentManager.sol#L319-L419
https://github.com/code-423n4/2023-09-centrifuge/blob/72b7a3ae44993459757b0c4f1623fe765cd02d8f/src/LiquidityPool.sol#L111-L137
https://github.com/code-423n4/2023-09-centrifuge/blob/72b7a3ae44993459757b0c4f1623fe765cd02d8f/src/LiquidityPool.sol#L154-L172
https://github.com/code-423n4/2023-09-centrifuge/blob/72b7a3ae44993459757b0c4f1623fe765cd02d8f/src/LiquidityPool.sol#L186-L194

# [L-3] Trance Token with decimal more than 18 will lead to calculation revert

The trance token decimal is set by admin from Centrifuge Chain. 

        tranche.decimals = decimals;

https://github.com/code-423n4/2023-09-centrifuge/blob/705e3500f8a43579cb15751d09100c93b249120f/src/PoolManager.sol#L206

There is no check that is has to be less than or equal to 18 decimals. If the decimal is more than 18, the converting will revert.

https://github.com/code-423n4/2023-09-centrifuge/blob/6e735d42c1e0cfe52bbbd74f0e8b0c3541073060/src/InvestmentManager.sol#L674-L693

# [L-4] If the currency is USDT, the approve should be set to 0 before setting to the new value

Approve function of USDT requires setting allowance to 0 first. 

        // To change the approve amount you first have to reduce the addresses`
        //  allowance to zero by calling `approve(_spender, 0)` if it is not
        //  already 0 to mitigate the race condition described here:
        //  https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
        require(!((_value != 0) && (allowed[msg.sender][_spender] != 0)));

https://forum.openzeppelin.com/t/can-not-call-the-function-approve-of-the-usdt-contract/2130/2

Function handleTransfer in PoolManager.sol should set the allowance to 0 first before setting to the new value, it may affect the operation of the pool if the currency is USDT.

        function handleTransfer(uint128 currency, address recipient, uint128 amount) public onlyGateway {
            address currencyAddress = currencyIdToAddress[currency];
            require(currencyAddress != address(0), "PoolManager/unknown-currency");

            EscrowLike(escrow).approve(currencyAddress, address(this), amount);
            SafeTransferLib.safeTransferFrom(currencyAddress, address(escrow), recipient, amount);
        }

https://github.com/code-423n4/2023-09-centrifuge/blob/705e3500f8a43579cb15751d09100c93b249120f/src/PoolManager.sol#L261

## Recommended Mitigation Steps

Add approve 0 first before setting to the new value.

            EscrowLike(escrow).approve(currencyAddress, address(this), 0);

# [L-5] Axelar bridge gas fee is paid by Centrifuge, can be subjected to DOS.

The bridge that send messages from and to Centrifuge Chain to other chain is Axelar. The gas fee is paid by Centrifuge. It can be subjected to DOS from users.

        function send(bytes calldata message) public onlyGateway {
            axelarGateway.callContract(axelarCentrifugeChainId, centrifugeGatewayPrecompileAddress, message);
        }

In the standard implementation, the msg.sender will attach the gas fee for bridging transactions. 

Please see here for details:
https://docs.axelar.dev/dev/general-message-passing/gas-services/pay-gas



# [NC-1] Wrong variable name

The name of the variable `currencyAmountInPriceDecimals` should be `trancheTokenAmountInPriceDecimals` because it is the amount of tranche tokens that the user will receive.

        uint256 currencyAmountInPriceDecimals = _toPriceDecimals(currencyAmount, currencyDecimals, liquidityPool).mulDiv(
            10 ** PRICE_DECIMALS, price, MathLib.Rounding.Down
        );

https://github.com/code-423n4/2023-09-centrifuge/blob/6e735d42c1e0cfe52bbbd74f0e8b0c3541073060/src/InvestmentManager.sol#L598-L600

# [NC-2] Wrong Natspec

Instance:

https://github.com/code-423n4/2023-09-centrifuge/blob/72b7a3ae44993459757b0c4f1623fe765cd02d8f/src/LiquidityPool.sol#L96 : only msg.sender can call, no ward
https://github.com/code-423n4/2023-09-centrifuge/blob/6e735d42c1e0cfe52bbbd74f0e8b0c3541073060/src/UserEscrow.sol#L39: only authorized admin, no destination address