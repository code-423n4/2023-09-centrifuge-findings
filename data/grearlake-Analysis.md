# [01] Summary of Codebase

## 1.1 Description
Centrifuge is a transparent market project on Ethereum, where allows borrowers and lenders to transact without unnecessary intermediaries. In liquidity pool, user can invest stablecoins and receive back shares. These shares can be be used to redeem to get stablecoins back to gain profit with diffirent rates at risks based on tranch

## 1.2 Flow

### 1.2.1 Add currency
- Centrifuge chain add currency that could be used to create pool through Gateway#handle() -> PoolManager#addCurrency()
- Check that currency was added previously or not
- Check that currency token have decimals <= 18 or not, because contract only support decimals that euqal or smaller
- approve UserEscrow and InvestmentManager contract to use token in PoolManager contract
### 1.2.2 Add pool
- Centrifuge chain add pool through Gateway#handle() -> PoolManager#addPool()
- Check that poolId haven't used before
### 1.2.3 Add currency to pool
- Centrifuge chain allow pool to support currency for investing through Gateway#handle() -> PoolManager#allowPoolCurrency(). One pool can have multiple currencies that being supported
- Check that pool and currency is valid to add
- currency is marked as allowed with that pool
### 1.2.4 Add tranche to pool
- Centrifuge chain add tranche to pool through Gateway#handle() -> PoolManager#addTranche(). One pool can have multiple tranches
- Tranche is created with input data and poolId that it linked with and creation time
### 1.2.5 Deploy tranche
- Tranche is deployed when anyone call PoolManager#deployTranche() with correct poolId and trancheId
- a ERC20 tranche token address is created
### 1.2.6 Deploy liquidity pool
- Liquidity pool is deployed when anyone call PoolManager#deployLiquidityPool() with correct poolId, trancheId linked with poolId and currency that accepted to investment in that pool
### 1.2.7 Investing currency to pool
- To be able to investing, user first need to be approved to be invested by Centrifuge chain by being added to members list in RestrictionManager contract.
- User call LiquidityPool#requestDeposit() with amount of token they want to invest. Only owner of assets can call this function due to withApproval modifier. Currency then will be transfered to escrow and an announcement about request invest will be sent to Centrifuge chain
### 1.2.7.1 Decrease investment
- If user want to decrease number of currency that they want to invest, they can call LiquidityPool#decreaseDepositRequest. Only owner of assets can call this function due to withApproval modifier
- A message about decrease invest will be sent to Centrifuge chain, and deposited assets will be refunded to user
- To be able to decrease investment, user must call this function before the end of epoch
### 1.2.7.2 Cancel investment
- If user want to cancel investment , they can call LiquidityPool#requestDeposit with currency amount = 0. Only owner of assets can call this function due to withApproval modifier
- A message about cancel invest will be sent to Centrifuge chain, and deposited assets will be refunded to user
- To be able to cancel investment, user must call this function before the end of epoch
### 1.2.8 Collect share
- After epoch end, user will receive number of shares based on currency amount and rate that returned by Centrifuge chain
- In case Centrifuge chain, by somehow, failed to trigger ExecutedCollectInvest messages to transfer payout to user after epoch end, user will manually call LiquidityPool#collectInvest to redeem shares
- User then call LiquidityPool#deposit() or LiquidityPool#mint() to collect share for deposited assets
### 1.2.9 Request share redemption
- User call LiquidityPool#requestRedeem() with amount of share they want to redeem. Only owner of shares can call this function due to withApproval modifier. Tranche token then will be transfered to escrow and an announcement about request redeem will be sent to Centrifuge chain
### 1.2.9.1 Decrease redeem
- If user want to decrease number of share that they want to redeem, they can call LiquidityPool#decreaseRedeemRequest. Only owner of assets can call this function due to withApproval modifier
- A message about decrease invest will be sent to Centrifuge chain, and share will be refunded to user
- To be able to decrease redeem, user must call this function before the end of epoch
### 1.2.9.2 Cancel redeem
- If user want to cancel redeem , they can call LiquidityPool#requestRedeem with share amount = 0. Only owner of assets can call this function due to withApproval modifier
- A message about cancel redeem will be sent to Centrifuge chain, and requested share will be refunded to user
- To be able to cancel redeem, user must call this function before the end of epoch
### 1.2.10 Redeem shares
- After epoch end, user will receive number of shares based on share amount and rate that returned by Centrifuge chain
- In case Centrifuge chain, by somehow, failed to trigger ExecutedCollectRedeem messages to transfer payout to user after epoch end, user will manually call LiquidityPool#collectDeposit to redeem shares
- User then call LiquidityPool#withdraw() or LiquidityPool#redeem() to collect assets for deposited share

# [02] Centralization risks
## 2.1 Wards can taken out token value in Escrow anytime
Since Wards can taken any token value out in Escrow anytime, functions that need to transfer token from escrow can be failed, which lead to break contract
## 2.2 Wards can mint unlimited tranche token to any address
Wards can call token/ERC20#mint() to mint unlimited tranche token to any address

# [03] Time spent:
Day 1: Have a overview about in-scope contracts
Day 2: Understand workflow of contracts
Day 3 + 4: Deep dive in 3 main contracts: LiquidityPool, PoolManager, InvestmentManager
Day 5: Checking the rest of contracts, Finishing Analysis

### Time spent:
45 hours