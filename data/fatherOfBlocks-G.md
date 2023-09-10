**src/admins/PauseAdmin.sol**
- L19 - The File event is created, but it is never used, therefore it should be removed.


**src/InvestmentManager.sol**
- L626/631/642/647 - If a prior validation is carried out, in L623, it is not necessary to validate underflows, for that we can use unchecked. This happens in lpValues.maxDeposit - _currency and in lpValues.maxMint - trancheTokens.


**src/gateway/Gateway.sol**
- L68 - The AuthLike interface is created, but it is never used, therefore it should be removed.


**src/PoolManager.sol**
- L34 - The LiquidityPoolLike interface is created, but it is never used, therefore it should be removed.
