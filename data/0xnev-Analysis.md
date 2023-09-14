## 1. Summary of Codebase
### 1.1 Pool Deployment
Here is an outline of how a Tranche with their associated liquidity pools can be deployed.

1. `PoolManager.addCurrency()`: Here, an asset is added to the global `currencyIdToAddress` and `currencyIdToAddress` mapping to signify support for currency asset to be allowed for liquidity pools

<br/>

2. `PoolManager.addPool()`: Here, an existing centrifuge RWA pool is added to the protocol to be allowed to be linked to liquidity pools, signified by the `pool.createdAt` timestamp updated.

<br/>

3. `PoolManager.allowPoolCurrency()`: Here, a specific currency assets are allowed to be used for centrifuge RWA pools that have been already added via `addPool()`. This is sgnified by the `pool.allowedCurrencies` mapping update.

<br/>

4. `PoolManager.addTranche()`: Here, an specific tranche is added linked to a centrifuge RWA pool already added via `addPool()`. This is signigied by the updates of the relevant data associated with tranche (RWA poolId, trancheId,  TT decimals, TT name and TT symbol) and more importantly the `tranche.createdAt` timestamp.

<br/>

5. `PoolManager.deployTranche()`: Note that steps 1-4 are not permisionless, and requires a governance proposal and vote by CFG holders before addition of liquidity pools and tranches for deployment. Here, a successfully added tranche token can be deployed. Note that the salt is constructed by the RWA poolId and trancheId to ensure same TT address for any compatible EVM chain.


<br/>

6. `PoolManager.deployLiquidityPool()`: This is the final step where successfully added tranches linked to specific RWA centrifuge pools can be deployed. This effectively means that a tranche is deployed for the RWA pool, since RWA pools can be linked to multiple tranches.

### 1.2 Investor Flow
1. `LiquidityPool.requestDeposit()`: Investors first has to request investments before actual tranche tokens can be minted. This deposit requests are added to the orderbook on centrifuge to be matched with redemption orders and executed at the end of the epoch. At any point of time, user can cancel outstanding investment orders by passing in `currencyAmount == 0`. This will communicate with the centrifuge chain to cancel any outstanding orders.

<br/>

2. `InvestmentManager.handleExecutedCollectInvest()`: After the end of the epoch, once investment orders have been successfully executed on the centrifuge chain, the incoming router contract will communicate with the gateway contract to call the `handleExecutedCollectInvest()` function. This mints the TT to the escrow contract and updates the necessary `lpValues.maxDeposit` and `lpValues.maxMint` for users to claim tranche tokens for successfully executed investment orders.

<br/>

3. `LiquidityPool.deposit()/mint()`: Once gateway contract has successfully communicated with the investment manager contract, users can claim their associated amount of tranche tokens for successfully executed investment orders.


### 1.3 Redemption Flow
1. `LiquidityPool.requestRedeem()`: Users first has to request redemption before actual currencies previously escrowed are transferred back to them. This redemption requests are added to the orderbook on centrifuge to be matched with investment orders and executed at the end of the epoch. At any point of time, user can cancel outstanding redemption orders by passing in `trancheTokenAmount == 0`. This will communicate with the centrifuge chain to cancel any outstanding orders.

<br/>

2. `InvestmentManager.handleExecutedCollectRedeem()`: After the end of the epoch, once redemption orders have been successfully executed on the centrifuge chain, the incoming router contract will communicate with the gateway contract to call the `handleExecutedCollectRedeem()` function. This burns the TT sent by user to escrow and updates the necessary `lpValues.maxRedeem` and `lpValues.maxWithdraw` for users to redeem underlying currencies for successfully executed redemption orders.

<br/>

3. `LiquidityPool.redeem()/withdraw()`: Once gateway contract has successfully communicated with the investment manager contract, users can call these functions to claim their associated amount of underlying currencies for successfully executed redemption orders.


### 1.4  Order Adjustments
1. `LiquidityPool.decreaseDepositRequest()/decreaseRedeemRequest()`: At any point of time before end of epoch, user can request a decrease in outstanding investment/redemption orders. The outgoing router will communicate with centrifuge chain to executed the relevant decrease in order amounts.

<br/>

2. `InvestmentManager.handleExecutedDecreaseInvestOrder()/handleExecutedDecreaseRedeemOrder()`: After the decrease is successfully executed on the centrifuge chain, the incoming router will communicate with the gateway contract which inturns communicates with the investment manager contract to call `handleExecutedDecreaseInvestOrder()/handleExecutedDecreaseRedeemOrder()`. This executes the relevant return of tranche tokens/underlying currency back to user from escrow respectively.

### 1.5 Manual Collection
- `LiquidityPool.collectDeposit()/collectRedeem()`: In case a investment/redemption order executed on centrifuge chain does not occur successfully, this are backup functions where user can manually trigger to collect the outcome of the executed orders on Centrifuge Chain.





## 2. Codebase Quality
| Codebase Quality Categories  | Comments |
| :---:| --- |
| **Testing**  | Unit test and Integration tests were present using Foundry, allowing ease of testing for PoCs|
| **Documentation** | The documentation for Centrifuge is well written and provides a satisfactory overview of the protocol, though some features such as order cancellation and cross chain transfers were not documented|
| **Organization** | Codebase is well organized with clear distinctions between the 18 contracts. |   

## 3. Improvements

### 3.1 `LiquidityPool.convertToShares()/convertToAssets()` could be misleading
The tranche token price denoted by `latestPrice` is only updated after the previous epoch requested orders is executed. Furthermore, the latest update of price is based on the latest order be it redemption or investment, so if there are any inaccuracies/anomalies arising from latest order executed it can cause a inaccurate `latestPrice`.

This functions are estimates useful for display purposes for users to estimate investment/redemption returns, so unknowing users would request investment/redemption orders based on previous epoch prices if the data is retrieved from these functions. Hence, sufficient warning needs to be given to users employing this functions for estimates.

### 3.2 Consider supporting an increase in investment/redemption orders

Currently, a decrease in investment/redemption orders are supported via `LiquidityPool.decreaseDepositRequest()/decreaseRedeemRequest()`. Consider supporting a increase in investment/redemption orders too for better UX so that they do not have to make another new separate order.

### 3.3 Consider separate `handle` functions for different function calls
Any changes or actions made to the protocol sent from the incoming router needs to communicate with the gateway contract via the `Gateway.handle()` function. However, currently this function checks the incoming message against all the relevant function calls until it matches. This can incur unecessary gas costs and may even cause revert on mainnet where gas costs are high with insufficient gas provided for the transaction. 


### 3.4 Weird ERC20 tokens
Any approval calls for liquidity pools deployed with non-standard tokens suchs as USDT as underlying token will revert due to differences in `approve()` function signatures, where USDT `approve()` function signature do specify a return value.

Similarly, for tokens with non-standard (e.g. DAI) or even phantom (e.g. WETH) `permit()` functions, the ERC-2612 spec could be broken which can affect any request for deposits utilizing the `LiquidityPool.requestDepositWithPermit()`. 

Hence, always take note deviating specifications when support is provided for certain tokens.


### 3.5 Consider allowing sufficient time to react to protocol pauses
During pausing of protocol, consider providing a time buffer for users to react to price changes during the pause for their executed investment/redemption orders on the centrifuge chain. This could be allowing cancellation of already executed investment/redemption orders and returning funds from escrow back to them during the time buffer.

### 3.6 Consider separating interfaces from core contracts
Consider separation of interfaces from main contract as well as a more consistent naming of interfaces to avoid confusion.

## 4. Centralization Risks

### 4.1 Protocol can deny any user from interacting with protocol via ERC1404 checks
Protocol can choose not to provide relevant ERC1404 membership for users to interact with protocol, which is required to make investment/redemption orders.

### 4.2 Admin can pause protocol at any time
The `PauseAdmin` can instantaneously pause the protocol at any time to prevent any further interactions with protocol. Similarly, the `DelayedAdmin` can pause the protocol albeit at a delay. This are fail safes for emergency scenarios but nevertheless the risk of centralization exists.

## 5. Time Spent
| Day  | Scope | Details|
| :---: | :---: | --- |
| 1-2 | [Centrifuge Docs](https://docs.centrifuge.io/) & [Contest Details](https://github.com/code-423n4/2023-09-centrifuge)| Review Docs and understand key components of protocol, namely: Pool and Tranche deployment, Investment orders and redemption orders |
| 3-5 | [Centrifuge contracts](https://github.com/code-423n4/2023-09-centrifuge) | Manual audits of contracts with the following flow, `PoolManager.sol` --> `LiquidityPool.sol` --> `InvestmentManager.sol`. The rest of the contracts were audited based on interactions with the above mentioned core contracts.
| 6 | - |Finish up Analysis report |

### Time spent:
40 hours