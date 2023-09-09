## Transfer to is not checked if it's address(0)

The InvestmentManager.handleExecutedDecreaseInvestOrder sends the specified currencyPayout of _currency token to the specified user address at https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L291

```
        SafeTransferLib.safeTransferFrom(_currency, address(escrow), user, currencyPayout);
```

The function is called via the Gateway contract by authorized routers on the function `handle`
The handle decodes a bytes via it's argument to make the call InvestmentManager.handleExecutedDecreaseInvestOrder, it's quite hard to realize that this have address(0) as user, if specified so and called the currency tokens of escrow will be sent to address(0) mistakenly and will never be recovered