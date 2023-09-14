### Summary
- There is a liquidity pool that represents a real world asset (eg real estate, car, paintings)
- In the liquidity pool, there is a main currency accepted and a main tranche token accepted
- The pool can have more than one tranche (junior / senior)
- A tranche determines the risk/reward ratio. A junior tranche has higher risk but higher reward. A senior tranche has lower risk and lower rewards

**For a user to participate** 
- Must be a member, checked in checkTransferRestriction()
- The user calls requestDeposit and deposits an arbitrary amount, eg 1000 USDC
- The protocol will receive the request. The user's money will be deposited into the escrow contract first
- The process is asynchronous, meaning that the protocol does not need to process the request immediately. Instead, the protocol waits for a period of time (an epoch) to process the request
- In the meantime (during the epoch period), the user can either decrease their sum deposited or cancel their deposit request
- Once the deposit is processed, the protocol will convert the deposit amount into tranche tokens and deposit the tokens into the escrow, depending on the exchange rate. Eg, the user's 1000 USDC is worth 750 tranche tokens, meaning 1 tranche token is worth 1.25 USD at that point in time.
- The user can then call deposit the withdraw the tranche tokens from the escrow. Keep in mind that the 1000 USDC has already been converted to tranche tokens, so the user can only withdraw tranche tokens
- To redeem the tranche tokens back to the native currency of the pool, eg USDC, the whole process is repeated (user calls requestRedeem)

### Codebase quality analysis
- Codebase is written well, and most edge cases is checked even with extremely high composability
- Decimals are calculated properly in the investment manager

### Centralization risks
- Huge centralization risk when it comes to Auth. Anyone with a ward status can deny everyone else of the ward status, including himself. Since the whole protocol revolves around Auth, if there is no wards in the protocol, every process will not function anymore.
- The router and the centrifuge chain must encode and decode messages properly because the whole process relies on converting native currency to tranche tokens and back. The centrifuge chain must be fair in allocating the price of the tranche tokens

### Mechanism Review
- The process of receiving tranche tokens by depositing native currency can be extremely long, especially if the epoch period is long. Users have to understand that they may have to wait for a long time, and they may not get the best conversion rate. 

### Time spent:
10 hours